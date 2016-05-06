---
title: java queue需要知道知识
tag: java
---

## {{ page.title }}

<br/>

1. 我原来总是有一种印象,总是觉得队列`Queue`是一种特殊的满足"先进先出"特征的`List`,
实际上只要满足"先进先出"的就是`Queue`,`Queue`的组织形式不一定是`List`.在java
中,`Queue`接口是继承自`Collection`,而不是`List`.
2. `Queue`感觉最容易混淆的是到底存不存在这个元素.比如`peek()`这个方法是返回`Queue`
的元素(但是不移除),如果此时`Queue`是空就返回`null`,如果不为空,但是下一个元素是就恰好是
`null`,也会返回`null`,这就搞不清楚到底`Queue`是不是为空了.但是有的`Queue`又是不能
插入`null`元素的,所以对于这种`Queue`来说`peek()`方法是能分辨的,如果是其它允许插入`null`
的`Queue`,还是建议用`element()`,但是这时候如果如果真是空的话,又回抛出`NoSuchElementException`
,这是一个`RuntimeException`,所以在使用`element()`之前,还是要先判断一下`isEmpty()`.
