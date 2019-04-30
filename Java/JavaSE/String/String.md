### String

#### 1. 创建类型为String对象会发生的事情

****

- 一般来说,在编译完后的class文件.里面有一个class常量池.常量池中存储了一个类的信息,包括一些常量.(如文本字符串这些).

- 在类加载阶段该class常量池的数据会转化并存储在运行时常量池.但String类型的常量比较特别,会另外保存在字符串常量池中.
- 字面量会在编译期间保存在class常量池中.然后在类加载阶段会在字符串常量池中生成对象.
- jdk7,把字符串常量池从方法区中移动到堆中了.jdk8 把方法区移除,改为直接内存的元空间.

##### 1.1 创建方式

- 字面量创建.

```java
String a = "1"
```

以上该行代码只会创建一个对象.在类加载阶段就已经在字符串常量池中创建值为"1"的对象.

- new 关键字创建

```java
String a = new String("1")
```

以上该行代码则会创建两个对象.

1. 首先编译阶段检查到字面量"1".会在类加载阶段在字符串常量池中创建值为"1"的对象.
2. 初始化时,会在堆中创建对象.a 会指向这个对象.(即引用).

- 特殊情况

```java
String a = new String("1") + new String("1")
```

以上该行代码会创建4个对象.

1. 2个new String的 匿名对象.
2. 编译后会变成StringBuilder.append("1").append("1").toString();会返回一个String值为"11".该对象在堆中
3. 字符池常量池中存在一个类加载阶段创建的值为"1"的对象.

注意:"11"该字面量在编译阶段并没有确定下来."11"是在程序运行后生成的,因此类加载阶段字符串常量池中并不会生成值为"11"的对象.



#### 2. String.intern()

***

```java
/**
* When the intern method is invoked, if the pool already contains a
* string equal to this {@code String} object as determined by
* the {@link #equals(Object)} method, then the string from the pool is
* returned. Otherwise, this {@code String} object is added to the
* pool and a reference to this {@code String} object is returned.
*/
public native String intern();
```

- 在jdk1.6的时候,调用该方法之后,虚拟机会在字符串常量池查找是否有内容与之相等的对象.

> 如果有,就返回这个对象.
>
> 如果没有,则会在字符串常量池中添加这个对象.**注意是把这个对象添加到字符串常量池**.

- 在jdk1.7之后,调用该方法之后,虚拟机会在字符串常量池中查找是否有内容与之相等的对象.

> 如果有,则返回这个对象.
>
> 如果没有,则会在堆中把这个对象的**引用**复制添加到字符串常量池中.**注意.此时添加的是对象在堆中的引用.**

那么,结合以上的知识来一道经典的面试题:

```java
String a = new String("1") + new String("1");
a.intern();
String b = "11";
System.out.println(a == b);
```

以上的题目在jdk1.6与jdk1.7中所得到的答案并不相同.

```java
// jdk 1.6 
false
// jdk 1.7
true    
```

一步一步地分析:

```java
String a = new String("1") + new String("1");
```

这一步创建了4个对象吶.但我们关心的只有两个.其他两个匿名对象可以不用管.

- 堆中会创建一个值为11的对象.字符串常量池会创建一个值为1的对象.

```java
a.intern();
```

在执行这个方法后,会把堆中值为11的对象的引用,复制一份到字符串常量池中.字符串常量池中有一个对象引用着堆中值为"11"的对象.

```java
String b = "11";
```

这一步会去字符串常量池找值为11的对象,发现有,直接返回引用.但该引用是堆中a的引用.所以根本是同一个引用.

得到最后结论是true.

```java
System.out.println(a == b);
```

***

那么现在如果改变一下`a.intern()`这句代码的位置,会发生什么?

```java
String a = new String("1") + new String("1");
String b = "11";
a.intern();
System.out.println(a == b);
```

`String a = new String("1") + new String("1");`这一步跟上面还是一样的,创建了四个对象.字符串常量池还是不会存在值为11的对象的.原因上面已经解释过了.

重点是接下来这一步:

```java
String b = "11";
```

这一步会去字符串常量池中查找,会发现没有值为"11"的对象.就会直接创建一个值为"11"的对象在字符串常量池中.

```java
a.intern();
```

这一步其实就没有作用了.不写也可以.因为字符串常量池中存在值为"11"的对象,会直接返回字符串常量池中的对象.

```java
System.out.println(a == b)
```

这里比较的是堆中的a 与 字符串常量池中的b.答案很明显就是`false`了.

怎么样?是不是很绕.但同时也很神奇.

总结:

1. 区分清楚对象究竟是在堆中还是字符串常量池中.
2. 变量拼接的对象会使用StringBuilder拼接然后toString()方法生成新的String对象.
3. .核心-重点是字面量.字面量如果没有在编译阶段放入class常量池的话,类加载阶段就不会在字符串常量池中创建该值的对象.就例如以上的情况: `String a = new String("1") + new String("1")`;



#### 3. 编译时的优化

***

编译规则:

1. `Sun`规定了String类型运算时会new一个`StringBuilder`对象,然后用append()方法追加字符串,最终以`StringBuilder`的`toString()`返回.而`toString()`方法是直接new了一个String对象返回的,如果是用 == 去比较两个String对象的地址值,肯定是不相等的.
2. 像"a" + "a"这种没有变量参与的,就会在编译时就能确定,并且这种拼接会被优化,编译器直接帮你拼好,就不需要new操作了.



所参考的文章:

[聊一聊让我蒙蔽一晚上的各种常量池](https://zhuanlan.zhihu.com/p/42184392)(这个大佬的解释直接让我思路清晰了很多.网上的文章五花八门,但都说不到点子上.而且还有错误).

[Java中String创建对象过程及其运算原理](<https://blog.csdn.net/qq_32625839/article/details/82634076>)
