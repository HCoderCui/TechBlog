### 并发工具类 —— CountDownLatch CyclicBarrier Semaphore Exchanger

由java1.5引入，存在于java.util.concurrent包下。

#### 等待多个线程完成的CountDownLatch

1. 是什么

   CountDownLatch 允许一个或多个线程等待其他线程完成操作。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。

2. 如何工作

   CountDownLatch是通过一个计数器来实现的，计数器的初始值为需要执行线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

   CountDownLatch的构造器

   ​	public void CountDownLatch(int count) {...}

   **count 实际上就是闭锁需要等待的线程数量**。这个值只能被设置一次，而且CountDownLatch**没有提供任何机制去重新设置这个计数值**。

   主线程必须在启动其他线程后立即调用**CountDownLatch.await()**方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

   其他N 个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。这种通知机制是通过 **CountDownLatch.countDown()**方法来完成的；每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。

3. 使用场景

   1. **实现最大的并行性**：有时我们想同时启动多个线程，实现最大程度的并行性。例如，我们想测试一个单例类。如果我们创建一个初始计数为1的CountDownLatch，并让所有线程都在这个锁上等待，那么我们可以很轻松地完成测试。我们只需调用 一次countDown()方法就可以让所有的等待线程同时恢复执行。
   2. **开始执行前等待n个线程完成各自任务**：例如应用程序启动类要确保在处理用户请求前，所有N个外部系统已经启动和运行了。
   3. **死锁检测：**一个非常方便的使用场景是，你可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。

4. 其他方法

   * await(long time, TimeUnit unit): 这个方法等待特定时间后，就会不再阻塞当前线程。

#### 让一组线程在一个时间点上达到同步的CyclicBarrier

1. 是什么

   CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier），能让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。

2. 如何工作

   CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。当调用次数达到了parties，所有被阻塞的线程继续执行。

   CyclicBarrier还提供一个更高级的构造函数CyclicBarrier(int parties, Runnable barrierAction)，用于在线程到达屏障时，优先执行barrierAction，方便处理更复杂的业务场景。

3. 使用场景

   CyclicBarrier可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水。

4. 其他方法

   * CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier能处理更为复杂的业务场景，比如如果计算发生错误，可以重置计数器，并让线程们重新执行一次。
   * getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。
   * isBroken方法用来知道阻塞的线程是否被中断。比如以下代码执行完之后会返回true。

#### 控制并发线程数的Semaphore

1. 是什么

   Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源（控流）。就如红绿灯控制车流一样。

2. 如何工作

   构造器:

   ```java
   public Semaphore(int permits) {...}
   ```

   permits表示可用的许可证数量，也就是线程的最大并发数。

   首先线程使用Semaphore的acquire()获取一个许可证，使用完之后调用release()归还许可证。还可以用tryAcquire()方法尝试获取许可证。

3. 应用场景

   Semaphore可以用于做流量控制，特别公用资源有限的应用场景，比如数据库连接。假如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程并发的读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有10个，这时我们必须控制只有十个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。这个时候，我们就可以使用Semaphore来做流控。

4. 其他方法

   * int availablePermits() ：返回此信号量中当前可用的许可证数。
   * int getQueueLength()：返回正在等待获取许可证的线程数。
   * boolean hasQueuedThreads() ：是否有线程正在等待获取许可证。
   * void reducePermits(int reduction) ：减少reduction个许可证。是个protected方法。
   * Collection getQueuedThreads() ：返回所有等待获取许可证的线程集合。是个protected方法。

#### 两个线程进行数据交换的Exchanger

1. 是什么

   Exchanger（交换者）是一个用于线程间协作的工具类。Exchanger用于进行线程间的数据交换。它提供一个同步点，在这个同步点两个线程可以交换彼此的数据。这两个线程通过exchange方法交换数据， 如果第一个线程先执行exchange方法，它会一直等待第二个线程也执行exchange，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

​

​

​