### 关于Java是值传递还是引用传递

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fzaiwitr22j20dw09a434.jpg)

对于Java是值传递还是引用传递的理解,自己以前一直认为是基本类型就是值传递,引用类型就是引用传递.然后今天所看到的一篇文章引起了我对这个问题的重新认识.

***

#### 实参与形参

> 形式参数: 在定义函数使用的参数,目的是用来接收传递进来的参数.
>
> 实际参数: 在调用函数时使用的参数.

- main 方法就是一个很好的例子

```java
public static void main(String[]args){
    sayHello("someone");
}

public String sayHello(String name){
   System.out.println("Hello:" + name);
}
```

sayHello函数括号里的就是形式参数.而在Main方法中调用时使用到的"someone"就是实际参数.

***

#### 值传递与引用传递

- 什么是值传递与引用传递?

> 值传递:在函数调用的时候将实际参数复制一份传递到函数中,那么在函数中对该参数做任何操作都不会影响到实际参数.
>
> 引用传递:在函数调用的时候将实际参数的地址直接传到函数中.在函数中对参数进行的修改,会影响到原来的实际参数.

- 先举一个比较简单的例子

```java
public static void main(String[] args) {
    ParamTest pt = new ParamTest();

    int i = 10;
    pt.pass(10);
    System.out.println("print in main , i is " + i);
}

public void pass(int j) {
    j = 20;
    System.out.println("print in pass , j is " + j);
}

---------------------------------------------------------
print in pass , j is 20
print in main , i is 10
```

可以看到pt.pass调用函数将实际参数传递到函数中修改并没有影响到原来的值.

- 那么再举一个可以推翻这个结论的例子

```java
public static void main(String[] args) {
    ParamTest pt = new ParamTest();

    User hollis = new User();
    hollis.setName("Hollis");
    hollis.setGender("Male");
    pt.pass(hollis);
    System.out.println("print in main , user is " + hollis);
}

public void pass(User user) {
    user.setName("hollischuang");
    System.out.println("print in pass , user is " + user);
}

-------------------------------------------------------------
print in pass , user is User{name='hollischuang', gender='Male'}
print in main , user is User{name='hollischuang', gender='Male'}    
```

在这个例子中可以看到实际参数的值发生改变了.这个跟我原来的认知不就符合了吗?

- 那么再来一个能推翻以上结论的例子

````java
public static void main(String[] args) {
    ParamTest pt = new ParamTest();

    String name = "Hollis";
    pt.pass(name);
    System.out.println("print in main , name is " + name);
}

public void pass(String name) {
    name = "hollischuang";
    System.out.println("print in pass , name is " + name);
}
------------------------------------------
print in pass , name is hollischuang
print in main , name is Hollis
````

是不是有点懵了.我刚看的时候也懵.

***

####  仔细分析

- 拿其中一个例子进行详细分析(对这个例子作一个小小的改动)

```java
public static void main(String[] args) {
    ParamTest pt = new ParamTest();

    User hollis = new User();
    hollis.setName("Hollis");
    hollis.setGender("Male");
    pt.pass(hollis);
    System.out.println("print in main , user is " + hollis);
}

public void pass(User user) {
    user = new User();
    user.setName("hollischuang");
    user.setGender("Male");
    System.out.println("print in pass , user is " + user);
}

print in pass , user is User{name='hollischuang', gender='Male'}
print in main , user is User{name='Hollis', gender='Male'}
```

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fzakkbd3joj20nv0k1tfq.jpg)

在创建hollis对象时在堆中开辟了一块内存`0x123456` .然后在调用函数时形式参数的user的引用也指向了这块内存空间`0x123456` ,接着在函数中又创建了一个新的对象开辟`0x456789` .此时可以看到实际参数并没有指向这个块新开辟的内存空间`0x456789` .如果是引用传递的话,理应实际参数也会指向新开的这块内存空间.



**所以上面的参数传递是值传递,只不过是把实际对象引用的地址当做值传递给了形式参数.**

***

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fzalle95tej20n40kcag4.jpg)

这个方法中,并没有对形参本身进行修改,而是修改形参持有的地址中存储的内容.

因此,值传递与引用传递的区别并不是内容的问题,而是实参有没有复制一份给形参.

而且判断实参内容有没有受影响,要看传的是什么,如果是地址,关注点应该在地址上,而且不是地址指向的对象的变化.

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fzalzpcailj20eb0b3wgx.jpg)

而这个例子,String虽然是个对象.但 name = "hollischuang",会导致创建了一个新的对象了.等价于name=new String("hollischuang");.因此引用发生改变了,对参数改变并不会对影响原来的实参.



**所以说，Java中其实还是值传递的，只不过对于对象参数，值的内容是对象的引用。**

> 引用类型的传递其实也是值传递,只不过表现出来的形式是引用传递.
>
> java中方法参数传递方式是按值传递。
> 如果参数是基本类型，传递的是基本类型的字面量值的拷贝。
> 如果参数是引用类型，传递的是该参量所引用的对象在堆中地址值的拷贝。

***

#### 参考资料

知乎:https://www.zhihu.com/question/31203609/answer/576030121