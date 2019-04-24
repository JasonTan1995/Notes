### String 创建对象的过程

#### 1. 概述

***

String类是日常开发中经常会使用到的类,从它的源码可得知String是不可继承的并且它的底层是一个char数组.

String类同时也是一个不可变的(`immutable`),由于字符串池(`string constant pool`)是堆中(`heap`)的一块存储空间 

- (字面量的情况下)当创建一个字符串时,如果池中存在就会返回一个它在堆中的引用.而不是直接在堆中新建一个对象.那此时如果字符串是可变的话,通过一个引用就能改变它的值,那其他引用它的值也会跟随着变化,从而会发生错误.
- (new 的情况下) 无论池中存不存在,都会在堆中创建对象.

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
     private final char value[];
}
```

```java
String s = "12";
String s1 = "34";
String s2 = s + s1; 
String s3 = "1234";
String s4 = "12" + "34"; 
String s5 = s + "34";
String s6 = "12" + new String("34");

//==================================
 System.out.println(s2 == s3); 
System.out.println(s2 == s4); 
System.out.println(s3 == s4); 
System.out.println(s3 == s5); 
System.out.println(s2 == s5); 
System.out.println(s3 == s6); 
```

以上是一道面试题,但能很好的理解String在创建对象的过程中究竟发生了什么.看看你能不能做对.答案在本文最后揭开.接下来分析下String创建对象的过程.



#### 2.  内存中存放的位置

***

说起String,肯定离不开字符串常量池(`string constant pool`),那字符串常量池究竟是什么?

![](https://images2017.cnblogs.com/blog/1177828/201709/1177828-20170929204305559-896175948.png)

##### 2.1 Class 常量池(Class Constant Pool)

- 每个Java类被编译后,就会形成一个class文件;class文件中除了包含类的版本,字段,方法,接口等描述信息外,还有一项信息就是常量池.常量池用于存放被编译后生成的各种字面量和符号引用.

##### 2.2 运行时常量池(Runtime Constant Pool)

- 运行时常量池是方法区(Non-Heap)的一部分.Class常量池中的内容会在类加载后存放在方法区中的运行时常量池中.

  > 什么是字面量和符号引用?
  >
  > 字面量: 1. 文本字符串. 2. 八种基本类型的值.  3. 被声明为final的常量.
  >
  > 符号引用: 1. 类和方法的全限定名.  2. 字段的名称和描述. 3. 方法的名称和描述.

- 相对于Class常量池来说,运行时常量池有一个重要特征就是动态性.可以在运行期间将新的常量放入池中,这种特性利用得比较多的是String.intern()方法.

##### 2.3 字符串常量池(string constant pool)

字符串常量池的位置就在下图中的`Interned Strings`中(但在jdk1.7被抽在堆里面去了,但原本的逻辑影响不大).

**注意:**  字符串常量池与运行时常量池是两个完全不同的区域,字符串常量池是全局共享的.而运行时常量池是每个类都会有一个的.

![](https://images2017.cnblogs.com/blog/1177828/201709/1177828-20170920153811462-862037386.jpg)



#### 3. String的创建过程

***

- String的创建方式有两种:
  1. 字面量.  String str = "hello";
  2. new 关键字. String str = new String("hello");

那这两种创建方式是否有区别呢?

##### 3.1 两种创建方式在内存中的区别

###### 3.1.1 首先以字面量的方式来给`str`赋值"Hello".

```java
String str = "Hello";
```

- 该文件编译后会得到一个.class后缀的文件,里面信息中有一块叫做常量池的区域(`Constant Pool`).Class常量池主要存储的就包括字面量.所以字符串"Hello"就存放在这里.
- 当程序用到该类的时候, .class文件会被解析到内存中的方法区.一般来说.class文件中常量池信息会被加载到运行时常量池中,但唯有String是特别的."Hello"会在堆中创建一个对象,同时字符串常量池会存放一个对象的引用.

![](https://images2017.cnblogs.com/blog/1177828/201709/1177828-20170920154211321-596178280.png)

- 这个时候该类只是被加载而已,变量`str`还没有初始化,堆中已经存在了一个对象了.

- 当变量`str`真正初始化的时候,虚拟机会去字符串常量池中找是否有`equals("Hello")`的string,有则返回该对象的引用,没有则会创建一个对象并把该对象的引用方法字符串常量池中.

  ![](https://images2017.cnblogs.com/blog/1177828/201709/1177828-20170920212341228-1393425975.png)

当使用字面量赋值的方法创建新的字符串,无论创建多少次,只要字符值相同,它们都是指向堆中的同一个对象.

```java
String str1 = "Hello";
String str2 = "Hello";
String str3 = "Hello";

System.out.println(str1 == str2); //true
System.out.println(str1 == str3); //true
System.out.println(str2 == str3); //true
```

![](https://images2017.cnblogs.com/blog/1177828/201709/1177828-20170920213030587-629195501.png)

###### 3.1.2  使用new 关键字创建字符串

如果使用的是new关键字创建字符串的话,**无论常量池中是否存在该字符串,都会在堆中开辟一块内存.**

```java
String str1 = "Hello";
String str2 = "Hello";
String str3 = new String("Hello");
```

![](https://images2017.cnblogs.com/blog/1177828/201709/1177828-20170920213937103-776364568.png)





不确定:

字符串常量池中存放的是字符串的字面量还是在堆中的引用?

运行时常量池与字符串常量池的关系?

参考自:

[JVM常量池浅析](https://www.jianshu.com/p/cf78e68e3a99)

[深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)

[为什么Java中的String类是不可变的](http://www.cnblogs.com/justcooooode/p/7514863.html)

[理解Java字符串常量池与intern()方法](http://www.cnblogs.com/justcooooode/p/7603381.html)

[我终于搞清楚了和String有关的那点事儿](https://www.hollischuang.com/archives/2517)

[Java中String创建对象过程及其运算原理](https://blog.csdn.net/qq_32625839/article/details/82634076)