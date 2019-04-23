#### ArrayList 源码分析

![](https://ws1.sinaimg.cn/large/6b297ce5gy1g03p9fsqpaj20mn0d8dl6.jpg)

#### 1. 概述

***

ArrayList 是比较常用的一个类,它的底层是通过一个定长数组来实现的.ArrayList允许空值以及重复元素,当大量元素插入时,会通过扩容机制重新生成一个更加大的数组,因此其核心也是扩容机制.另外,正因为是底层是基于数组实现的,访问元素的效率是非常高的.(`时间复杂度为o(1)`).

#### 2. 源码分析

***

##### 2.1 构造方法

```java
private static final int DEFAULT_CAPACITY = 10;
   
private static final Object[] EMPTY_ELEMENTDATA = {};
  
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
transient Object[] elementData; // non-private to simplify nested class access
   
private int size;

public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
   }
}
```

ArrayList 有两个构造方法,无参的构造方法是我们比较常用的.无论有参还是无参都能看出数组初始化的时候是一个空数组.当第一次插入元素时才会进行扩容且扩建的容量为10.因此需要注意的是,假如我们能清楚插入多少元素的时候,最好还是指定一下初始化的容量.毕竟扩容是非常耗性能的操作.

##### 2.2 插入元素

```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
}

public void add(int index, E element) {
        rangeCheckForAdd(index);
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
}
```

对于数组结构而言,插入元素大多数都会有两种情况:

1. 在数组的最后一个元素后追加

   ![](https://ws1.sinaimg.cn/large/6b297ce5gy1g03gcqvf6hj218e0a0tek.jpg)

   >这种情况下的处理会比较简单:
   >
   >1. 检查数组是否有足够的空间插入
   >2. 将新的元素插入到序列的尾部

2. 在数组指定位置插入.

   ![](https://ws1.sinaimg.cn/large/6b297ce5gy1g03gctmzq4j218g0cq7cp.jpg)

   > 而在指定的位置插入就比较复杂了:
   >
   > 1. 检查数组是否有足够的空间插入
   > 2. 将该位置的元素以及该位置往后的元素都向后挪一位
   > 3. 将新的元素插入到指定的位置

小结: 在日常开发应该尽量避免使用第二种方法来添加元素到数组,因为指定的位置及其后的元素都需要向后移动一位(`时间复杂度为o(N)`),频繁的移动元素会导致效率问题.

##### 2.3 扩容机制

扩容后的容量是扩容前的1.5倍.

```java
private static final int DEFAULT_CAPACITY = 10;

/** 计算最小容量 */
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果是一个空数组的话,其扩容的容量就为10.
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

/** 扩容的入口方法 */
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

/** 扩容的核心方法 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // newCapacity = oldCapacity + oldCapacity / 2 = oldCapacity * 1.5
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 扩容(将旧数组中的元素搬到新数组中去)
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    // 如果最小容量超过 MAX_ARRAY_SIZE，则将数组容量扩容至 Integer.MAX_VALUE
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

##### 2.4 删除元素

```java
public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
}


public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
}

private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
}
```

与插入元素不相同的是,删除方法需要指定索引或要删除的对象.

![](https://ws1.sinaimg.cn/large/6b297ce5gy1g03m048amaj218e0c4jzh.jpg)

> 删除元素需要进行的操作:
>
> 1. 如果需要返回被删除元素的对象,就先获取index对应元素的值.
> 2. 将所有元素的值往前移动一位.覆盖要删除的元素的值.
> 3. 将最后一个元素置空,并将size减一
> 4. 返回被删除的元素

需要注意的是,假如我们大量的删除一个数组的元素,就会造成数组有大量的闲置空间.但是又没有自动进行缩容的机制.那就会导致大量的闲置空间不能被释放,造成浪费.针对这种情况ArrayList也提供了相应的处理方案:

```java
/**
 * Trims the capacity of this <tt>ArrayList</tt> instance to be the
 * list's current size.  An application can use this operation to minimize
 * the storage of an <tt>ArrayList</tt> instance.
 */
public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
```

![](https://ws1.sinaimg.cn/large/6b297ce5gy1g03me4zrx5j218g092q7e.jpg)



#### 3. 其他细节

***

##### 3.1 遍历

ArrayList 实现了RandomAccess接口(`跟Serializable一样是个标志`),代表它具有随机访问的能力,一般情况下我们会使用forEach进行遍历,但我们都知道forEach是语法糖,其编译后的实现还是迭代器,这并不是推荐的遍历方式,如果在一些效率要求比较高的场合下,会更推荐使用:

```java
for (int i =0;i< list.size();i++) {
    list.get(i);
}
```

##### 3.2 关于fast-fail(快速失败机制)

我在掘金的这篇文章已经有大概的介绍了,想了解的朋友可以去看看:

https://juejin.im/post/5b7cc7ce51882542b45dd220

在这里想对以上的文章作一点补充.

我在文章的最后所说的`但如果你的Collection/Map对象实际只有一个或两个元素的时候，ConcurrentModificationException异常并不会抛出。`.有觉得好奇为什么会这样吗?

```java
List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        for (Integer i: list) {
            if (i == 1) {
                list.remove(i);
        }
}
```

上面说过,forEach是Java中的语法糖其实现是迭代器,那么编译后就等价于:

```java
List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        Iterator<Integer> it = list.iterator();
        while (it.hasNext()) {
            Integer next = it.next();
            if (next == 1) {
                list.remove(next);
            }
}
```

这里的迭代器是在ArrayList里面实现的:

```java
private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    
       final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
}
```

next()和remove()操作之前都會進行checkForComodification()检查是否有并发修改.

那么现在如果是以下这种情况就会出现异常了:

```java
List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        Iterator<Integer> it = list.iterator();
        while (it.hasNext()) {
            Integer next = it.next();
            if (next == 1) {
                list.remove(next);
            }
}
```

原因是因为之前有两个元素的时候,当元素为1时,1被删除后,元素计数器size=1,而游标cursor也等于1,hasNext()就不成立了,不会进入循环了.所以也不会触发Next()的checkForComodification().按理来说不是不触发异常,而是连触发的机会都没有了.

***

#### 参考文章

http://www.tianxiaobo.com/2018/01/28/ArrayList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/