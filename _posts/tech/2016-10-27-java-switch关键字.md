---
title: Java的switch关键字
---

## {{ page.title }}

Java的`switch`的用法随便找本讲Java基本知识的书看看就知道怎么用了。本文讨论一些`switch`里面
更加深入一点的知识,当然,我没有看Java编译器对`switch`的实现。还是从编译反编译的现象来观察。
先说结论:我们都知道Java的`switch`关键字可以作用于`int`,`byte`,`char`,`enum`(jdk5),`String`(jdk7),但是本质上
`switch`只支持`int`。下面是具体分析。

### enum的源代码

~~~java
    @Test
    public void testSwitchEnum(){
        MyEnum mm = new Random(System.currentTimeMillis()).
        nextBoolean() ? MyEnum.a : MyEnum.b;
        
        switch (mm){
            case a:
                System.out.println("a");
                break;
            case b:
                System.out.println("b");
        }
    }

    private enum MyEnum {
        a(1),b(2);

        private final int val;

        MyEnum(int val){
            this.val = val;
        }
    }
~~~

### enum反编译之后的代码

~~~java
    @Test
    public void testSwitchEnum() {
        TestSwitch.MyEnum mm = (new Random(System.currentTimeMillis())).
        nextBoolean()?TestSwitch.MyEnum.a:TestSwitch.MyEnum.b;
        
        switch(TestSwitch.SyntheticClass_1.
        $SwitchMap$TestSwitch$MyEnum[mm.ordinal()]) {
        case 1:
            System.out.println("a");
            break;
        case 2:
            System.out.println("b");
        }

    }

    private static enum MyEnum {
        a(1),
        b(2),
        c(3);

        private final int val;

        private MyEnum(int val) {
            this.val = val;
        }
    }
~~~

观察反编译之后的代码,可以看到`switch(enum)`的本质是`switch(int)`。

### byte的源代码

~~~java
    @Test
    public void testSwitchByte(){
        final byte zero = 33;
        final byte one = 11;
        byte a = new Random(System.currentTimeMillis()).
        nextBoolean() ? zero : one;
        
        switch (a){
            case zero:
                System.out.println("zero");
                break;
            case one:
                System.out.println("one");
                break;
        }
    }
~~~

### byte反编译之后的源代码

~~~java
    @Test
    public void testSwitchByte() {
        boolean zero = true;
        boolean one = true;
        int a = (new Random(System.currentTimeMillis())).
        nextBoolean()?33:11;
        
        switch(a) {
        case 11:
            System.out.println("one");
            break;
        case 33:
            System.out.println("zero");
        }

    }
~~~

switch byte比较源代码和反编译之后的代码非常有意思,可以看到有"编译器常量"这个现象,"编译器常量"就像
是C语言中的`# define N 100`一样,在编译期就直接被编译器给替换了。在本例子中,编译器在编译的时候就把
zero替换成了33,把one替换成了11。这里编译器还把byte型号的zero和one给优化替换成了boolean型,这个我暂时
还不知道编译器为什么这么做。可以看到case 的后面只能跟常量,即使源代码中是final的那种常量,编译器在编译的时候
会自动帮你替换成真正的int型常量。这个例子有点特殊,编译器居然直接把a的byte类型给优化成了int类型,这个一半不会这样
的。上面这个例子有点特殊,下面换个例子。

### byte的源代码2

~~~java
    @Test
    public void testSwitchByte(){
        byte zero = 33;
        byte one = 11;
        byte a = new Random(System.currentTimeMillis()).
        nextBoolean() ? zero : one;
        
        switch (a){
            case (byte) 25777:
                System.out.println("three");
                break;
            case 33:
                System.out.println("zero");
                break;
            case 11:
                System.out.println("one");
                break;
        }
    }
~~~

### byte的反编译的源代码2

~~~java
    @Test
    public void testSwitchByte() {
        byte zero = 33;
        byte one = 11;
        byte a = (new Random(System.currentTimeMillis())).
        nextBoolean()?zero:one;
        
        switch(a) {
        case -79:
            System.out.println("three");
            break;
        case 11:
            System.out.println("one");
            break;
        case 33:
            System.out.println("zero");
        }

    }
~~~

这个版本编译器没有帮我把a的类型优化成int,看来javac编译器以后还要好好看看啊,不然在用反射改东西的还
不好整。但是值得注意的是虽然看起来好想是`switch(byte)`,但是实际上是把byte转化成了int,所以实际上
是`switch(int)`,但是如果在`switch(byte)`的`case`里面跟一个超过了byte范围的值,编译器会报错。对于超过byte范围的int,必须强
转成byte才行。在限定了int的范围之后,在程序员看来就好想是真的`switch(byte)`一样。然后`switch(char)`和`switch(byte)`相似,
都是转化成了`switch(int)`在处理。下面介绍一下jdk7引入的`switch(String)`。

### String的源代码

~~~java
    @Test
    public void testSwitchString() {
        String aa = new Random(System.currentTimeMillis()).
        nextBoolean() ? "abc" : "cba";
        
        switch (aa) {
            case "abc":
                System.out.println("abc");
                break;
            case "cba":
                System.out.println("cba");
                break;
        }
    }
~~~

### String反编译之后的源代码

~~~java
    @Test
    public void testSwitchString() {
        String aa = (new Random(System.currentTimeMillis())).
        nextBoolean()?"abc":"cba";
        
        byte var3 = -1;
        switch(aa.hashCode()) {
        case 96354:
            if(aa.equals("abc")) {
                var3 = 0;
            }
            break;
        case 98274:
            if(aa.equals("cba")) {
                var3 = 1;
            }
        }

        switch(var3) {
        case 0:
            System.out.println("abc");
            break;
        case 1:
            System.out.println("cba");
        }

    }
~~~

这个编译器优化的有点多,通过比较,我们可以发现其实并没有`switch(String)`这回事儿,本质上还是`switch(int)`
`switch(String)`本质上是通过`switch(String.hashCode())`来实现的。那这边有个问题,如果有两个String的
hashCode是相同的怎么办呢?这里就跟HashMap里面先判断hashCode,再用equal来判断是不是真的相同是一个道理。
在进入case之后,会进一步判断是不是真的equal。上面的例子可能还不够直观,下面给出一个更加直观的例子。

### String的源代码2

~~~java
    @Test
    public void testSwitchString() {
        String aa = new Random(System.currentTimeMillis()).
        nextInt(2) == 0 ? "buzzards" : "righto";
        
        switch (aa) {
            case "buzzards":
                System.out.println("buzzards");
                break;
            case "righto":
                System.out.println("righto");
                break;
        }
    }
~~~

### String反编译之后的源代码2

~~~java
    @Test
    public void testSwitchString() {
        String aa = (new Random(System.currentTimeMillis())).
        nextInt(2) == 0?"buzzards":"righto";
        
        System.out.println(aa.hashCode());
        byte var3 = -1;
        switch(aa.hashCode()) {
        case -931102253:
            if(aa.equals("righto")) {
                var3 = 1;
            } else if(aa.equals("buzzards")) {
                var3 = 0;
            }
        default:
            switch(var3) {
            case 0:
                System.out.println("buzzards");
                break;
            case 1:
                System.out.println("righto");
            }

        }
    }
~~~

在这个例子中,在这个例子,sb编译器又做了一些优化,真的看到这些优化我真的害怕啊,你万一优化出一些副作用出来,我真的要吐血啊。
废话不多说了,在这个例子中,`"buzzards"`和`"righto"`的hashCode值都是-931102253,在我的源代码中是有两个case的,但是反编译
出来的代码只有一个case了,由于hashCode相同,所以进一步在这个case的内部用了equal做了进一步的区分。\\
编译器的优化将我的break去掉了,然后在default中又搞了一个switch进行了输出。再说一次,这个优化真的看着害怕。


