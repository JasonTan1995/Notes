### Singleton Pattern (单例模式)

#### :wrench:1. 概述

***

单例模式是设计模式中使用最为普遍的模式之一。它是一种创建对象的模式，它能确保系统当中只产生一个对象，那它有什么好处呢？

- 对于频繁使用的对象，可以省略创建对象所花费的时间。
- 由于创建对象的次数减少了，因而会降低对系统内存的使用频率；这将会减轻GC压力，缩短GC停顿的时间。

总的来说对于系统的关键组件和被频繁使用的对象，使用单例模式便可以有效地改善系统的性能。

|  角色  |             作用             |
| :----: | :--------------------------: |
| 单例类 | 提供单例的工厂，返回单例对象 |
| 使用者 |      获取并使用单例对象      |

![QQ20160406-0](http://www.hollischuang.com/wp-content/uploads/2016/04/QQ20160406-0.png)

#### :nut_and_bolt:2. 使用示例

***

##### 2.1 简单的单例类实现方法

单例类的核心在于通过一个接口返回唯一的实例对象，先举一个较为简单的例子：

```java
public class SimpleSingleton {

    private SimpleSingleton() {
    }

    private static SimpleSingleton instance = new SimpleSingleton();

    public static SimpleSingleton getInstance() {
        return instance;
    }
}    
```

以上的这种实现单例类的方法俗称为：饿汉式，因为每次使用前就已经先实例好对象了。

- 首先单例类的构造函数必须为私有的，只有这样才能确保单例对象不会在系统其他代码内被实例。
- 其次instance成员变量和getInstance()方法必须是static的。

缺点：

- 无法进行延迟加载
- 假设创建单例对象的过程很慢，而instance成员变量是由static定义的，所以在JVM加载单例类时，单例对象就会被创建，如果这时系统有地方使用到这个单例类的静态方法的话，不管是否使用到单例对象，该对象都会被实例化。

🌰例子：

```java
public class SimpleSingleton {

    private SimpleSingleton() {
        System.out.println("SimpleSingleton has been created");
    }

    private static SimpleSingleton instance = new SimpleSingleton();

    public static SimpleSingleton getInstance() {
        return instance;
    }

    public static String createString() {
        return "Hello World!";
    }
}

public static void main(String[] args) {
        System.out.println(SimpleSingleton.createString());
    // SimpleSingleton has been created
    // Hello World
}
```

可以看出执行createString()这个方法，即使没有调用getInstance()，单例对象还是会被创建出来。

因此就需要引入延迟加载机制了。



##### 2.2. 使用延迟加载实现单例(懒汉式以及静态内部类)

以下为引入延迟加载机制的单例类(俗称懒汉):

```java
public class LazySingleton {

    private LazySingleton() {
        System.out.println("Singleton has been created");
    }

    private static LazySingleton instance = null;

    public static synchronized LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }
}
```

来分析下优缺点,优点: 

1.确保了系统启动时没有额外的负载.

2.getInstance()使用了同步,在多线程的环境下,不会导致多个实例被创建.

缺点:

- 因为使用了同步关键字(synchronized),所以系统的性能会有所下降.

并不能因为引入了延迟加载而降低了系统的性能,所以对以上的版本进行改良:

```java
public class StaticSingleton {

    private StaticSingleton() {
        System.out.println("StaticSingleton has been created");
    }

    private static class SingletonHolder {
        private static StaticSingleton instance = new StaticSingleton();
    }

    public static StaticSingleton getInstance() {
        return SingletonHolder.instance;
    }
}
```

优点:

- 在这个实现中,单例模式使用了内部类来维护单例的实例,当StaticSingleton被加载时,内部类并不会被初始化,因此可以确保当StaticSingleton这个类被载入JVM时,不会初始化实例类.
- getInstance()方法被调用时,才会加载SingletonHolder,从而初始化实例类.
- 实例的建立是在类加载时完成的,所以天生对多线程友好,并不需要synchronized.

##### 2.3扩展延伸:

1.为什么内部类不会被JVM初始化?

2.为什么类加载时完成,天生对多线程友好?



#### :key:3. 问题

***

虽说以上的实现方法是已经比较安全的了,但还是会有缺点的,单例类会出现的问题大概有以下几个:

1. 线程安全问题.
2. 序列化与反序列化问题
3. 反射(通过暴力反射使用私有构造方法产生多个实例)



##### 3.1 序列化与反序列化

多线程以上的实现方法已经能解决问题了.所以直接来看下序列化与反序列化的问题:

序列化与反序列化能破坏单例.就拿最简单的饿汉式来举例:

```java
public class SerSingleton implements Serializable {

    private SerSingleton(){
        System.out.println("Singleton is created");
    }

    private static SerSingleton instance = new SerSingleton();

    public static SerSingleton getInstance() {
        return instance;
    }

    public static void createString() {
        System.out.println("createString in Singleton");
    }
}


public static void main(String[] args) throws Exception{
        SerSingleton s = null;
        SerSingleton s1 = SerSingleton.getInstance();

        FileOutputStream fos = new FileOutputStream("Singleton.txt");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(s1);
        oos.flush();
        oos.close();
        FileInputStream fis = new FileInputStream("Singleton.txt");
        ObjectInputStream ois = new ObjectInputStream(fis);
        s = (SerSingleton) ois.readObject();
        System.out.println(s);
        System.out.println(s1);
        System.out.println(s == s1);
        //false
}
```

测试结果说明单例类产生了不同的实例了,但如果加上这段代码的话:

```java
public class SerSingleton implements Serializable {

    private SerSingleton(){
        System.out.println("Singleton is created");
    }

    private static SerSingleton instance = new SerSingleton();

    public static SerSingleton getInstance() {
        return instance;
    }

    public static void createString() {
        System.out.println("createString in Singleton");
    }
//------------------------------------
    private Object readResolve() {
        return instance;
    }
//------------------------------------    
}
```

测试结果就会变成true了,事实上,在实现了私有的readResolve()方法后,readObject()已经形同虚设了,它直接使用readResolve()替换了原本的返回值.



##### 3.1.2扩展延伸:

序列化与反序列化(readResolve()?)究竟反序列化的过程中发生了什么导致了单例被破坏了?

对象的序列化过程通过ObjectOutputStream和ObjectInputputStream来实现的.因此来看看ObjectInputStream的readObject方法.

调用栈: ObjectInputStream.readObject() ---> readOrdinaryObject().主要是在readOrdinaryObject()这个方法里.

```java
private Object readOrdinaryObject(boolean unshared)
        throws IOException
    {
        if (bin.readByte() != TC_OBJECT) {
            throw new InternalError();
        }

        ObjectStreamClass desc = readClassDesc(false);
        desc.checkDeserialize();

        Class<?> cl = desc.forClass();
        if (cl == String.class || cl == Class.class
                || cl == ObjectStreamClass.class) {
            throw new InvalidClassException("invalid class descriptor");
        }

        Object obj;
        try {
            obj = desc.isInstantiable() ? desc.newInstance() : null;
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }
}        
```

其中这部分代码:

```java
Object obj;
        try {
            //--------------------------------------------
            obj = desc.isInstantiable() ? desc.newInstance() : null;
            //--------------------------------------------
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }
```

> `isInstantiable`：如果一个serializable/externalizable的类可以在运行时被实例化，那么该方法就返回true。针对serializable和externalizable我会在其他文章中介绍。
>
> `desc.newInstance`：该方法通过反射的方式调用无参构造方法新建一个对象。

- 为什么序列化能破坏单例呢?

**因为序列化会通过反射调用无参构造方法实例一个新的对象.**

- 如何防止序列化与反序列化破坏单例?

解决方案: 上面也有提到过,只需要在Singleton类中定义readResolve方法就能解决该问题.

```java
public class SerSingleton implements Serializable {

    private SerSingleton(){
        System.out.println("Singleton is created");
    }

    private static SerSingleton instance = new SerSingleton();

    public static SerSingleton getInstance() {
        return instance;
    }

    public static void createString() {
        System.out.println("createString in Singleton");
    }

    private Object readResolve() {
        return instance;
    }
}
```

- 那么为什么只需定义该方法就能实现不破坏单例呢?

那就需要结合源码来分析了:

```java
if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            Object rep = desc.invokeReadResolve(obj);
            if (unshared && rep.getClass().isArray()) {
                rep = cloneArray(rep);
            }
            if (rep != obj) {
                handles.setObject(passHandle, obj = rep);
            }
}
```

`hasReadResolveMethod`:如果实现了serializable 或者 externalizable接口的类中包含`readResolve`则返回true

`invokeReadResolve`:通过反射的方式调用要被反序列化的类的readResolve方法。

以上的资料参考自:http://www.hollischuang.com/archives/1144.



##### 3.2反射

在代码中仍然可以通过反射调用私有的构造函数来产生多个实例,那怎么防止呢?

可以通过在私有构造函数中判断:

```java
public class StaticSingleton {

    private StaticSingleton(){
        System.out.println("StaticSingleton has been created");
        if (SingletonHolder.instance != null) {
            throw new RuntimeException("The instance is Singleton");
        }
    }

    private static class SingletonHolder {
        private static StaticSingleton instance = new StaticSingleton();
    }

    public static StaticSingleton getInstance() {
        return SingletonHolder.instance;
    }
}
```

这样就能防止通过反射来创建多个实例了.



#### :telescope:4. 其他实现单例类的方法

总的来说,单例类有5种实现方法.上面大概介绍了3种单例类的实现方法.包括:饿汉,饥汉,静态内部类.还有2种方法分别为:

##### 4.1 枚举

- 枚举是目前为止实现单例类很好的方式,为什么这么说,下面仔细地分析:

##### 4.1.1 枚举单例写法简单:

- "双重校验锁"方式和枚举方式实现单例进行对比

双重校验锁:

```java
public class DoubleLockCheckSingleton {

    private DoubleLockCheckSingleton(){
        System.out.println("Singleton has been created");
    }

    private static DoubleLockCheckSingleton singleton = null;

    public static  DoubleLockCheckSingleton getInstance() {
        if (singleton == null) {
            synchronized (DoubleLockCheckSingleton.class) {
                if (singleton == null) {
                    singleton = new DoubleLockCheckSingleton();
                }
            }
        }
        return singleton;
    }
}
```

枚举实现单例:

```java
public enum EnumSingleton {
    instance
}
```

相比之下双重校验锁的代码会显得十分臃肿,为了解决线程安全问题,但还是无法解决反序列化的问题.



##### 4.1.2 枚举可以解决线程安全问题

如果使用非枚举的方式实现单例类,都要自己保证线程的安全,那为什么枚举就不需要解决线程问题呢?

> 原因是底层做了线程安全方面的保证.

######   4.1.2.1 枚举怎样实现线程安全

- 要想知道怎么实现,那就需要撸源码了.先来看看枚举反编译后的代码:

```java
public final class EnumSingleton extends Enum
{

    public static EnumSingleton[] values()
    {
        return (EnumSingleton[])$VALUES.clone();
    }

    public static EnumSingleton valueOf(String s)
    {
        return (EnumSingleton)Enum.valueOf(singleton/EnumSingleton, s);
    }

    private EnumSingleton(String s, int i)
    {
        super(s, i);
    }

    public static final EnumSingleton instance;
    private static final EnumSingleton $VALUES[];

    static 
    {
        instance = new EnumSingleton("instance", 0);
        $VALUES = (new EnumSingleton[] {
            instance
        });
    }
}
```

1. 首先如果使用了enum这个关键字,虚拟机会帮我们自动继承Enum这个类,并且该类是final不能被继承的.
2. 类中的所有成员变量均为static类型的,因为static类型的属性会在类被加载之后初始化.当一个Java类第一次被真正使用到的时候静态资源被初始化,Java类的加载和初始化过程都是安全的.

扩展延伸:

[Java类的加载,链接和初始化](http://www.hollischuang.com/archives/201)

[深度分析Java的ClassLoader机制（源码级别）](https://www.hollischuang.com/archives/199)



##### 4.1.3 枚举能自己处理序列化

在枚举之前所有的单例模式都有一个比较大的问题,就是一旦实现了Serializable接口之后,就不再是单例的了.

> 因为,每次调用readObject()方法都会返回一个新创建出来的对象.

而枚举**为了保证枚举类型像Java规范中所说的那样，每一个枚举类型极其定义的枚举变量在JVM中都是唯一的，在枚举类型的序列化和反序列化上，Java做了特殊的规定**:

> 在序列化的时候Java仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过java.lang.Enum的valueOf方法来根据名字查找枚举对象。同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法。

```JAVA
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                                String name) {
        T result = enumType.enumConstantDirectory().get(name);
        if (result != null)
            return result;
        if (name == null)
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException(
            "No enum constant " + enumType.getCanonicalName() + "." + name);
}
```

可以跟踪enumConstantDirectory()这个方法看:

```java
Map<String, T> enumConstantDirectory() {
        if (enumConstantDirectory == null) {
            T[] universe = getEnumConstantsShared();
            if (universe == null)
                throw new IllegalArgumentException(
                    getName() + " is not an enum type");
            Map<String, T> m = new HashMap<>(2 * universe.length);
            for (T constant : universe)
                m.put(((Enum<?>)constant).name(), constant);
            enumConstantDirectory = m;
        }
        return enumConstantDirectory;
    }
    private volatile transient Map<String, T> enumConstantDirectory = null;

//*************************************************************************************************
T[] getEnumConstantsShared() {
        if (enumConstants == null) {
            if (!isEnum()) return null;
            try {
                final Method values = getMethod("values");
                java.security.AccessController.doPrivileged(
                    new java.security.PrivilegedAction<Void>() {
                        public Void run() {
                                values.setAccessible(true);
                                return null;
                            }
                        });
                @SuppressWarnings("unchecked")
                T[] temporaryConstants = (T[])values.invoke(null);
                enumConstants = temporaryConstants;
            }
            // These can happen when users concoct enum-like classes
            // that don't comply with the enum spec.
            catch (InvocationTargetException | NoSuchMethodException |
                   IllegalAccessException ex) { return null; }
        }
        return enumConstants;
    }
//****************************************************************************************
public static EnumSingleton[] values()
{
        return (EnumSingleton[])$VALUES.clone();
}
// 克隆后的引用是同一个对象来的.
```

其实到最后会用反射的方式调用enumType的values方法返回单例对象.所以JVM对序列化是有保证的.

参考自:

[知识星球-单例模式专题](https://wx.zsxq.com/dweb/#/index/142111152552)

[为什么我墙裂建议大家使用枚举来实现单例](http://www.hollischuang.com/archives/201)

[深度分析Java的枚举类型—-枚举的线程安全性及序列化问题](https://www.hollischuang.com/archives/197)



##### 4.2 双重锁校验

上面与枚举实现单例比较已经把代码贴出来了,缺点是为了保证线程安全,并且在线程安全和锁粒度之间作权衡,代码会显得复杂臃肿.

```java
public class DoubleLockCheckSingleton{

    private DoubleLockCheckSingleton(){
        System.out.println("Singleton has been created");
    }

    private static DoubleLockCheckSingleton singleton = null;

    public static  DoubleLockCheckSingleton getInstance() {
        if (singleton == null) {
            synchronized (DoubleLockCheckSingleton.class) {
                if (singleton == null) {
                    singleton = new DoubleLockCheckSingleton();
                }
            }
        }
        return singleton;
    }
}
```

现在来分析为什么要这样写(判断了两次):

- 第一次校验:由于单例只需要创建一次对象,对于之后的每次调用就可以直接返回对象,大大地提高了性能,不需要进入同步代码块里去竞争锁了.
- 第二次校验:如果没有第二次校验的话,假设有两个线程,线程t1经过第一次判断后,对象为null,此时线程t2获得了CPU的执行权,也经过第一次判断后,并实例了对象,此时线程t1重新拿回CPU的执行权.那么就会产生了多个实例的情况.

但其实还隐藏了一个问题:private static volatile DoubleLockCheckSingleton singleton=null;需要加volatile关键字，否则会出现错误

> 原因是因为JVM指令重排优化的存在.

```java
singleton = new DoubleLockCheckSingleton();
```

这一步可分为3个操作:

1. 分配内存空间
2. 初始化对象
3. instance引用指向内存空间

线程A执行1、3后让出cpu，此时还未执行2，别的线程拿到cpu，发现instance不为null，直接返回使用，就会有问题，因为instance还未初始化。

因此需要加上volatile关键字来防止指令重排.

```java
private static volatile DoubleLockCheckSingleton singleton = null;
```



##### 4.3 不使用synchronized和lock,实现一个线程安全的单例

枚举,静态内部类以及饿汉实现单例的真正原理都是借助了ClassLoader的线程安全机制.

- 饿汉

​        是通过定义静态的成员变量,以保证单例对象在类初始化的时候被实例化.但类的初始化是由ClassLoader完成的

- 静态内部类

​       原理跟饿汉的差不多,单例类被装载的时候,instance不一定被初始化.因为SingletonHolder类没有被主动使用,只有通过调用getInstance方法时,才会去装载SingletonHolder类,从而被实例化.

- 枚举

  枚举上面已经说过了,说到底其实也是ClassLoader的线程安全机制.

结论:以上三种都是基于ClassLoader的线程安全机制来实现的

**为什么ClassLoader是线程安全的呢?**

> 因为在类加载的方法,loadClass 整个过程都使用了synchronized这个关键字.

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

那究竟不使用synchronized和lock怎么实现线程安全的单例模式呢?

`CAS`,在JDK1.5 中新增`java.util.concurrent(JUC)`就是建立在CAS之上的.相对于`sychronized`这种阻塞算法,CAS是一种非阻塞算法的常见实现.

- 借助CAS(AtomicReference)实现单例模式:

```java
public class CasSingleton {

    private static final AtomicReference<CasSingleton> SINGLETON = new AtomicReference<>();

    private CasSingleton(){}

    public static CasSingleton getInstance() {
        while(true) {
            CasSingleton current = SINGLETON.get();
            if (current != null) {
                return current;
            }
            current = new CasSingleton();
            if (SINGLETON.compareAndSet(null, current)) {
                return current;
            }
        }
    }
}    
```

- 优点:CAS是一种基于忙等待的算法,依赖底层硬件的实现,相对于锁它没有切换线程和阻塞的额外消耗.可以支持比较大的并行度.

- 缺点: 如果忙等待一直执行不成功(在死循环中),会对CPU造成比较大的执行开销.


**参考资料:**

[单例模式-双重校验锁](https://www.cnblogs.com/renyuanwei/p/9203088.html)

[单例双重锁线程不安全](https://blog.csdn.net/u013212958/article/details/81735809)

[不使用synchronized和lock,如何实现一个线程安全的单例](https://www.hollischuang.com/archives/1860)

[不使用synchronized和lock,如何实现一个线程安全的单例(2)](https://www.hollischuang.com/archives/1866)