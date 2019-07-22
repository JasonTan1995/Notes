## G1 (Garbage-First)收集器

------

G1收集器是当今收集器技术发展最前沿的成果之一，并且在JDK1.9 之后被设置为默认的垃圾收集器.

### 1. 特点

------

G1是一款面向服务端应用的垃圾收集器,与其他收集器相比,G1拥有如下的特点:

1. 并行并发

   G1能充分利用多CPU,多核环境下的硬件优势,使用多个CPU来缩短`STOP THE WORLD`停顿的时间,G1收集器仍然可以通过并发的方式让Java程序继续执行.

2. 分代收集

   分代概念在G1中依然得以保留,G1收集器不需要与其他收集器配合就能独立管理整个GC堆.

3. 空间整合

   G1从整体来看是基于`标记-整理`算法实现的收集器,从局部(两个`Region`之间)来看是基于复制算法实现的.不管是复制还是标记整理都意味着G1不会产生内存空间碎片,保证了收集后能提供规整的可用内存.

   > 该特性保证了程序的长时间运行,分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC.

4. 可预测的停顿

   G1能建立可预测的停顿时间模型,能让使用者明确指定在一个长度为M毫秒的时间片段内,消耗在垃圾收集上的时间不得超过N毫秒.



### 2. G1收集器中Java堆的内存布局

------

在G1之前的其他收集器进行收集的范围都是整个新生代或者老年代,而G1不再是这个了.G1收集器将整个Java堆划分为多个大小相等的独立区域(`Region`).虽然还保留新生代和老年代的概念,但新生代和老年代不再是物理隔离的了,因为它们都是一部分`Region`(不需要连续)的集合.

从传统的连续堆内存布局设计,逐步走向了物理上不连续但是逻辑上依旧连续的内存块.

![ZL29lq.md.png](https://s2.ax1x.com/2019/07/17/ZL29lq.md.png)

传统`Hotspot`堆的结构:

![ZOsFQf.png](https://s2.ax1x.com/2019/07/18/ZOsFQf.png)



### 3. 可预测的停顿时间模型

------

G1可以有计划地避免在整个Java堆中进行全区域的垃圾收集,那么到底是怎么有计划地避免呢?

G1会跟踪各个`Region`里面的垃圾堆积的价值大小,在后台会维护一个优先列表,每次根据允许的收集时间,优先回收价值最大的`Region`.这个也是G1名称的来由(`Garbage-First`).

哪么这种(Region划分内存空间以及有优先级的区域)回收方式有什么好处呢?

保证了G1收集器在有限的收集时间内可以获取尽可能高的收集效率.



### 4. G1收集器的回收方式

------

既然G1把堆中的内存分为多个`Region`,哪么垃圾收集是否会以`Region`为单位来进行垃圾收集呢?

但`Region`其实是不可能孤立的,因为一个对象被分配到某个`Region`中,它并不是只被本`Region`中的其他对象引用,而是可以与整个Java堆任意的对象发生引用关系的.那在进行可达性分析时判断对象是否存活时,还得扫描整个Java堆才能保证准确性, 如果回收新生代时也不得不同时扫描老年代的话,那么Minor GC的效率可能会下降不少.

哪么在G1收集器中是如何避免进行可达性分析时引起全堆扫描的呢?

在G1收集器中,Region之间的对象引用以及其他收集器中的新生代与老年代之间的对象引用,虚拟机都是使用`Remembered Set`来避免全堆扫描的.

1. G1中每个`Region`都有一个与之对应的`Remembered Set`,虚拟机发现程序在对Reference类型的数据进行写操作时,会产生一个`Write Barrier`(写屏障)暂时中断写操作,检查Reference引用的对象是否处于同一个`Region`之中(分代的话就是检查老年代中的对象是否引用了新生代中的对象).

1. 如果是,就会通过`CardTable`把相关引用信息记录到被引用对象所属的`Region`的`Remembered Set`之中.那么在进行内存回收时,在GC根节点的枚举范围中加入`Remembered Set`即可保证不引起全堆扫描也不会有遗漏.

`Remembered Set` ?

> `Remembered Set`是一种抽象的概念,`Remembered Set`是在实现部分垃圾收集(`Partial GC`)时用于记录**从非收集部分指向收集部分的指针的集合**的抽象数据结构.
>
> 分代式GC,通常会分为两代分别叫做`young gen`和`old gen`;通常能单独收集的只是`young gen`.此时`remembered set`记录的就是从`old gen`指向`young gen`的跨代指针.
>
> `Region collector`也是一种部分垃圾收集的方式,G1使用的就是该种方式,此时`remembered set`记录的是跨`region`的指针.
>
> `Remembered Set`是一个集合(set),所以不会包含重复.



### 5. G1 收集器的运作

------

如果不计算维护`Remembered Set`的操作,G1收集器的运作大致可划分为以下几个步骤:

- 初始标记(`Inital Marking`)

  该阶段需要`STOP THE WORLD`,但耗时很短.该阶段会标记`GCRoots`能直接关联到的对象.并且修改TAMS(Next Top  at Mark Start)的值,让下一阶段用户程序并发运行时,能在正确可用的`Region`中创建对象.

- 并发标记(`Concurrent Marking`)

  该阶段是从`GCRoots`开始对堆中对象进行可达性分析,找出存活对象,这阶段耗时会比较长.但能与用户程序并发执行.

- 最终标记(`Final Marking`)

  该阶段是为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分记录标记,虚拟机将这段时间对象变化记录在线程`Remembered Set Logs`里面,然后需要把`Remembered Set Logs`的数据合并到`Remembered Set`中.

  该阶段需要停顿线程,但可以并行执行.

- 筛选回收(`Live Data Counting and Evacuation`)

  该阶段会首先对各个`Region`的回收价值和成本进行排序,然后根据用户所期望的GC停顿时间来制定回收计划.

  这阶段也可以做到与用户程序并发执行.但因为只是回收一部分的`Region`,时间是用户可控制的,而且停顿用户线程将大幅提升收集效率.

![ZXtDk6.png](https://s2.ax1x.com/2019/07/18/ZXtDk6.png)

以上讲了一堆词语,现在来解释下:

上面提到过`Remembered Set`是用于记录从非收集部分指向收集部分的指针的集合.即不同`Region`的对象的引用的记录或者老年代到年轻代的引用的记录.而`CardTable`是一种`remembered set`.

在每个分区内部又被分成了若干个大小为512Bytes的`card`,标识堆内存最小可用粒度所有分区的卡片将会记录在全局卡片标中(`Global Card Table`)中,分配的对象会占用物理上连续的若干个卡片,当查找对分区对象的引用时便可通过记录卡片来查找该引用对象.每次对内存的回收,都是对指定分区的卡片进行处理.

- 每个`Region`都有自己的`cardTable`.
- 维护`remembered set`需要`mutator`线程在可能跨`Region`的引用的时候通知`collector`.这种方式通常叫做`write barrier`.
- 每个线程都会有自己的`remembered set log`.

![ZXtg6H.png](https://s2.ax1x.com/2019/07/18/ZXtg6H.png)

参考资料:

[名词链接贴](https://hllvm-group.iteye.com/group/topic/21468)4楼

[详解 JVM Garbage First(G1) 垃圾收集器](https://blog.csdn.net/coderlius/article/details/79272773)

[Garbage First G1收集器 理解和原理分析](https://liuzhengyang.github.io/2017/06/07/garbage-first-collector)

[Getting Started with the G1 Garbage Collector](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html)

