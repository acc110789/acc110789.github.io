---
tag: java
title: AbstractCollection
---
## {{page.title}}

### AbstractCollection
`AbstractCollection`实现了`Collection`的大多数方法,还有以下两个方法没有实现.

~~~java
       public abstract Iterator<E> iterator();

       public abstract int size();
~~~

实际上`size()`方法可以在`AbstractCollection`实现如下.

~~~java
        @Override
        public int size() {
            int num = 0;
            Iterator<E> iterator = iterator();
            while (iterator.hasNext()){
                num ++;
            }
            return num;
        }
~~~

但是这种效率极低的方式看起来确实有点蛋疼,不过理论上确实可以这么实现.

### AbstractList
`AbstractList`继承自`AbstractCollection`,这个虚类还有两个方法没有实现.

~~~java
        abstract public E get(int index);

        public abstract int size();
~~~

`AbstractList`定义了一个内部类`Itr`实现了`Iterator`接口.

~~~java
private class Itr implements Iterator<E> {

        int cursor = 0;

        //记录最近访问的index,方便remove方法删除元素
        int lastRet = -1;

        //modCount是AbstractList的成员变量,
        //每当list有修改的,modCount就会发生变化.
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public E next() {
            checkForComodification();
            try {
                int i = cursor;
                E next = get(i);
                lastRet = i;
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
~~~

所以`AbstractList`的`iterator()`的实现就是new一个`Itr`返回.因此,实现了
`iterator()`方法,但是产生了`get()`这个虚方法.\\
然后这里还有一个需要注意的点,ConcurrentModificationException的抛出是因为
modCount!=expectedModCount,在next和remove的时候都会执行checkForComodification
,所以要注意以下.

1. 在遍历的代码内部,除了通过当前的`iterator`的`remove()`方法之后,不能调用其它修改list(修改`modCount`)
的方法,否则会抛出ConcurrentModificationException
2. 如果在两个线程中遍历同一个list,则任意一个线程都不能调用当前`iterator`的`remove()`方法,否则
会造成另一个线程在执行iterator的next方法的时候,checkForComodification失败,继而抛出
ConcurrentModificationException.

### AbstractSet
基本上还是`AbstractCollection`的接口函数,比较重要的是重写了`equals`函数,新函数认为,只要
对方也是set,且二者包含的元素相同就认为这个set相等.\\
`hashCode`是把所有元素的`hashCode`加起来.\\
最重要的是重写`removeAll`方法.

~~~java
    //AbstractCollection中的实现
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    //AbstractSet中的实现
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        if (size() > c.size()) {
            for (Iterator<?> i = c.iterator(); i.hasNext(); )
                modified |= remove(i.next());
        } else {
            for (Iterator<?> i = iterator(); i.hasNext(); ) {
                if (c.contains(i.next())) {
                    i.remove();
                    modified = true;
                }
            }
        }
        return modified;
    }
~~~

`remove`方法仅仅remove找到的第一个元素(如果存在多个`equal`的元素),而`removeAll`是remove
"所有"的元素(即如果本Collection存在多个元素和目标Collection中的某一个元素相同,则需要
将本Collection的这多个元素全部remove).\\
所以在AbstractCollection,现在有两种方式,要么先遍历本Collection,再遍历目标Collection.
要么先遍历目标Collection,再遍历本Collection.读者仔细想一想,先遍历本Collection效率
会高一点点,因为在遍历目标Collection的过程中,可能还没有遍历完就已经找到有和当前元素相同的元素了.
但是如果先遍历目标Collection却一定要将两个遍历全都走一遍才行.\\
AbstractSet由于其特殊性可以在removeAll的进一步提高效率,相同的元素我先输入中文行不行
