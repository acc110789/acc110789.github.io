---
title: Apache HttpRequest接口
---

## {{ page.title }}

### 接口设计

先说接口,HttpRequest继承自HttpMessage,在HttpMessage的基础上就多了一个方法,那
就是\\
`RequestLine getRequestLine();`,这个前面已经说了。HttpRequest在实际的协议中,有8中请求
格式,分别是`Get`,`Head`,`Options`,`Patch`,`Post`,`Put`,`Trace`,`Delete`,其中,`Patch`,
`Post`,`Put`三种是可以有MessageBody的,其它几种没有body。为了解决这个问题,设计了一个子接口\\
`HttpEntityEnclosingRequest`,它在`HttpRequest`的基础上,多了三个方法。

~~~
boolean expectContinue();
void setEntity(HttpEntity entity);
HttpEntity getEntity();
~~~

具体的方法作用就不说了,到用的时候看下文档就行了,`HttpEntity`就可以理解成Body。使用的时候领`Post`,
`Put`,`Patch`实现`HttpEntityEnclosingRequest`即可。其它几种Http实现普通的`HttpRequest`即可。

### 其它Api设计

在HttpRequest这一块的接口,有两套具体的实现,一套是不可以配置(没有实现`Configurable`接口),另一套是可以配置的。
实际上使用的时候,好想都是使用可配置的那一套Api,我暂时也不知道不可配置的那一套Api是用来干嘛的。哦,对了,`Configurable`的接口是这样的。
`RequestConfig getConfig();`,在`RequestConfig`中,可以对配置很多东西,比如Http代理、重定向、Cookie处理等、很多都还没有搞懂,感觉
这样的设计很好,把我一些细节的东西都放在一起了。

还是先来说一说不可配置的那一套实现,这套实现比较简单。先说一个叫`AbstractHttpMessage`的抽象类用`HeaderGroup`
实现了`HttpMessage`中关于`Header`的增、删、查、改操作。然后`BasicHttpRequest`继承了`AbstractHttpMessage`并且
实现了`HttpRequest`接口。`BasicHttpRequest`的构造函数需要把method,uri和协议版本传进去(默认是http1.1)。
`BasicHttpRequest`这个Api是不需要Body的。对于需要Body的请求,有个叫`BasicHttpEntityEnclosingRequest`的Api,它
继承自`BasicHttpRequest`,然后实现了`HttpEntityEnclosingRequest`接口,感觉这套实现好简单,就是用户用起来可能有点不爽。

然后来介绍可以位置的一套实现,这套Api的关键就是`HttpRequestBase`,里面持有一个`RequestConfig`实现了`Configurable`接口。
还实现了一个新的接口`HttpUriRequest`,这个接口继承自`HttpRequest`。新增了四个方法。

~~~
String getMethod();
URI getURI();
void abort() throws UnsupportedOperationException;
boolean isAborted();
~~~

`HttpRequestBase`是一个抽象类,`getMethod()`仍然保持为抽象方法,有具体的类比如`HttpGet`去实现。持有一个`URI`对象
实现了`URI getURI();`方法。\\
`HttpRequestBase`继承自`AbstractExecutionAwareRequest`,后面两个方法均在\\
`AbstractExecutionAwareRequest`中实现。

不用Body的Http请求直接继承`HttpRequestBase`,然后实现`public String getMethod()`即可。
对于需要Body的三个Http请求,`HttpEntityEnclosingRequestBase`继承自`HttpRequestBase`,在其基础上
持有一个`HttpEntity`对象实现了`HttpEntityEnclosingRequest`接口,另外三种需要Body的Http请求就是
继承了`HttpEntityEnclosingRequestBase`,然后实现了`public String getMethod()`方法。