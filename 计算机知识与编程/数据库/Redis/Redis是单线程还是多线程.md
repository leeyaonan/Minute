# Redis 到底是单线程还是多线程？

>[阅读原文](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247493806&idx=1&sn=c4988a38efd6555338615d932ce7522e&chksm=eb506d98dc27e48eab55be68da483102bc828704485a5740785afdf7bec525acbafc4697a4a8&mpshare=1&scene=1&srcid=0603YRV5OccKWtyCmfgLYmyw&sharer_sharetime=1591180628237&sharer_shareid=5dcf9269b47ffe68881e9bd2c8050254&key=0b622c5b94cbe1fa1c97312c1183154c3acad66fba169bae7ecf66e1921d92505f2e85800851bb4e6a4cc62496469b1864ab179209aebe30a2b279304b997c31c03ae4bec3ebac1438f1c557180f1ec1&ascene=1&uin=MTk5MjAzMzY2NQ%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AXJgaVhP6ugkyzQgIUOFSJQ%3D&pass_ticket=tOMLA1dSntxJGb2mQjkr2GFaaQXdTeNIq6xiR4Br5qZY7whqfQ1HSWncLNDnECBn)

我们习惯上称Redis是单线程的，但是这个问题要从多个方面回答，仅仅说Redis是单线程是不准确的。

## Redis的单线程是指什么？

我们所熟知的 Redis 确实是单线程模型，指的是执行 Redis 命令的核心模块是单线程的，而不是整个 Redis 实例就一个线程，Redis 其他模块还有各自模块的线程的。



>Redis基于Reactor模式开发了网络事件处理器，这个处理器被称为文件事件处理器。它的组成结构为4部分：多个套接字、IO多路复用程序、文件事件分派器、事件处理器。
>
>因为文件事件分派器队列的消费是单线程的，所以Redis才叫单线程模型。
>
>参考：https://www.jianshu.com/p/6264fa82ac33

## Redis不仅仅是单线程

一般来说 Redis 的瓶颈并不在 CPU，而在内存和网络。如果要使用 CPU 多核，可以搭建多个 Redis 实例来解决。

其实，Redis 4.0 开始就有多线程的概念了，比如 Redis 通过多线程方式在后台删除对象、以及通过 Redis 模块实现的阻塞命令等。

来源官方的解释：



如果你能说到这里，对 Redis 单/多线程的理解也有你自己更多的认识了。

另外，前些天 [Redis 6](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247493693&idx=2&sn=72cad8c4e5b996903a4d131662ff9dc9&scene=21#wechat_redirect) 正式发布了，其中有一个是被说了很久的多线程IO：

这个 Theaded IO 指的是在网络 IO 处理方面上了多线程，如网络数据的读写和协议解析等，需要注意的是，执行命令的核心模块还是单线程的。

所以，你要是再把 Redis 6.0 网络处理多线程这块回答上了，你也不至于 "请回" 了。

之前有的人在后台和我杠精说：**Redis 6 不是还没发布吗？**

Redis 6 Beta 版本多线程这个说了多久了，作为一个程序员，如果这个还不能 get 到的话，那就有点 OUT 了，如果确实没听说还好，如果听说了，还要和我杠精，我就无言以对了，对于新技术的发展和学习不就是我们和面试官的谈资吗？

## 为什么网络处理要引入多线程

之前的段落说了，Redis 的瓶颈并不在 CPU，而在内存和网络。

内存不够的话，可以加内存或者做数据结构优化和其他优化等，但网络的性能优化才是大头，网络 IO 的读写在 Redis 整个执行期间占用了大部分的 CPU 时间，如果把网络处理这部分做成多线程处理方式，那对整个 Redis 的性能会有很大的提升。

网上也有对 Redis 单/多线程情况下的 get/set 操作性能做了对比：

> 参考：blog.csdn.net/weixin_45583158/article/details/100143587

从上面的性能测试图来看，多线程的性能几乎是单线程的两倍了，从该文章来看，这个只是简单的针对多线程性能的验证，并没有做很多严谨的测试，不能作为线上指标参考。

但可以知道的是，Redis 在网络处理方面上了多线程确实会让 Redis 性能上一个新台阶，不过 Redis 6.0 刚发布，不可能有企业马上上生产环境，可能还需要一段时间的优化和验证，我们再期待吧。

最后，目前最新的 6.0 版本中，IO 多线程处理模式默认是不开启的，需要去配置文件中开启并配置线程数，有兴趣的研究下吧。