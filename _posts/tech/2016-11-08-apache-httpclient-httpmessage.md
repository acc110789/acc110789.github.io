---
title: Apache的HttpClient的HttpMessage接口笔记
---

## {{ page.title }}

### 感想
Apache的HttpClient的设计是面向对象设计的非常好的例子,抽象和具体分离。设计的很好。
本文就是我记的一些笔记。学这些东西我觉得很重要的是理论要结合实践。要将我们看到的东西
和实际上的东西对应起来才能真正的理解。

### 基本的东西
如果不是https(不涉及到ssl/tls协议),只需要用
telnet就可以模拟一个http协议过程,而如果是https,可以用openssl把前面的秘钥协商过程
先帮我们做了,然后再模拟http过程。本文我们以zhihu为例子,而知乎用的https协议,所以用openssl来模拟http协议。
命令是`openssl s_client -host www.zhihu.com -port 443`。下面我先后输入下列命令

~~~
openssl s_client -host www.zhihu.com -port 443  #建立ssl/tls连接

GET https://www.zhihu.com/people/ifuck/followees HTTP/1.1
HOST:www.zhihu.com  #http get请求,在本句话完毕之后连续两个回车
~~~

下面是截图。

<img width="700dp" src="/images/ssl_1.png" alt="ssl_1">

从这个图中我们可以看到,叶子证书显示的组织竟然是锤子科技?难道罗永浩和知乎有什么py交易?
还是被劫持了?就不管了。

<img width="700dp" src="/images/ssl_2.png" alt="ssl_2">

这样秘钥协商阶段就结束了。

<img width="700dp" src="/images/http_1.png" alt="http_1">

这里我访问了一个并不存在的页面:\\
`https://www.zhihu.com/people/ifuck/followees`来演示一个GET请求的过程。
知乎也给了一个典型的http的response,后面的介绍就重点关注这张图。

### HttpMessage

这个库给出了这样一个接口,这个接口代表了一个Http报文,它可以是一个HttpRequest,也可以是一个HttpResponse。
或者说HttpMessage是HttpRequest和HttpResponse抽象出来的公共接口部分。在HttpMessage的Doc中有这样的描述。\\

~~~
       generic-message = start-line
                         *(message-header CRLF)
                         CRLF
                         [ message-body ]
       start-line      = Request-Line | Status-Line
~~~

也就是一个通用的Http报文的第一行是一个start-line(对于HttpRequest来说start-line是Request-Line,而
对于HttpResponse来说start-line是Status-Line)。紧接着接若干个(*表示0个或者1个或者2个或者多个)header(header本质上就是一个键值对)
然后一个空行,然后是一个message-body,对于GET请求来说没有body,对于Post请求来说可以有body,而对于HttpResponse来说,body
可以是一个html,也可以是一个json或者图片文件什么的都有可能。

我想如果让我来设计HttpMessage的接口的话,HttpMessage应该下面还有两个衍生接口,分别是HttpRequest和HttpResponse。
而对于HttpMessage这个接口本身来说,应该有一个方法是`StartLine getStartLine()`,StartLine是一个抽象的东西,又分别有
RequestLine和StatusLine继承自StartLine。

观察上面给出的图3,RequestLine实际上就是\\
`GET https://www.zhihu.com/people/ifuck/followees HTTP/1.1`\\
而StatusLine实际上是`HTTP/1.1 302 Found`

所以RequestLine和StatusLine有个公共的地方,那就是`Http/1.1`,可以抽象到StartLine中。所以StartLine中应该有一个方法。

~~~
ProtocolVersion getProtocolVersion();
~~~

`ProtocolVersion`的设计如下.

~~~java
public class ProtocolVersion{
    private final String protocol; //代表Http/1.1中的Http
    private final int major;      //代表Http/1.1中的的第一个1
    private final int minor;      //代表Http/1.1中的的第二个1
}
~~~

RequestLine除了有StatusLine的方法外,还多了两个东西,所以RequestLine应该是如下结构。

~~~
Method getMethod();  //在本例中Method是GET,Method可以是枚举类型,或者直接是String都行吧
URI getUri();       //返回请求的资源地址
~~~

StatusLine,多了状态码和具体的原因,StatusLine的结构应该是如下。

~~~
int getStatusCode();
String getMessage();
~~~

然后在HttpRequest中重写`getStartLine()`使其返回值是RequestLine,HttpResponse的`getStartLine()`返回StatusLine。
这样start-line这一块的结构就搞定了。\\
可惜,不是我在设计这个接口。实际上HttpMessage里面并没有
`getStartLine()`这个接口,但是有`ProtocolVersion getProtocolVersion();`这个Api。
但是在HttpRequest里面有`RequestLine getRequestLine();`这个APi。
在HttpResponse里面也有`StatusLine getStatusLine();`这个Api。其实呢,仔细考虑一下,在用户实际上使用HttpClinet的时候,
肯定是把HttpRequest和HttpResponse分开在使用,所以这样分开设计也是有道理,之所以设计HttpMessage这个接口,纯粹地就是想把
公共方法放在一起,并不是想让用户把HttpRequest和HttpResponse当成一个东西来考虑。\\
下面是Header(message-header),Header是Request和Response的公共接口
对Header的操作应该放在HttpMessage中,HttpMessage的衍生接口不再关心Header。而一个HttpMessage可能有多个Header,
对Header的操作不外乎就是增、删、查、改。下面讲讲Header部分的结构。

~~~
//下面是Header的结构
message-header = field-name ":" [ field-value ]
 field-name     = token
 field-value    = *( field-content | LWS )
 field-content  = &lt;the OCTETs making up the field-value
                  and consisting of either *TEXT or combinations
                  of token, separators, and quoted-string&gt;
                  
                  
//上面的field-value又可以进一步划分为一些子结构 HeaderElement
header  = [ element ] *( "," [ element ] )
 element = name [ "=" [ value ] ] *( ";" [ param ] )
 param   = name [ "=" [ value ] ]
 
 name    = token
 value   = ( token | quoted-string )
 
 token         = 1*&lt;any char except "=", ",", ";", &lt;"&gt; and
                        white space&gt;
 quoted-string = &lt;"&gt; *( text | quoted-char ) &lt;"&gt;
 text          = any char except &lt;"&gt;
 quoted-char   = "\" char
~~~

~~~
//Header的主要Api
String getName();
String getValue();
HeaderElement[] getElements() throws ParseException;
~~~

~~~
//HeaderElement的主要Api
String getName();
String getValue();
NameValuePair[] getParameters();
~~~

~~~
//NameValuePair的Api,所以Parameter的本质就是键值对
String getName();
String getValue();
~~~

Header的结构是`键:值`(Header就是以冒号作为分割),然后Header的值里面包含多个HeaderElement,任意
两个HeaderElement以逗号进行分割。HeaderElement的结构是\\
`键=值;参数Name0=参数值0;参数Name1=参数值1;……`\\
HeaderElement可能分成多个Part,各个Part之间用分号进行分割。每个Part本质上都是第一个键值对,键和值之间用等号进行分割,
第一个部分提供HeaderElement的Name和Value。第二个Part及以后的Part都是作为Parameter。

下面以一个实际的Header作为例子说明每个方法返回的是什么。一个Header如下。

~~~
Set-Cookie: _xsrf=; Domain=zhihu.com; expires=Fri, 13 Nov 2015 08:25:12 GMT; Path=/
~~~

记这个Header的一个引用是header,那么`header.getName()`返回的是`Set-Cookie`,`header.getValue()`返回的是\\
`_xsrf=; Domain=zhihu.com; expires=Fri, 13 Nov 2015 08:25:12 GMT; Path=/`\\
`header.getElements()`的返回如下。

~~~
_xsrf=; Domain=zhihu.com; expires=Fri    //第0个HeaderElement,后面记为element0吧
13 Nov 2015 08:25:12 GMT; Path=/         //第1个HeaderElement,后面记为element1吧
~~~

`element0.getName()`返回`_xsrf`,`element0.getValue()`返回``(空的字符串)。`element0.getParameters()`返回
一个键值对数组如下。\\

~~~
Domain=zhihu.com
expires=Fri
~~~

另一个Header的例子,Header的结构如下。

~~~
X-Req-ID: 79265375826D1E6
~~~

`header.getName()`返回`X-Req-ID`,`header.getValue()`返回`79265375826D1E6`。\\
`header.getElements()`的返回如下。

~~~
79265375826D1E6                         //只有一个HeaderElement,后面记为element0吧
~~~

`element0.getName()`返回`79265375826D1E6`,`element0.getValue()`返回`null`。
`element0.getParameters()`返回的数组长度是0。

至此,Header的结构应该全都将清楚了。
