# JAVA的并发

## 线程

1. 线程的实现
   1. 将执行的代码放在一个类的run方法中，这个类要实现Runable接口。
   1. 将继承这个类的实体代码放入Thread中，然后调用Thread.start方法
2. 常用代码

~~~java
java.lang.Thread
Thread(Runable target) //构造一个新线程，调用指定目标的run()方法
void start() //启动这个线程，从而调用run()方法。
static void sleep(long millis) //休眠指定的毫秒数

java.lang.Runable
void run() //执行的方法
~~~

3. 线程状态（6种）

   1. New（新建）

      1. 当用new操作符创建一个新线程时，如new Thread(r),这个线程还没有开始运行。这时的状态时新建(new)

   2. Runnable（可运行）

      1. 1. 一旦调用start方法，线程就处于可运行（runnable）状态。一个可运行的线程可能正在运行也可能没有运行。

         ~~~java
         java.lang.Thread
         static void yield() //使当前正在执行的线程向另一个线程交出运行权。
         ~~~

   3. Blocked（阻塞）

   4. Waiting（等待）

      1. 当线程处于阻塞或等待状态时，暂时是不活动的。要由线程调度器重新激活这个线程。具体细节取决于它是怎样到达非活动状态的
         1. 当一个线程试图获取一个内部的对象锁，而这个锁目前被其他线程占用，该线程就会被阻塞。当所有其他线程都释放了这个锁，并且线程调度器允许该线程持有这个锁时，它将变成非组赛状态。
         2. 当线程等待另一个线程通知调度器出现一个条件时，这个线程会进入等待状态
         3. 有几个方法由超时参数，调用这些方法会让线程进入计时等待状态。

   5. Timed waiting（计时等待）

   6. Terminated（终止）

      1. 由两个原因会导致进程终止
         1. run方法正常退出，线程自然终止
         2. 因为一个没有捕获的异常终止了run方法，使线程意外终止

   7. 要确定一个线程的当前状态，只需要调用getState()方法

      ~~~java
      java.lang.Thread
      Thread.State getState() //得到这个线程的状态：取值为NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING或TERMINATED
      ~~~

4. 线程属性

   1. 中断线程：

      1. 通过interrupt方法可以用来请求终止一个线程
      2. Thead.currentThread().isInterrupted()，判断当前进程是否被中断。Thead.currentThread()方法获取当前进程。线程处于阻塞状态，会抛出异常（InterruptedException）
      3. 常用代码

      ~~~java
      java.lang.Thread
      void.interrupt() //向线程发送中断请求。线程的中断状态将被设置为true
      static boolean interrupted() //测试当前线程是否被中断
      boolean isInterrupted() //测试线程是否被中断
      static Thread currentThread（） //返回表示当前正在执行的线程的Thread对象
      ~~~

   2. 守护线程

      1. 守护线程的唯一用于是为其他线程提供服务

      2. ~~~java
         java.lang.Thread
         void setDaemon(boolean isDaemon) //标识该线程为守护线程或用户线程。这一方法必须在线程启动之前调用
         ~~~

   3. 线程名

      1. setName方法为线程设置任何名字

   4. 线程优先级

      1. 有10个优先级，从1到10。用setPriority方法提高或降低任何一个线程的优先级。

## 同步

1. 概念：在多线程应用中，两个或两个以上的线程需要共享对同一数据的存取。这会破坏该对象，这种现象称为竞态条件(race condition)。为了防止这种线程，需要同步数据存取。

2. 有两种机制可防止并发访问代码块。

   1. synchronized关键字和ReentrantLock类

   - ReentrantLock类

   1. ReentrantLock类产生的锁称为重入（reentrant）锁，因为线程可以反复获得已拥有的锁。锁有一个持有计数（hold count）来跟踪对lock方法的嵌套调用。线程每一次调用lock后都要调用unlock来释放锁。

   2. ~~~
      java.util.concurrent.locks.lock
      void lock() //获得这个锁；如果锁当前被另一个线程占用，则阻塞
      void unlock() //释放这个锁
      java.util.concurrent.locks.Reentrantlock
      ReentrantLock() //构造一个重入锁，可以用来保护临界区
      ~~~

   3. 条件对象

      1. 线程进入临界区后却发现只有满足了某个条件之后才能执行。可以使用条件对象来管理那些已经获得了一个锁却不能做有用工作的线程。

      2. ~~~java
         java.util.concurrent.locks.Lock
         Condition newCondition() //返回一个与这个锁相关联的条件对象
         
         java.util.concurrent.locks.COndition
         void await() //将该线程放在这个条件的等待集中
         void signalAll() //解除该条件等待集中所有线程的阻塞状态
         void signal() //从该条件的等待集中随机选择一个线程，解除其组赛状态
         ~~~

      3. 总结：

         1. 锁用来保护代码片段，一次只能有一个线程执行被保护的代码
         2. 锁可以管理试图进入被保护代码段的线程
         3. 一个锁可以有一个或多个相关联的条件对象
         4. 每个条件对象管理那些已经进去被保护代码段但还不能运行的线程。

   - synchronized关键字

     1. 对象内部锁：Java中的每个对象都有一个内部锁。如果一个方法声明时拥有synchronized关键字，那么对象的锁将保护整个方法。
     2. 内部对象锁只有一个关联条件。wait方法将一个线程增加到等待集中，notifyAll/notify方法可以解除等待现成的阻塞。

   - synchronized关键字和ReentrantLock类常用做法

     1. 在许多情况下，可以使用java.util.concurrent包中的某种机制，它会为你处理所有的锁定。
     2. 尽量使用synchronized关键字

   - 同步块

     1. ~~~java
        synchronized(obj)
        {
           ....
        }
        ~~~

     2.  使用一个对象的锁来实现额外的原子操作，这种做法称为客户端锁定（client-side locking）。

3. volatile字段

   1. volatile变量不能提供原子性。但是，如果只是为了读写一两个实例字段，或者换句话说，屏蔽多线程间的共享变量，就可以使用volatile字段。



## 线程集合的安全

1. java.util.concurrent提供了映射，有序集和队列的高效实现：ConcurrentHashMap，ConcurrentSkipListMap，ConcurrentSkipListSet和ConcurrentLinkedQueue。

2. 阻塞队列

   1. 大部分线程问题可以用一个或多个队列以优雅而安全的方式描述。生产者线程向队列插入元素，消费者线程则获取元素。
   2. 组赛队列方法

   | 方法    | 正常动作               | 特殊情况下的动作                             |
   | ------- | ---------------------- | -------------------------------------------- |
   | add     | 添加一个元素           | 如果队列满，则抛出illegalStateException异常  |
   | element | 返回队头元素           | 如果队列空，则抛出NoSuchElementException异常 |
   | offer   | 添加一个元素并返回true | 如果队列满，则返回false                      |
   | peek    | 返回队头元素           | 如果队列空，则返回null                       |
   | poll    | 移除并返回队头元素     | 如果队列空，则返回null                       |
   | put     | 添加一个元素           | 如果队列满，则阻塞                           |
   | remove  | 移除并返回队头元素     | 如果队列空，则抛出NoSuchElementException异常 |
   | take    | 移除并返回队头元素     | 如果队列空，则阻塞                           |

## 任务和线程池

 1. Callable接口与Runnable类似，但是有返回值。

    1. ~~~java
       java.util.concurrent.Callable<V>
       V call() //运行一个将产生结果的任务
       ~~~

 2. Future保存异步计算的结果。

    1. ~~~java
       java.util.concurrent.Future<V>
       V get()
       V get(long,TimeUnit unit) //获取结果，这个方法会阻塞，知道结果可用或者超过了指定的时间。如果不成功，第二个方法会抛出TimeoutException异常
       boolean cancel(boolean mayInterrupt) //尝试取消这个任务的运行。如果跟任务已经开始，并且mayInterrupt参数值为true，它就会被中断。如果成功执行了取消操作，则返回true
       boolean isCancelled() //如果任务在完成前被取消，则返回true
       boolean isDone() //如果任务结束，无论是正常完成，中途取消，还是发生异常，都返回true。
       ~~~

 3. 执行Callable的一种方法是使用FutureTask，它实现了Future和Runnable接口。

    1. ~~~java
       java.util.concurrent.FutureTask<V>
       FutureTask(Callable<V> task)
       FutureTask(Runable task,V result) //构造一个即是Future<V> 又是Runable的对象
       ~~~

 4. 执行器(Executors)类有许多静态工厂方法，用来构造线程池。

    1. 执行者工厂方法

       | 方法                             | 描述                                         |
       | -------------------------------- | -------------------------------------------- |
       | newCachedThreadPool              | 必要时创建新线程，空闲线程会保留60秒         |
       | newFiexdThreadPool               | 池中包含固定数目的线程；空闲线程会一直保留   |
       | newWorkStealingPool              | 一种适合"fork-join"任务的线程池              |
       | newSingleThreadExecutor          | 只有一个线程的"池"，会顺序地执行所提交的任务 |
       | newScheduledThreadPool           | 用于调度执行的固定线程池                     |
       | newSingleThreadScheduledExecutor | 用于调度执行的单线程"池"                     |

    2. 使用连接池时所做的工作：

       1. 调用Executors类的静态方法newCachedThreadPool 或 newFixedThreadPool
       2. 调用submi提交Runnable或Callable对象
       3. 保存好返回的Future对象，以便得到结果或者取消任务
       4. 当不想再提交任务任务时，调用shutdow

## 异步计算

1. 可完成Future

   1. 当有一个Future对象时，需要调用get来获得值，这个方法会阻塞，直到值可用。CompletableFuture类实现了Future接口，他提供了获得结果的另一种机制。

      1. ~~~
         CompletableFuture<String> f =...;
         f.thenAccept(s -> Process the result string s);
         ~~~
