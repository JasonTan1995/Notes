### Obejct

​     Object是所有类的根类,即父类.所有对象包括数组都会实现这个类的所有方法.

​     先来看Object的方法:

```java 
private static native void registerNatives();
static {
        registerNatives();
}
public final native Class<?> getClass();
public native int hashCode();
public boolean equals(Object obj) {return (this == obj);}
protected native Object clone() throws CloneNotSupportedException;
public String toString() {
   return getClass().getName() + "@" + Integer.toHexString(hashCode());
}        
public final native void notify();
public final native void notifyAll();
public final native void wait(long timeout) throws InterruptedException;
public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && timeout == 0)) {
            timeout++;
        }

        wait(timeout);
    }
public final void wait() throws InterruptedException {
        wait(0);
    }
    
protected void finalize() throws Throwable { }    
```

奇怪的是,由native关键字修饰的方法,都是没有方法体的.这个类没有继承,因为它就是顶级类了.那么究竟是怎么一回事呢?

#### Native

​     实际上带有native修饰的都是本地方法,而这些本地方法并不是由JAVA语言实现的,而是JAVA通过JNI,像调用Java的方法一样调用这些本地方法.实际上调用的是虚拟机中实现的C或C++方法.并且这些方法被编译成了dll,这个也是Java的底层机制.

#### JNI

​    JNI是Java Native Interface,它提供了若干的API实现了Java和其他语言的通信(主要是C和C++).

​    但JNI一开始是为了C和C++设计得,但它也并不妨碍使用其他编程语言,但注意的是JAVA程序会因此丧失了平台的可移植性.

​    JNI的副作用,一旦使用JNI,JAVA程序就会丧失了JAVA平台的两个优点.

        1. 程序不再跨平台,要想跨平台,必须在不同的系统环境下重新编译本地语言部分.
        2. 程序不再是绝对安全的，本地代码的不当使用可能导致整个程序崩溃。一个通用规则是，你应该让本地方法集中在少数几个[类](https://baike.baidu.com/item/%E7%B1%BB/6824577)当中。这样就降低了JAVA和C之间的耦合性

#### 方法的作用



参考资料:

https://blog.csdn.net/xiaojie_570/article/details/79204475