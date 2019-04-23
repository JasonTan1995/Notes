#### Java MethodHandle

​       Java7在JSR 292中增加了对动态类型语言的支持，使得java也可以像C语言那样将方法作为参数传递。

​       在java.lang.invoke包中MethodHandle作用类似于反射中的Method类，但它比Method类要更加灵活和轻量级。

​       Reflection是java api层面的反射调用，而MethodHandle则是jvm层面支持调用。因此Reflection是重量级，MethodHandle则是轻量级的。下面来看看怎么使用

##### 通过MethodHandle进行方法调用一般需要：

1. 创建MethodType对象，指定方法的签名（即方法参数以及方法返回值的类型）。
2. 在MethodHandles.Lookup中查找类型为MethodType的MethodHandle；
3. 传入方法参数并调用MethodHandle.invoke或者MethodHandle.invokeExact方法。

##### MethodType

​    通过该对象设置方法的返回值以及参数列表中的类型。

​    第一个参数是返回类型，后面的剩余参数是方法的参数类型。

```java
MethodType.methodType(Class<?>class,Class<?>class1);
```

##### Lookup

​     通过findXX方法可以得到相应的MethodHandle。

##### invoke

​     invokeExact:调用此方法与直接调用底层方法一样，需要做到参数类型精确匹配；

​     invoke:参数类型松散匹配，通过asType自动适配；

示例代码：

```java
public class MethodHandleTest {

    public String toString(String s){
        return "Hello"+s+"MethodHandle";
    }


    public static void main(String[] args) {
        MethodHandleTest mht = new MethodHandleTest();
        MethodHandle mh = getString();
        try {
            String o = (String) mh.invokeExact(mht, "ssss");
            System.out.println(o);
        }catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        
        MethodHandle methodHandle = mh.bindTo(mht);
        try {
            String ssss = (String) methodHandle.invokeWithArguments("ssss");
            System.out.println(ssss);
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
    }



    public static MethodHandle getString(){
        MethodType mt = MethodType.methodType(String.class,String.class);
        MethodHandle mh = null;
        try {
            mh = MethodHandles.lookup().findVirtual(MethodHandleTest.class,"toString",mt);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return mh;
    }
}
```

