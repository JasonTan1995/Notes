#### Transient

Java中对象的序列化指的是将对象转换成以字节序列的形式来表示，这些字节序列包含了对象的数据和信息，一个序列化后的对象可以被写到数据库或文件中，也可用于网络传输，一般当我们使用缓存cache（内存空间不够有可能会本地存储到硬盘）或远程调用rpc（网络传输）的时候，经常需要让我们的实体类实现Serializable接口，目的就是为了让其可序列化.

transient 关键字

Java中transient关键字的作用,就是让某些被修饰的成员变量不被序列化.

什么情况下适用?

1. 类中的字段值可以根据其他字段推导出来,就好像三角形一样.有长度,宽度,面积.那么在序列化的时候面积这个属性就没有必要被序列化了.
2. 根据自己业务的需求,看哪些字段不想被序列化.

在HashMap中该关键字也被使用了多次.

```java
/**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;

    /**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * The number of key-value mappings contained in this map.
     */
    transient int size;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     */
    transient int modCount;
```

就拿modCount来举例,该字段主要用于判断HashMap是否被修改.但并不需要持久化它的值.

