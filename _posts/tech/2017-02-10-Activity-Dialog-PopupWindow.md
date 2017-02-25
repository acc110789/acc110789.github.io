---
title: Activity和Dialog和PopupWindow Touch事件传递对比分析
tag: android
---

## {{page.title}}

本文主要分析题目中的三个东西的TouchEvent分发的区别，KeyEvent的分发与TouchEvent的分发差不多类似。
注：本文分析的源码版本是API25。

### 三者通用的逻辑部分

三者在显示的时候都是调用`WindowManager.addView(View,WindowManager.LayoutParams);`\\
，其中的参数View就是我们理解的每个View树的rootView。从这个方法进去,最终为每个rootView都分配一个
ViewRootImpl。ViewRootImpl同时持有rootView的引用和一个`WindowInputEventReceiver`(继承自\\
`InputEventReceiver`)的引用。每当发生一个KeyEvent或者MotionEvent时，`WindowInputEventReceiver`\\
首先从Native方法（这里说的Native方法是指jni底下的那些c++调用）那里收到一个这个Event，然后把这个Event\\
加入到队列，然后又取出来进行处理，处理过程就是让Event流过一个又一个InputStage（这个是个责任链模式，\\
每个InputStage持有另一个InputStage的引用），最后一个InputStage是`ViewPostImeInputStage`。\\
来看`ViewPostImeInputStage`的部分源码。

~~~
        @Override
        protected int onProcess(QueuedInputEvent q) {
            if (q.mEvent instanceof KeyEvent) {
                return processKeyEvent(q);
            } else {
                final int source = q.mEvent.getSource();
                if ((source & InputDevice.SOURCE_CLASS_POINTER) != 0) {
                    return processPointerEvent(q);
                } else if ((source & InputDevice.SOURCE_CLASS_TRACKBALL) != 0) {
                    return processTrackballEvent(q);
                } else {
                    return processGenericMotionEvent(q);
                }
            }
        }
~~~

~~~
            private int processKeyEvent(QueuedInputEvent q) {
            final KeyEvent event = (KeyEvent)q.mEvent;

            // Deliver the key to the view hierarchy.
            if (mView.dispatchKeyEvent(event)) {   //这里进入，这样就执行到了我们平时看见的dispatchKeyEvent
                return FINISH_HANDLED;
            }
            
            …………
~~~

~~~
        private int processPointerEvent(QueuedInputEvent q) {
            final MotionEvent event = (MotionEvent)q.mEvent;

            mAttachInfo.mUnbufferedDispatchRequested = false;
            final View eventTarget =
                    (event.isFromSource(InputDevice.SOURCE_MOUSE) && mCapturingView != null) ?
                            mCapturingView : mView;     //我们一般遇到的TouchEvent情况的eventTarget就是mView
            mAttachInfo.mHandlingPointerEvent = true;
            boolean handled = eventTarget.dispatchPointerEvent(event); //从这里进入
            maybeUpdatePointerIcon(event);
            mAttachInfo.mHandlingPointerEvent = false;
            if (mAttachInfo.mUnbufferedDispatchRequested && !mUnbufferedInputDispatch) {
                mUnbufferedInputDispatch = true;
                if (mConsumeBatchedInputScheduled) {
                    scheduleConsumeBatchedInputImmediately();
                }
            }
            return handled ? FINISH_HANDLED : FORWARD;
        }
~~~

然后下面是`View.dispatchPointerEvent(MotionEvent)`的代码

~~~
    public final boolean dispatchPointerEvent(MotionEvent event) {
        if (event.isTouchEvent()) {
            return dispatchTouchEvent(event); //这样就执行到了我们平时看到的dispatchTouchEvent了
        } else {
            return dispatchGenericMotionEvent(event);
        }
    }
~~~

总结：上面说的逻辑是Activity，Dialog，PopupWindow公共的，事件都是从ViewRootImpl的InputEventReciver产生，\\
然后传给对应的rootView。

### 不同的逻辑

Activity,Dialog是都有Window的，而是都是PhoneWindow(手机上，平板或者电视是什么window不确定)。Activity和\\
Dialog的rootView是由PhoneWindow负责创建（实际上PhoneWindow创建的rootView就是DecorView）,Activity和Dialog\\
只负责实现Window.Callback即可。ok，在前面公共逻辑部分说到TouchEvent会传递到rootView的dispathTouchEvent方法中\\
,这里的rootView也就是DecorView，看看DecorView的该方法源码.

~~~
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        final Window.Callback cb = mWindow.getCallback();
        return cb != null && !mWindow.isDestroyed() && mFeatureId < 0
                ? cb.dispatchTouchEvent(ev) : super.dispatchTouchEvent(ev);
    }
~~~

所以TouchEvent又被传递到了Window.Callback了，这里Activity实现的`Window.Callback.dispatchTouchEvent`如下。

~~~
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
    
    public boolean onTouchEvent(MotionEvent event) {
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }

        return false;
    }
~~~

所以又调用了`getWindow().superDispatchTouchEvent(ev)`,ok,`PhoneWindow.superDispatchTouchEvent`源码如下。

~~~
    @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);  //mDecor就是DecorView
    }
~~~

`DecorView.superDispatchTouchEvent`源码如下。

~~~
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return super.dispatchTouchEvent(event);
    }
~~~

DecorView继承自FrameLayout,FrameLayout没有重写dispatchTouchEvent，所以这里就是调用了`ViewGroup.dispatchTouchEvent`。\\
后面就是正常的ViewGroup和View的touch事件的分发了。

Dialog实现的`Window.Callback.dispatchTouchEvent`如下。

~~~
    @Override
    public boolean dispatchTouchEvent(@NonNull MotionEvent ev) {
        if (mWindow.superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }

    public boolean onTouchEvent(@NonNull MotionEvent event) {
        if (mCancelable && mShowing && mWindow.shouldCloseOnTouch(mContext, event)) {
            cancel();
            return true;
        }
        
        return false;
    }
~~~

与Activity的类似。

PopupWindow是没有Window的，rootView也不是DecorView，在API25中，rootView是PopupDecorView，也是继承自FrameLayout。\\
PopupDecorView的部分源码。

~~~
        @Override
        public boolean dispatchTouchEvent(MotionEvent ev) {
            if (mTouchInterceptor != null && mTouchInterceptor.onTouch(this, ev)) {
                return true;
            }
            return super.dispatchTouchEvent(ev);
        }

        @Override
        public boolean onTouchEvent(MotionEvent event) {
            final int x = (int) event.getX();
            final int y = (int) event.getY();

            if ((event.getAction() == MotionEvent.ACTION_DOWN)
                    && ((x < 0) || (x >= getWidth()) || (y < 0) || (y >= getHeight()))) {
                dismiss();
                return true;
            } else if (event.getAction() == MotionEvent.ACTION_OUTSIDE) {
                dismiss();
                return true;
            } else {
                return super.onTouchEvent(event);
            }
        }
~~~




