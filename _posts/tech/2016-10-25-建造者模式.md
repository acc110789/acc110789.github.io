---
title: 建造者模式
tag: design_pattern
---

## {{ page.title }}

建造者模式适用于构建复杂的对象(即对象包含了很多属性)特别适用于对于构造过程有特别要求的构造。
建造者模式实现了部件的构造过程和每个部件的构造细节的结耦。构造过程应该是统一的,构造过程
由总监类即Director来控制(如果构造过程不统一的话,可以考虑将构造过程抽象化)。构造细节由
Builder来完成,由于不同的产品的构造细节是不同的,因此,构造细节应该是抽象的。举个例子,比如
造飞机的的大体过程都是造机头、然后造机身、造机翼、造机尾、最后在飞机上喷上飞机的型号。这个大体过程
可以由一个Director来控制,但是不同的飞机比如B787和A380的具体建造过程建造的是不同的机身、机翼、机头、机尾。
下面是建造者模式造飞机的例子。

### 飞机这个产品

~~~ java
public class Plane {

    //机头
    private String head;
    //机身
    private String body;
    //机翼
    private String wings;
    //机尾
    private String tail;

    private String name;

    public String getHead() {
        return head;
    }

    public void setHead(String head) {
        this.head = head;
    }

    public String getBody() {
        return body;
    }

    public void setBody(String body) {
        this.body = body;
    }

    public String getWings() {
        return wings;
    }

    public void setWings(String wings) {
        this.wings = wings;
    }

    public String getTail() {
        return tail;
    }

    public void setTail(String tail) {
        this.tail = tail;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void print(){
        System.out.println(name + "     " + 
        head + "," + body + "," + wings + "," + tail);
    }
}
~~~

### 飞机的细节构造接口

~~~java
public interface PlaneBuilder {
    void buildName();
    void buildHead();
    void buildBody();
    void buildWings();
    void buildTail();
    Plane getPlane();
}
~~~

### 飞机的大体构造过程

~~~ java
public class PlaneDirector {
    private PlaneBuilder builder;

    public PlaneDirector(PlaneBuilder builder){
        this.builder = builder;
    }

    public Plane construct(){
        builder.buildHead();
        builder.buildBody();
        builder.buildWings();
        builder.buildTail();
        builder.buildName();
        return builder.getPlane();
    }
}
~~~

### 飞机细节构造的抽象类

~~~java
public abstract class AbstractPlaneBuilder implements PlaneBuilder {
    Plane plane = new Plane();

    @Override
    public Plane getPlane() {
        return plane;
    }
}
~~~

### A380的细节构造类

~~~java
public class A380Builder extends AbstractPlaneBuilder {
    @Override
    public void buildName() {
        plane.setName("A380");
    }

    @Override
    public void buildHead() {
        plane.setHead("a21机头");
    }

    @Override
    public void buildBody() {
        plane.setBody("a380专属机身");
    }

    @Override
    public void buildWings() {
        plane.setWings("127长的机翼");
    }

    @Override
    public void buildTail() {
        plane.setTail("436长的机尾");
    }
}
~~~

### B787的细节构造类

~~~java
public class B787Builder extends AbstractPlaneBuilder {
    @Override
    public void buildName() {
        plane.setName("B787");
    }

    @Override
    public void buildHead() {
        plane.setHead("k114大机头");
    }

    @Override
    public void buildBody() {
        plane.setBody("c234机身");
    }

    @Override
    public void buildWings() {
        plane.setWings("很灵活的机翼");
    }

    @Override
    public void buildTail() {
        plane.setTail("平尾");
    }
}
~~~

### 客户端构造一个飞机

~~~java
public class Client {
    public static void main(String[] args){
        Plane plane1 = new PlaneDirector(new B787Builder()).construct();
        plane1.print();

        Plane plane2 = new PlaneDirector(new A380Builder()).construct();
        plane2.print();
    }
}
~~~