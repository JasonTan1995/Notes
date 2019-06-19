### PreparedStatement 为什么能够防止sql注入?

#### 1. PreParedStatement的预编译与防sql注入

我们一直以来都知道`PreParedStatement`有预编译以及防`sql`注入的功能，但背后的原理是怎样做到的呢？

举个例子：select * from user where id = ? 假设使用的是`PreparedStatement`在进行参数拼接的时候会使用`?`占位符。

这样做有什么好处呢？为什么它这样做就能预防`sql`注入呢？是因为`sql`语句在程序运行前已经进行了预编译，在程序运行时第一次操作数据库之前，`sql`语句已经被数据哭分析，编译和优化，对应的执行计划也会缓存下来并允许数据库以参数化的形式进行查询，当运行时动态地把参数传给`PreParedStatment`时，即使参数中有敏感字符如or‘1=1’，数据库会作为一个参数来处理(对敏感字符进行转义)，而不是作为`sql`指令。

由于已经进行了预编译，因此执行效率也是非常让人满意的。

#### 参考资料

[预处理prepareStatement是怎么防止sql注入漏洞的？](https://www.cnblogs.com/yaochc/p/4957833.html)
