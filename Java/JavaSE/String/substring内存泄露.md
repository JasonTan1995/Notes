### subString() 方法的内存泄露

#### 1. 概述

***

String类中的substring(int beginIndex, int endIndex)是日常开发中比较常用的一个方法.但他在jdk1.6和jdk1.7中却是有不同的实现的.而jdk1.6中的substring方法会引发内存泄露的问题.

什么是内存泄露?

> 是指为一个对象分配内存之后,在对象已经不在使用时未及时的释放,导致一直占据内存单元,使实际可用内存减少.



#### 2. substring的作用

***

- substring(int beginIndex, int endIndex)方法截取字符串并返回[beginIndex, endIndex-1] 范围内的内容.

```java
String a = "abcdefg";
String substring = a.substring(1, 3);
System.out.println(substring); //bc
```

![](https://www.programcreek.com/wp-content/uploads/2013/09/string-immutability1-650x303.jpeg)

- 因为x是不可变的,当使用x.substring(1,3)对x赋值的时候,它会指向一个全新的字符串.可堆中真正发生的事情并没有那么简单.jdk1.6和jdk1.7的实现是完全不一样的.



#### 3. jdk1.6 中的substring

***

String的底层是通过字符数组实现的.在jdk6中, String类包含三个成员变量:int offset, int count, char[]value,分别为数组的第一个位置的索引,字符串中包含的字符个数, 储存的字符数组.

![](https://www.programcreek.com/wp-content/uploads/2013/09/string-substring-jdk6-650x389.jpeg)

当调用substring()方法的时候,会创建一个新的string对象,但是这个string的值仍然指向堆中的同一个字符数组.它们两个之间的不同只有offset,count的值.

代码的实现:

```java
public String substring(int beginIndex, int endIndex) {
	if (beginIndex < 0) {
	    throw new StringIndexOutOfBoundsException(beginIndex);
	}
	if (endIndex > count) {
	    throw new StringIndexOutOfBoundsException(endIndex);
	}
	if (beginIndex > endIndex) {
	    throw new StringIndexOutOfBoundsException(endIndex - beginIndex);
	}
	return ((beginIndex == 0) && (endIndex == count)) ? this :    //如果开始位置等于且结束为止等于count字符个数。那么就返回本身
	    new String(offset + beginIndex, endIndex - beginIndex, value);   //否则创建一个新的字符串对象
}
```

##### 3.1 JDK6中导致的问题

如果你有一个很长很长的字符串，但是当你使用substring进行切割的时候你只需要很短的一段。这可能导致性能问题，因为你需要的只是一小段字符序列，但是你却引用了整个字符串（因为这个非常长的字符数组一直在被引用，所以无法被回收，就可能导致内存泄露）。在JDK6中，一般用以下方式来解决该问题，原理其实就是生成一个新的字符串并引用他。

```java
x = x.substring(x, y) + ""
```



#### 4. JDK7 的substring

***

上面提到的问题，在jdk 7中得到解决。在jdk 7 中，substring方法会在堆内存中创建一个新的数组。

![](https://www.programcreek.com/wp-content/uploads/2013/09/string-substring-jdk71-650x389.jpeg)

代码的实现:

- substring方法

```java
public String substring(int beginIndex, int endIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
        int subLen = endIndex - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
```

- 构造方法:

```java
public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
```

- 继续挖:

```java
public static char[] copyOfRange(char[] original, int from, int to) {
        int newLength = to - from;
        if (newLength < 0)
            throw new IllegalArgumentException(from + " > " + to);
    //----------------------------------------------------------
        char[] copy = new char[newLength];
    //----------------------------------------------------------
        System.arraycopy(original, from, copy, 0,
                         Math.min(original.length - from, newLength));
        return copy;
    }
```

可以看到是重新创建了一个数组了.并不是像jdk6那样用回原来的数组.

以上是JDK 7中的subString方法，其使用`new String`创建了一个新字符串，避免对老字符串的引用。从而解决了内存泄露问题。



原文:

[三张图彻底了解JDK6,JDK7中substring的原理及区别](https://www.hollischuang.com/archives/1232)

