---
title: Apache的HttpClient的Api笔记
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

这里我访问了一个并不存在的页面`https://www.zhihu.com/people/ifuck/followees`来演示一个GET请求的过程。
知乎也给了一个http的response。