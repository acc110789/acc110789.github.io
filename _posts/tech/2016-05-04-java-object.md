---
title: java object相关的知识点
tag: java
---

## {{ page.title }}

<br/>

### hashCode
java的`Object`的`hashCode()`默认使用对象的地址计算散列码,具体算法现在还不清楚,
这里先填一个坑,以后有机会再补上.

<br/>

### equal
java的equal方法要满足几个特征(下面的式子先不要考虑什么null造成的NullPointerException).

1. 自反性. 对于任意x,一定有`x.equal(x)`返回`true`.
2. 对称性. 对于任意的x,y,如果`x.equal(y)`恒等于`y.equal(x)`.
3. 传递性. 对于任意x,y,z,如果`x.equal(y)`以及`y.equal(z)`返回`true`,则必有`y.equal(z)`返回`true`.

额外的,对于任意不为`null`的x,有`x.equal(null)`是`false`.默认的Object的equal实现是`==`.\\
如果`==`成立,必然有`equal()`返回`true`,如果`equal()`返回`true`,必然有`hashCode`相同.
`hashCode`相同不一定`equal()`返回`true`.`equal`返回`true`,`==`也不一定成立.

<br/>

可以这么说,`==`表示`相同`,`equal`表示`相等`,而`hashCode`是`相等`的预判断,
或者给`hashCode`取个名字叫`相似`.