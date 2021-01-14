__jdk线程池原理__

本文主要是对jdk源生线程池的源码注释的翻译，jdk版本为1.8.0_212.java.util.concurrent.ThreadPoolExecutor继承AbstractExecutorService,AbstractExecutorService实现了ExecutorService接口，ExecutorService继承自Executor接口。execute方法何时被执行？
execute方法的执行逻辑？


ExecutorService是使用几个池化的线程中的一个线程执行每一个被提交的任务，通常情况下是使用Executors的工厂方法配置的线程池。

线程池解决了两个不同的问题，这两个问题在执行大量的异步任务的时候通常会提供更高的性能。主要是因为减少了每个任务的调用开销，并且它们提供了一种限制和管理线程资源的方式，包括线程、执行一个任务集合的消耗。每一个ThreadPoolExecutor也提供一些基本的统计信息，比如完成的任务数量等。

为了在大量的上下文环境中是可用的，这个类提供了许多可调节的参数和扩展性的钩子方法。然而，程序员们被期望使用更方便实用的Executors的工厂方法,比如Executors.newCachedThreadPool（没有限定的线程池，自动线程管理），Executors.newFixedThreadPool(固定数量的线程池)和Executors.newSingleThreadExecutor(单个后台线程),这些方法已经预先对最常用的线程池使用场景进行了配置。另外，也可以手动配置线程池类的参数。
核心线程池数
最大线程池数
一个ThreadPoolExecutor实例会根据核心线程池数和最大线程池数自动调整线程池的大小。
当一个新的任务被提交到execute方法的时候，如果正在运行(应该是指没有被销毁)的线程的数量小于核心线程池数，即使线程池中有其他可用的线程，线程池也会创建一个新的线程处理这个请求。如果线程池数量大于核心线程池数，小于最大线程池数，只有当队列满的时候才会创建新的线程。如果将核心线程池数和最大线程池数设置为一样，创建的就是一个固定大小的线程池，如果将最大线程池数设置为一个本质上无上限的值，比如Integer.MAX_VALUE，创建的就是允许线程池容纳任意数量的并发任务。最典型的做法是，只在构造方法上设置核心线程池数和最大线程池数，但是也可以被setCorePoolSize和setMaximunPoolSize这两个方法动态的改变。

        构造方法

默认情况下，只有心的任务被提交的时候，核心线程才会被初始化创建和启动，但是这也可以被prestartCoreThread或者prestartAllCoreThread方法动态覆盖。当你创建一个队列非空的线程池的话，你可能会想要预先启动线程。

        创建新的线程

线程池中的新线程是使用ThreadFactory创建的。如果没有其他的特别指定，线程池的构造方法会使用Executors.defaultThreadFactory方法，创建的所有线程都是相同的线程组、默认的优先级（5），非守护线程状态。如果使用一个其他的线程工厂类，你可以指定线程的名字，线程组，优先级，守护状态等。如果线程工厂创建线程失败，执行器会继续工作，但是可能不能执行任何任务。线程应该处理“modifyThread”。如果工作线程或者其他使用这个线程池的线程不处理这个权限，服务可能会被降级：配置变更不会立马生效，shutdown线程池会停留在可能终止的状态，但不是完全终止。

        存活时间

如果线程池当前线程池数量多于核心线程池数，并且超过核心线程池数的线程处在空闲状态的时间超过指定的存活时间会被终止。在线程池并不是很活跃的时候，这提供了一种减少资源消耗的手段。如果线程池在稍后的时间变得更活跃了，新的线程会被创建。这个参数也可以使用setKeepAliveTime方法动态地改变。使用Long.MAX_VALUE可以有效的避免空闲线程被终止。默认情况下，keep-alive 策略仅仅适用在线程池数量多于核心线程池数的场景，但是allowCoreThreadTimeOut方法可以把这种超时策略应用在核心线程上，只要keepAliveTime的值是非零的。

	    队列

任何实现了BlockingQueue接口的类都可以被用来转发和保存提交的任务。队列的使用和线程池的数量相关。
如果正在运行的线程池数量少于核心线程池数，执行器会优先创建一个新的线程，而不是将任务入队。
如果超过核心线程池数的线程在运行，执行器总是会优先将请求入队列，而不是创建一个新的线程。
如果一个请求不能被加入到队列中，线程池会创建一个新的线程，直到达到最大线程池数。如果达到最大线程池数，这个任务会被拒绝。

	    线程排队的三种策略

SynchronousQueue、LinkedBlockingQueue、ArrayBlockingQueue
直接转发。
工作队列的一个号的默认选择就是SynchronousQueue，他会不保存任务并直接将提交的任务转发给执行线程。当没有线程是直接可用的来执行任务的时候，视图讲一个任务保存到队列中会失败，所以新的线程会被创建。这种策略在保存一个可能存在内部依赖的请求集合的时候，可以避免死锁。直接转发通常要求没有限制的最大线程池数量，避免新提交的任务被拒绝。当任务提交持续提交的比处理的更快的时候，这反而导致无限制的线程增长的可能性。

无限制的队列。
使用一个无界队列(比如没有预先定义容量的LinkedBlockingQueue)，在所有的核心线程都不在空闲状态时，会引起新提交的任务在队列中等待。但是不会有比核心线程池数更多的线程被创建(并且最大线程池数量的值没有任何影响)。这种策略比较适合各个任务是完全独立完成的，所以任务之间不会相互影响其他的执行，比如在一个网页服务器里面。虽然这种队列方式在处理短暂的请求并发的时候很有用，但是在任务到达持续比任务处理要快的时候会导致无限制的工作队列增长

有界队列
有界队列，比如ArrayBlockingQueue，在使用有限的最大线程池数量的时候可以防止资源耗尽，但是对线程的协调和控制会更复杂一些。队列尺寸和最大线程池数量可以折中考虑，使用较大的队列尺寸和较小的线程池数量可以使CPU占用率、操作系统资源，上下文切换的开销降到最低，但是会导致人为地降低系统吞吐量。如果任务可以频繁地阻塞(比如IO密集型)，系统可以为线程调度更多的时间。使用较小的队列尺寸和较大的线程池数量，这样可以保证较高的CPU使用率但是可能会碰到不可接受的调度开销，也会降低系统吞吐量。

	    拒绝策略
当执行器关闭或者使用有限的队列尺寸和最大线程池数量的时候，新的任务提交给execute方法会被拒绝执行。两种场景下，execute方法都会调用RejectedExecutionHandler.rejectedExecution(Runnable, ThreadPoolExecutor)方法。线程池提供四种预先定义的拒绝策略。
ThreadPoolExecutor.AbortPolicy。处理器在拒绝的时候回直接抛出运行时异常。
ThreadPoolExecutor.CallerRunsPolicy调用execute的线程处理被拒绝的任务。这种方式提供一种简单的新的降低新的任务被提交的概率的反馈控制机制。
ThreadPoolExecutor.DiscardPolicy不能被执行的任务简单的丢弃。
ThreadPoolExecutor.DiscardOldestPolicy如果执行器没有关闭，工作队列头部的任务被丢弃，然后执行被拒绝的任务重新尝试执行(重试的时候可能会再次失败，然后导致这个过程重复)
可以定义并使用其他的RejectedExecutionHandler类，这样操作要求一些考虑，特别是当拒绝策略被设计只在特定的线程池容量和工作队列策略的时候工作的时候。

		钩子方法
这个类提供了protected 可以被覆盖的方法，beforeExecute和afterExecute，在每一个任务被执行前和执行后会被调用。这些方法可以被用来操作执行环境，比如重新初始化ThreadLocals,收集统计信息，或者添加日志等。另外，terminated方法也可以被覆盖用来执行任何需要在执行器完全终止后进行的特殊的处理。
如果钩子方法或者回调方法抛出异常，内部的工作线程会失败并且突然终止。

		队列维护
getQueue允许获得队列进行监控和调试。强烈建议不要使用这个方法进行任何其他的用途。当大量的队列任务被取消的时候，remove和purge这两个方法可用于帮助回收队列存储空间。

		最后
程序中不再被引用的线程池并且没有存活的线程会被自动shutdown。如果希望确保没有被引用的线程池在调用者忘记调用shutdown的时候仍然能够被回收，必须保证没有被使用的线程最终都会死亡，可选的方案为设置合适的keep-alive times，使用下界为0的核心线程数或者设置allowCoreThreadTimeOut。

		扩展示例
多数这个类的扩展会覆盖一个或者多个protected钩子方法。比如一个添加了pause/resume特征的子类
class PausableThreadPoolExecutor extends ThreadPoolExecutor {
    private boolean isPaused;
    private ReentrantLock pauseLock = new ReentrantLock();
    private Condition unpaused = pauseLock.newCondition();

    public PausableThreadPoolExecutor(...) { super(...); }
     
    protected void beforeExecute(Thread t, Runnable r) {
      super.beforeExecute(t, r);
      pauseLock.lock();
      try {
        while (isPaused) unpaused.await();
      } catch (InterruptedException ie) {
        t.interrupt();
      } finally {
        pauseLock.unlock();
      }
    }
     
    public void pause() {
      pauseLock.lock();
      try {
        isPaused = true;
      } finally {
        pauseLock.unlock();
      }
    }
     
    public void resume() {
      pauseLock.lock();
      try {
        isPaused = false;
        unpaused.signalAll();
      } finally {
        pauseLock.unlock();
      }
    }
  }



主要属性

ctl

主要的线程控制状态，是一个包含了两个概念上属性的atomic integer。workCount限制为2^29 -1，workCount是允许start而不能stop的工作线程，这个数据可能暂时和实际存活的线程数量不一致，比如线程工厂创建线程失败，线程终止前仍然在记账。用户可见的线程池数量是当前工作线程集合的数量。5个runState，和线程的5种状态不一样。

workQueue

用来保存任务和给出到工作线程的队列。不要求workQueue.poll()返回null表示workQueue.isEmpty()，仅仅依赖isEmpty()方法来看队列是否为空(比如是否决定从SHUTDOWN过渡到TIDYING的时候)。

mainLock

持有工作线程集合的访问并且和记账相关。使用某种形式的并发集合，结果显示使用锁更好。这序列化了中断空闲的工作线程，避免了中断风暴，尤其是在shutdown的时候。否则已经存在的线程会并发地中断还没有中断的线程，也简化了一些相关的记账统计，比如最大线程池数量等。在shutdown和shutdownNow的时候仍然持有mainLock，为了保证在独立地检查试图中断和实际中断的时候工作线程集合是稳定的。

workers

包含所有工作线程的集合，只有在持有mainLock的时候才能访问。

termination

等待条件支持awaitTermination。

threadfactory

创建线程的工厂。所有的线程都是用这个工厂调用addWorker方法创建的。所有的调用者必须准备由于系统或者用户的限制策略导致addworker方法调用失败。即使这样，它也不会被当作一个错误。创建线程失败会导致新的任务被拒绝或者已经存在的任务阻塞在队列中。

主要方法描述
主要的线程池控制状态，是一个原子性的整数封装的两个概念性的属性，workerCount(表明活跃的线程数量)，runState(表明是否正在运行或者关闭等)

为了将这两个属性封装到一个int变量中，JDK限制workerCount最大为2^29-1(大约是500million)，而不是2^31-1(2billion)。如果这将来成为一个issue，这个int变量可以被改成一个AtomicLong类型，并且低位的shift/mask常量相应的发生变化。但是相对于使用AtomicLong，AtomicInteger会更简单、更快一些。

workerCount是被允许开启并且不能停止的线程的数量。他的值可能是暂时性的和实际存活的线程数量不一样，比如当一个线程工厂被调用来创建一个线程失败的时候，并且退出的线程在终止前仍然执行记账的时候。用户可见的线程池数量被记录为当前工作队列集合的数量。

runState提供主要的线程生命周期控制，取值为
RUNNING:	接受新的任务并且处理工作队列的任务
SHUTDOWN:	不接受新的任务，但是处理工作队列中的任务
STOP:		不接受新的任务，不处理工作队列中的任务并且中断正在执行的任务
TIDYING:	所有的任务停止后，workerCount为0，过渡到TIDYING状态的线程将会执行terminated()这个钩子方法
TERMINATED:	terminated()方法执行完成

The numerical order among these values matters, to allow
      ordered comparisons. The runState monotonically increases over
      time, but need not hit each state. The transitions are:
     
      RUNNING -> SHUTDOWN
         On invocation of shutdown(), perhaps implicitly in finalize()
      (RUNNING or SHUTDOWN) -> STOP
         On invocation of shutdownNow()
      SHUTDOWN -> TIDYING
         When both queue and pool are empty
      STOP -> TIDYING
         When pool is empty
      TIDYING -> TERMINATED
         When the terminated() hook method has completed

当线程状态变为TERMINATED的时候，在awaitTermination()方法中等待的线程将会返回。

检测到SHUTDOWN到TIDYING状态的转换并不像期望中的那么直截了当，因为队列在SHUTDOWN状态期间可能会从费控变为空或者从空变为非空，但是JDK只能在看到队列为空，workerCount为0(有时候可能还会需要再次检查)的的时候终止线程。

使用二进制位封装runState和workerCount成AtomicInteger类型的ctl，高三位表示runState，低29位表示workerCount。使用CAS操作来增加或者减少workerCount。

decrementWorkerCount()
这个方法使用CAS自旋，专门用来减少ctl属性中的workerCount值。它只会在线程突然终止的时候被调用(processWorkerExit)。其他减少workerCount值的方法在getTask方法中被执行。

workQueue
这个属性用来存储任务和转发任务到工作线程。JDK不要求workQueue.isEmpty()的时候workQueue.poll()方法必须返回null，所以仅仅只能依靠isEmpty()方法判断队列是否为空(比如在决定是否从SHUTDOWN状态转换为TIDYING状态的时候，JDK必须判断队列是否为空)。这是为了使用特殊目的的队列，比如DelayQueues允许poll()返回null，即使它可能在稍后delays失效返回非空。

mainLock
锁持有工作队列集合的访问和相关记录。当我们使用某种类型的并发集合，通常更适合使用锁。这种现象的原因是它序列化了interruptIdleWorkers，避免不必要的中断风暴，特别是在shutdown期间。否则现存的线程会并发地中断还没有被中断的线程。同时，它也简化了一些相关的最大线程数记录统计等。为了在单独检查权限准备中断或者实际上正在中断的时候保证工作线程集合是稳定的，我们也可以在shutdown和shutdownNow的时候持有mainLock锁。

workers
一个包含线程池中所有工作线程的集合。只有当持有mainLock的时候可以访问。

termination
支持awaitTermination的等待条件

largestPoolSize
跟踪最大获取的线程数量，只有在mainLock下可以访问。

completedTaskCount
已经完成的任务数量的计数器。只有在工作线程终止的时候更新。只有在持有mainLock的时候可以访问。

threadFactory
所有用户控制参数都被声明为volatiles，这样的话正在进行的行为都是基于最新的值，但是不需要锁，因为没有依赖于他们的内部不变量随着其他行为同步变化。
创建新的线程的工厂。所有的线程都是通过这个工厂的addWorker创建的。所有的调用者必须处理addWorker方法失败的场景，失败会返回系统或者用户定义的限制线程数量的策略。即使它不被看做是一个错误，创建线程失败可能导致被拒绝的新任务或者已经存在的任务停留在队列中。
即使面对OutOfMemoryError这样的错误，我们走的更远并且保存线程池不变量，这些错误可能在试图创建线程的时候被抛出来。由于需要在Thread.start的时候分配一个本地栈，这些错误更平常一些，或者用户希望执行清空线程池。可能会有足够可用的内存给清除代码在遇见另一个OutOfMemoryError错误前完成清除动作。

handler
当线程池饱和或者在执行的时候关闭会被调用

keepAliveTime
以纳秒为单位的空闲线程等待工作的超时时间

allowCoreThreadTimeOut
默认false，如果是false，即使是空闲的核心线程也会保持保持存活状态。如果是true，核心线程使用keepAliveTime等待工作并且会超时。

corePoolSize
核心线程数量是保持存活(不允许超时等)的工作线程的最小数量，如果allowCoreThreadTimeOut被设置为true，核心线程数为0.

maximumPoolSize
最大线程数量。实际的最大线程数量用CAPACITY在内部绑定。

defaultHandler
默认拒绝执行策略处理器。

shutdownPerm
shutdown和shutdownNow方法调用者的权限要求。

