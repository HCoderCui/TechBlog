# Different Between AsyncAppender and AsyncLogger
### 背景
公司的日志框架采用主流的slf4j+log4j2的组合。但是，大多数项目中log4j2停留在2.1版本，并且只是使用AsyncAppender异步输出日志。从[官方的文档](http://logging.apache.org/log4j/2.x/performance.html)中看到，引入了AsyncLogger后，log4j2在吞吐量和延时的表现有了质的飞跃。因此，本文力争对这两者，特别是AsyncLogger做一个深入的研究，望请赐教。
### 简单对比
这两者都能做到日志的异步输出，应用程序调用*Log4jLogger*的方法，主要是做了日志级别的过滤、信息参数化以及将信息包装成一个*log event*即可返回，由后台线程将*event*放入*queue*。两者性能差异的主要决定因素在于*queue*的选择上，*AsyncAppender*采用Java的*Blocking Queue*——*ArrayBlockingQueue*，而*AsyncLogger*则是基于[LMAX Disruptor(无锁的数据结构)](http://lmax-exchange.github.io/disruptor/)。
在多线程环境下，假定*log event queue*足够大，*AsyncLogger*的总吞吐量(*Total Throughput*)可以随线程数的增加而增加，反观*ArrayBlockingQueue*，线程增加导致锁竞争的增大，总吞吐量保持不变，甚至有所下降，意味着单个线程的吞吐量下降了。
当然，一旦*queue*满了，应用程序线程也会阻塞，直到*queue*有空间为止，所以要控制好两者的线程数量。
#### 使用
#### 实现
### 原理
#### AsyncAppender
#### AsyncLogger
### 总结
