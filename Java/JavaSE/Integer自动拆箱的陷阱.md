### Integer 的一个小问题

这个问题是源于以下的这道题目:

```java
Integer a = 3;
Integer b = 3;
Integer c = 321;
Integer d = 321;
System.out.println(a == b);//true
System.out.println(c == d); //false
```

可是答案就非常奇怪.明明都是一样为什么得出的结果会不一样呢?以下慢慢进行剖析.



#### 1. Integer 的创建方式

***

Integer 常用的创建方式有两种:

1. 使用`new`关键字:

```java
Integer i = new Integer(1);
```

这种方式使用了`new`关键字.调用了Integer类的构造方法.

```java
   /**
     * The value of the {@code Integer}.
     *
     * @serial
     */
    private final int value;

    /**
     * Constructs a newly allocated {@code Integer} object that
     * represents the specified {@code int} value.
     *
     * @param   value   the value to be represented by the
     *                  {@code Integer} object.
     */
    public Integer(int value) {
        this.value = value;
    }
```

2. 字面量方式创建:

```java
Integer i = 1;
```



参考文章:

[Java Integer(-128~127)值的==和equals比较产生的思考](https://www.cnblogs.com/csniper/p/5882760.html)

[Integer的自动拆装箱的陷阱（整型数-128到127的值比较问题）](https://blog.csdn.net/ma451152002/article/details/9076793)

