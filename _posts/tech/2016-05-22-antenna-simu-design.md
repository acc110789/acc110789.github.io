---
tag: antenna
title: antenna_simu设计原则
---
## {{page.title}}

<br/>

在普通的定向天线中,各个天线之间的时钟是不一样的,所以一个节点随时都有可能收到一个不相关
的信号,所以将节点接收信号的原则设置如下:

1. 在`nav`和`writeMode`的情况,不接受任何信号,当节点切换到`readMode`的时候,将此时
应该收到的信号取回。
2. 当接收到一个完整的没有`dirty`的`frame`的时候(如果是`dirty`的`frame`就当什么都没有
收到),如果这个`frame`是本`station`期待
的`frame`,则接着进行下一步。如果不是本`station`期待的`frame`,则也当什么都没有收到。