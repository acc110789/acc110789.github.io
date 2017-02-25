---
title: android在哪里调用了我们常用的onCreate、onResume等方法？
tag: android
---

## {{page.title}}

其实很早就想写这篇文章了，一直被拖延症阻碍着，现在狠下决心一定要把东西\\
总结出来,再不总结的话感觉又快要忘光了。

话不多说，本文来总结Activity的生命周期的根源，其实Android对于\\
Activity的管理是跨进程的，但是本文还是从Client的根源来观察源码。\\
没有涉及到Server端。本文中有些地方暂时可能有一些我的主观臆断。

`ActivityThread`中有两个重要的方法,分别是

~~~
private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason)
final void handleResumeActivity(IBinder token,
            boolean clearHide, boolean isForward, boolean reallyResume, int seq, String reason)
~~~

第一个方法的调用的时机是`Activity.H`(这个类继承自`Handler`)\\
收到消息`LAUNCH_ACTIVITY`的时候，第二个方法的调用时机是收到消息\\
收到消息`RESUME_ACTIVITY`的时候，这里我暂时猜测这两个消息都是由
另一个管理Activity的进程发出的消息(我猜测是`ActivityManagerService`),
关于Server的Client的通信这块内容以后有时间再看。

首先来看在收到消息`LAUNCH_ACTIVITY`之后都干了什么事情。

~~~
                case LAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                    final ActivityClientRecord r = (ActivityClientRecord) msg.obj;

                    r.packageInfo = getPackageInfoNoCheck(
                            r.activityInfo.applicationInfo, r.compatInfo);
                    handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
~~~

ok,`msg.obj`是一个`ActivityClientRecord`,然后就执行了\\
`handleLaunchActivity(r , null , "LAUNCH_ACTIVITY");`,该方法\\
最重要的语句有两个，分别是

~~~
Activity a = performLaunchActivity(r, customIntent);

handleResumeActivity(r.token, false, r.isForward,
                          !r.activity.mFinished && !r.startsNotResumed, r.lastProcessedSeq, reason);
~~~

下面进入`performLaunchActivity(r, customIntent);`来看一些重要的代码。\\
首先是

~~~
        ComponentName component = r.intent.getComponent();
        if (component == null) {
            component = r.intent.resolveActivity(
                mInitialApplication.getPackageManager());
            r.intent.setComponent(component);
        }

        if (r.activityInfo.targetActivity != null) {
            component = new ComponentName(r.activityInfo.packageName,
                    r.activityInfo.targetActivity);
        }
~~~

如果intent里面没有明确指定要启动的Activity，那么就要根据Intent的Action等参数解析出
真正要启动的Activity。但是如果`r.activityInfo.targetActivity`的优先级更高，\\
如果它不为空的话，就用它作为要启动的Activity，这个信息怎么指定，后面有时间再跟进。

然后是

~~~
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to instantiate activity " + component
                    + ": " + e.toString(), e);
            }
        }

~~~

这里面`mInstrumentation`的类型是`Instrumentation`,来看看\\
`mInstrumentation.newActivity()`这个方法干了什么事情。

~~~
    public Activity newActivity(ClassLoader cl, String className,
            Intent intent)
            throws InstantiationException, IllegalAccessException,
            ClassNotFoundException {
        return (Activity)cl.loadClass(className).newInstance();
    }
~~~

可见，就是简单的把上面解析出来的`component`对应的Activity创建了一个新的实例。\\
这里只是创建了Activity的实例，这个Activity还是空的。还没有进入Activity的生命周期。\\
下面就是往这个Activity里面注入一些东西。接着往下面走。

~~~
                Context appContext = createBaseContextForActivity(r, activity);
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mCompatConfiguration);
                if (r.overrideConfig != null) {
                    config.updateFrom(r.overrideConfig);
                }
                if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                        + r.activityInfo.name + " with config " + config);
                Window window = null;
                if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window);

                if (customIntent != null) {
                    activity.mIntent = customIntent;
                }
                r.lastNonConfigurationInstances = null;
                activity.mStartedActivity = false;
                int theme = r.activityInfo.getThemeResource();
                if (theme != 0) {
                    activity.setTheme(theme);
                }
~~~

`createBaseContextForActivity()` 方法代码如下。

~~~
//关注其中重点的一些代码，其它的我也暂时不知道是什么意思
        …………
        ContextImpl appContext = ContextImpl.createActivityContext(
                this, r.packageInfo, r.token, displayId, r.overrideConfig);
        appContext.setOuterContext(activity);
        Context baseContext = appContext;
        …………
        return baseContext;
~~~

`ContextImpl.createActivityContext()`方法代码如下。

~~~
return new ContextImpl(null, mainThread, packageInfo, activityToken, null, 0,
                null, overrideConfiguration, displayId);
~~~

ok,所以`createBaseContextForActivity()`说穿了，就是创建了一个`ContextImpl`,需要注意的
是`appContext.setOuterContext(activity)`也保证了里面的`ContextImpl`也持有了Activity\\
引用。下一个很重要的地方如下。

~~~
activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window);
~~~

来看一下`Activity.attach()`方法。

~~~
            final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window) {
        attachBaseContext(context);

        mFragments.attachHost(null /*parent*/);

        mWindow = new PhoneWindow(this, window);
        mWindow.setWindowControllerCallback(this);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
            mWindow.setSoftInputMode(info.softInputMode);
        }
        if (info.uiOptions != 0) {
            mWindow.setUiOptions(info.uiOptions);
        }
        mUiThread = Thread.currentThread();

        mMainThread = aThread;
        mInstrumentation = instr;
        mToken = token;
        mIdent = ident;
        mApplication = application;
        mIntent = intent;
        mReferrer = referrer;
        mComponent = intent.getComponent();
        mActivityInfo = info;
        mTitle = title;
        mParent = parent;
        mEmbeddedID = id;
        mLastNonConfigurationInstances = lastNonConfigurationInstances;
        if (voiceInteractor != null) {
            if (lastNonConfigurationInstances != null) {
                mVoiceInteractor = lastNonConfigurationInstances.voiceInteractor;
            } else {
                mVoiceInteractor = new VoiceInteractor(voiceInteractor, this, this,
                        Looper.myLooper());
            }
        }

        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager();
        mCurrentConfig = config;
    }
~~~

首先是`attachBaseContext(context)`,Activity、Service、Application都是Context的实例，
但是实际上他们都是`ContextWrapper`(Context的子类)，他们本身并没有实现Context的抽象方法，\\
而是持有了另一个Context的引用，他们本身只是一个代理。这里的`attachBaseContext(context)`就是
向Activity里面填充一个Context，让Activity这个壳子里面有了真正的东西。接着创建了PhoneWindow\\
作为自己的Window对象，设置了几个window的callback。然后还有个很重要的点如下。

~~~
mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
~~~

进入`Window.setWindowManager()`方法。

~~~
public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
            boolean hardwareAccelerated) {
        mAppToken = appToken;
        mAppName = appName;
        mHardwareAccelerated = hardwareAccelerated
                || SystemProperties.getBoolean(PROPERTY_HARDWARE_UI, false);
        if (wm == null) {
            wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
        }
        mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
    }
~~~

~~~
//WindowManagerImpl.createLocalWindowManager()

public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
        return new WindowManagerImpl(mContext, parentWindow);
    }
~~~

要注意的是,`mWindowManager`不是baseContext调用getSystemService得到的，而是重新创造了一个\\
新的local的WindowManager。这两个WindowManager是不一样的。

~~~
//Activity
@Override
    public Object getSystemService(@ServiceName @NonNull String name) {
        if (getBaseContext() == null) {
            throw new IllegalStateException(
                    "System services not available to Activities before onCreate()");
        }

        if (WINDOW_SERVICE.equals(name)) {
            return mWindowManager;
        } else if (SEARCH_SERVICE.equals(name)) {
            ensureSearchManager();
            return mSearchManager;
        }
        return super.getSystemService(name);
    }
~~~

可以看到Activity重写了`getSystemService`方法，调用`getSystemService(Context.WINDOW_SERVICE)`\\
返回的是本地的那个WindowManager，调用`Activity.getSystemService(Context.WINDOW_SERVICE)`和\\
`Activity.getBaseContext().getSystemService(Context.WINDOW_SERVICE)`将返回不同的结果，这两个\\
WindowManager具体哪里不一样，以后再进行跟进。

ok，回到`ActivityThread.performLaunchActivity()`中接着往下走，`Activity.attach()`调用完毕之后，\\
对Activity的填充也就完成了，这个Activity就变成了一个可以用的Activity了，往下就可以开始Activity\\
的生命周期了。

~~~
                activity.mCalled = false;
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                if (!activity.mCalled) {
                    throw new SuperNotCalledException(
                        "Activity " + r.intent.getComponent().toShortString() +
                        " did not call through to super.onCreate()");
                }
~~~

走到`r.isPersistable()`涉及到Activity的一个属性`android:persistableMode`,可以在AndroidManifest中指定。\\
进入到`Instrumentation.callActivityOnCreate()`里面看看。

~~~
    public void callActivityOnCreate(Activity activity, Bundle icicle) {
        prePerformCreate(activity);
        activity.performCreate(icicle);
        postPerformCreate(activity);
    }
~~~

`Instrumentation`翻译过来是仪表盘，应该是一些数据的记录所以`prePerformCreate(activity);`和\\
`postPerformCreate(activity);`应该是记录一些什么东西，这里捡重要的看。走到\\
`activity.performCreate(icicle);`里面。

~~~
    final void performCreate(Bundle icicle) {
        restoreHasCurrentPermissionRequest(icicle);
        onCreate(icicle);
        mActivityTransitionState.readState(icicle);
        performCreateCommon();
    }
~~~

我的神啊，终于走到了我们熟悉的`onCreate()`了。这边还有个小的知识点，当我们重写Activity的onCreate方法时，一定\\
要调用`super.onCreate()`,这是为什么呢？因为在`ActivityThread.performLaunchActivity()`中，在调用\\
`mInstrumentation.callActivityOnCreate()`之前，执行了`activity.mCalled = false`。然后在\\
`mInstrumentation.callActivityOnCreate()`执行完毕之后会检查`activity.mCalled`的值，如果是false，\\
就会抛出Exception，那`activity.mCalled`怎么才会变成true呢，答案是一定要调用`super.onCreate()`,\\
Activity在onCreate的最后将`mCalled`置为true。

ok，onCreate这个生命周期终于走到了，下面继续。

~~~
                r.activity = activity;
                r.stopped = true;
                if (!r.activity.mFinished) {
                    activity.performStart();
                    r.stopped = false;
                }
                if (!r.activity.mFinished) {
                    if (r.isPersistable()) {
                        if (r.state != null || r.persistentState != null) {
                            mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state,
                                    r.persistentState);
                        }
                    } else if (r.state != null) {
                        mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state);
                    }
                }
~~~

进入`activity.performStart()`

~~~
                mActivityTransitionState.setEnterActivityOptions(this, getActivityOptions());
                mFragments.noteStateNotSaved();
                mCalled = false;
                mFragments.execPendingActions();
                mInstrumentation.callActivityOnStart(this);
                if (!mCalled) {
                    throw new SuperNotCalledException(
                        "Activity " + mComponent.toShortString() +
                        " did not call through to super.onStart()");
                }
                mFragments.dispatchStart();
                mFragments.reportLoaderStart();
        
                // This property is set for all builds except final release
                boolean isDlwarningEnabled = SystemProperties.getInt("ro.bionic.ld.warning", 0) == 1;
                boolean isAppDebuggable =
                        (mApplication.getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE) != 0;
        
                if (isAppDebuggable || isDlwarningEnabled) {
                    String dlwarning = getDlWarning();
                    if (dlwarning != null) {
                        String appName = getApplicationInfo().loadLabel(getPackageManager())
                                .toString();
                        String warning = "Detected problems with app native libraries\n" +
                                         "(please consult log for detail):\n" + dlwarning;
                        if (isAppDebuggable) {
                              new AlertDialog.Builder(this).
                                  setTitle(appName).
                                  setMessage(warning).
                                  setPositiveButton(android.R.string.ok, null).
                                  setCancelable(false).
                                  show();
                        } else {
                            Toast.makeText(this, appName + "\n" + warning, Toast.LENGTH_LONG).show();
                        }
                    }
                }
        
                mActivityTranactivity.onStart();sitionState.enterReady(this);
~~~

注意到其中的`mInstrumentation.callActivityOnStart(this);`,进去

~~~
activity.onStart();
~~~

ok,`onStart()`这个生命周期也执行到了。

回到`ActivityThread.performLaunchActivity()`中继续往下走。

~~~
                if (!r.activity.mFinished) {
                    if (r.isPersistable()) {
                        if (r.state != null || r.persistentState != null) {
                            mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state,
                                    r.persistentState);
                        }
                    } else if (r.state != null) {
                        mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state);
                    }
                }
~~~

这里就是`onRestoreInstanceState()`这个生命周期了，跟其它的生命周期差不多，就不多介绍了。这样,\\
`ActivityThread.performLaunchActivity()`这个方法就走完了，总结一下，在这个方法中，主要干了下面几件事情。

1. 解析得到要启动的目标Activity是哪一个.
2. 创建对应的Activity实例，创建ContextImpl.
3. 然后对Activity绑定、填充一些数据使之成为一个可以用的Activity(包括Application、Window、WindowManager、Token、Instrumentation、ActivityInfo等等)。
4. 生命周期onCreate()
5. 生命周期onStart()
6. 生命周期onRestoreInstanceState()





