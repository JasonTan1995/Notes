### JVM 内存结构

`Hotspot VM`的垃圾回收都采用分代回收的算法.分代算法:针对不同对象的生命周期有不同的回收方式,以便提高效率.

`Hotspot VM`将内存划分为不同的物理区域,`JVM`内存主要由新生代,老年代,永久代组成.

新生代(`Young Generation`):大多数对象会在新生代中创建,其中很多对象的生命周期是很短的.每次新生代的垃圾回收(`Minor GC`)后只有少量对象存活,所以选用的是复制算法,只需要少量的复制成本就可以完成回收了.

> 新生代内又会分为三个区:一个`Eden`区,两个`Survivor`区,大部分的对象会在`Eden`区中生成.当`Eden`满时,如果还有对象存活就会复制到两个`Survivor`区中,当`Survivor`满时对象存活而又未达到晋升条件就会被复制到另外一个`Survivor`中,对象每经历一个`Minor GC`,年龄就会加1,当达到晋升阀值时就会被放到老年代中.这个晋升阀值通过`MaxTenuringThreshold`设定,默认值为15.

- 关于"晋升阀值"问题:

  阀值设置的过大,会让原本应该晋升的对象一直停留在`Survivor`区,直到`Survivor`区溢出,一旦溢出发生:`Eden`和`Survivor`中的对象不再依据年龄晋升到老年代,会导致对象老化机制失效.

  阀值设置的过小:过早晋升即对对象不能在新生代充分被回收,大量短期对象被晋升到老年代,老年代的空间迅速增长会频繁的触发`Major GC`.分代失去了意义,`GC`性能严重被英雄.

  因为以上的两种情况,引入了动态年龄计算:`Hotspot`遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了`survivor`区的一半时，取这个年龄和`MaxTenuringThreshold`中更小的一个值，作为新的晋升年龄阈值

老年代(`Old Generation`):在新生代经历了N次垃圾回收后仍然存活的对象,就会被放到老年代.这个区域的对象存活率很高.老年代的垃圾回收(`Major GC`)使用的是标记-整理,整理-标记算法.新生代和老年代的垃圾回收称为`Full GC`

> `Major GC`会触发` Full GC`

永久代(`Prem Generation`):主要存放元数据,例如:Class,Method的元信息,与垃圾回收要回收的Java对象关系不大.

#### 常见的垃圾回收器

- 串行(`Serial`)回收器是一个单线程的回收器,简易,易实现,效率高.
- 并行(`ParNew`)回收器是`Serial`的多线程版本,可以充分利用CPU资源,减少回收时间.
- 吞吐量优先(`Parallel Scavenage`)回收器,侧重于吞吐量.
- 并发标记清除(`CMS, Concurrent Mark Sweep`)回收器是一种以获取最短回收停顿时间为目标的回收器,该回收器是基于标记-清除算法实现的.

#### 垃圾回收的触发条件

针对`HotSpot VM`的实现,它的GC其实准备分类只有两种:

- `Partial GC`: 并不收集整个GC堆的模式.
  - `Young GC`: 只收集`young gen`的GC.
  - `Old GC`:只收集`old gen`的GC.但只有CMS的concurrent collection是这个模式.
  - `Mixed GC`:收集整个`young gen`以及部分`old gen`的GC.只有G1有这个模式.
- `Full GC`:收集整个堆,包括`young gen`,`old gen`,`perm gen`等所有部分的模式.

按`HotSpot VM` 的` serial gc`的实现来看,触发条件是:

- `young gc`:当`young gen`中的`eden`区分配满的时候触发.
- `full gc`:当准备要触发一次`young gc`时,如果发现统计数据说之前`young gc`的平均晋升大小比目前`old gen`剩余空间大,则不会 触发`young gc`转为触发`full gc`.`System.gc()`也是默认触发`full gc`.

#### 参考资料:

[JVM系列-（八）CMS与G1](https://zhuanlan.zhihu.com/p/52544311)

[从实际案例聊聊Java应用的GC优化](https://tech.meituan.com/2017/12/29/jvm-optimize.html)

[Major GC和Full GC的区别是什么？触发条件呢？](https://www.zhihu.com/question/41922036)

