## 1.进程与线程

**进程**是程序的一次执行过程，是系统运行程序的基本单位 

**线程**是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。 

## 2.线程的生命周期

 Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态 

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

- 当程序使用new关键字创建了一个线程之后，该线程就处于新建状态，此时仅由JVM为其分配内存，并初始化其成员变量的值 

- 当线程对象调用了start()方法之后，该线程处于就绪状态。Java 虚拟机会为其创建方法调用栈和程序计数器，等待调度运行。 

- 如果处于就绪状态的线程获得了CPU，开始执行run()方法的线程执行体，则该线程处于运行状态。 

- 阻塞状态是指线程因为某种原因放弃了cpu 使用权，也即让出了cpu timeslice，暂时停止运行。 直到线程进入可运行(runnable)状态，才有机会再次获得cpu timeslice 转到运行(running)状态。阻塞的情况分三种

  - 等待阻塞（o.wait -> 等待队列）

    运行(running)的线程执行o.wait()方法，JVM会把该线程放入等待队列(waitting queue) 中。 

  - 同步阻塞（lock -> 锁池）

    运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。 

  - 其他阻塞（sleep/join）

    运行(running)的线程执行Thread.sleep(long ms)或t.join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入可运行(runnable)状态。 

- 线程会以下面三种方式结束，结束后就是死亡状态。 

  - 正常结束

    run()或call()方法执行完成，线程正常结束

  - 异常结束

    线程抛出一个未捕获的Exception或Error。

  - 调用stop

    直接调用该线程的stop()方法来结束该线程—该方法通常容易导致死锁，不推荐使用。 

 线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。 

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java+%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E8%BF%81.png)

## 3.线程死锁

多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。 

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-4/2019-4%E6%AD%BB%E9%94%811.png)

产生死锁必须具备以下四个条件：

1. 互斥条件：该资源任意一个时刻只由一个线程占用。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件:线程已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

## 4.Java线程实现

1. 继承Thread类创建线程类

   - 创建Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体
   - 创建Thread子类的实例，即创建了线程对象
   - 调用线程对象的start()方法来启动线程

2. 通过Runnable接口创建线程类

   - 定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。
   - 创建Runnable实现类的实例，并依次实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象
   - 调用线程对象的start()方法来启动线程

3. 通过Callable和Future创建线程

   - 创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值
   - 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值
   - 使用FutureTask对象作为Thread对象的target创建并启动线程
   - 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值

## 5.Java锁

1. 乐观锁

   乐观锁是一种乐观思想，即认为读多写少，遇到并发写的可能性低，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作（比较跟上一次的版本号，如果一样则更新），如果失败则要重复读-比较-写的操作。 
   java 中的乐观锁基本都是通过 CAS 操作实现的，CAS 是一种更新的原子操作，比较当前值跟传入值是否一样，一样则更新，否则失败。

2. 悲观锁

   悲观锁是就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会block直到拿到锁。java中的悲观锁就是Synchronized,AQS框架下的锁则是先尝试cas乐观锁去获取锁，获取不到，才会转换为悲观锁，如RetreenLock。

3. 自旋锁

   自旋锁原理非常简单，如果持有锁的线程能在很短时间内释放锁资源，那么那些等待竞争锁的线程就不需要做内核态和用户态之间的切换进入阻塞挂起状态，它们只需要等一等（自旋），等持有锁的线程释放锁后即可立即获取锁，这样就避免用户线程和内核的切换的消耗。 

4. 公平锁

   公平锁指的是锁的分配机制是公平的，通常先对锁提出获取请求的线程会先被分配到锁，ReentrantLock在构造函数中提供了是否公平锁的初始化方式来定义公平锁。 

5. 非公平锁

   JVM 按随机、就近原则分配锁的机制则称为不公平锁，ReentrantLock 在构造函数中提供了是否公平锁的初始化方式，默认为非公平锁。非公平锁实际执行的效率要远远超出公平锁，除非程序有特殊需要，否则常用非公平锁的分配机制。 

6. 可重入锁

   可重入锁，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。在JAVA环境下 ReentrantLock 和synchronized 都是可重入锁。 

7. 独占锁

   独占锁模式下，每次只能有一个线程能持有锁，ReentrantLock 就是以独占方式实现的互斥锁。独占锁是一种悲观保守的加锁策略，它避免了读/读冲突，如果某个只读线程获取锁，则其他读线程都只能等待，这种情况下就限制了不必要的并发性，因为读操作并不会影响数据的一致性。 

8. 共享锁

   共享锁则允许多个线程同时获取锁，并发访问共享资源，如：ReadWriteLock。共享锁则是一种乐观锁，它放宽了加锁策略，允许多个执行读操作的线程同时访问共享资源。 

## 6.Synchronized

synchronized 关键字解决的是**多个线程之间访问资源的同步性**，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。 

 **synchronized 关键字最主要的三种使用方式：** 

1.  **修饰实例方法:** 作用于当前对象实例加锁，进入同步代码前要获得 **当前对象实例的锁** 
2.  **修饰静态方法:** 也就是给当前类加锁，会作用于类的所有对象实例 ，进入同步代码前要获得 **当前 class 的锁**。 
3.  **修饰代码块** ：指定加锁对象，对给定对象/类加锁。`synchronized(this|object)` 表示进入同步代码库前要获得**给定对象的锁**。 

**Synchronized核心组件：**

1. Wait Set：哪些调用wait方法被阻塞的线程被放置在这里；
2. Contention List：竞争队列，所有请求锁的线程首先被放在这个竞争队列中；
3. Entry List：Contention List中那些有资格成为候选资源的线程被移动到Entry List中；
4.  OnDeck：任意时刻，多只有一个线程正在竞争锁资源，该线程被成为OnDeck；
5. Owner：当前已经获取到所资源的线程被称为Owner；
6.  !Owner：当前释放锁的线程。 

**Synchronized实现：**

![](https://img-service.csdnimg.cn/img_convert/32b2bbe073f6b94d4dd660cf5a47ac1b.png#pic_center)

1. JVM 每次从队列的尾部取出一个数据用于锁竞争候选者（OnDeck），但是并发情况下，ContentionList会被大量的并发线程进行CAS访问，为了降低对尾部元素的竞争，JVM会将一部分线程移动到EntryList中作为候选竞争线程。
2.  Owner 线程会在 unlock 时，将 ContentionList 中的部分线程迁移到 EntryList 中，并指定EntryList中的某个线程为OnDeck线程（一般是先进去的那个线程）。
3.  Owner 线程并不直接把锁传递给 OnDeck 线程，而是把锁竞争的权利交给 OnDeck，OnDeck需要重新竞争锁。这样虽然牺牲了一些公平性，但是能极大的提升系统的吞吐量，在JVM中，也把这种选择行为称之为“竞争切换”。
4. OnDeck线程获取到锁资源后会变为Owner线程，而没有得到锁资源的仍然停留在EntryList中。如果Owner线程被wait方法阻塞，则转移到WaitSet队列中，直到某个时刻通过notify或者notifyAll唤醒，会重新进去EntryList中。
5.  处于 ContentionList、EntryList、WaitSet 中的线程都处于阻塞状态，该阻塞是由操作系统来完成的（Linux内核下采用pthread_mutex_lock内核函数实现的）。
6. Synchronized是非公平锁。 Synchronized在线程进入ContentionList时，等待的线程会先尝试自旋获取锁，如果获取不到就进入 ContentionList，这明显对于已经进入队列的线程是不公平的，还有一个不公平的事情就是自旋获取锁的线程还可能直接抢占 OnDeck 线程的锁资源。
7. 每个对象都有个 monitor 对象，加锁就是在竞争 monitor 对象，代码块加锁是在前后分别加上monitorenter和monitorexit指令来实现的，方法加锁是通过一个标记位来判断的
8. synchronized 是一个重量级操作，需要调用操作系统相关接口，性能是低效的，有可能给线程加锁消耗的时间比有用操作消耗的时间更多。
9. Java1.6，synchronized进行了很多的优化，有适应自旋、锁消除、锁粗化、轻量级锁及偏向锁等，效率有了本质上的提高。在之后推出的 Java1.7 与 1.8 中，均对该关键字的实现机理做了优化。引入了偏向锁和轻量级锁。都是在对象头中有标记位，不需要经过操作系统加锁。
10. 锁可以从偏向锁升级到轻量级锁，再升级到重量级锁。这种升级过程叫做锁膨胀；

### 偏向锁

1. 偏向锁主要用来优化**同一线程多次申请同一个锁**的竞争，在某些情况下，大部分时间都是同一个线程竞争锁资源
2. 偏向锁的作用
   1. 当一个线程再次访问同一个同步代码时，该线程只需对该对象头的**Mark Word**中去判断是否有偏向锁指向它
   2. **无需再进入Monitor去竞争对象**（避免用户态和内核态的**切换**）
3. 当对象被当做同步锁，并有一个线程抢到锁时
   1. 锁标志位还是**01**，是否偏向锁标志位设置为**1**，并且记录抢到锁的**线程ID**，进入_**偏向锁状态**_
4. 偏向锁**不会主动释放锁**
   1. 当线程1再次获取锁时，会比较**当前线程的ID**与**锁对象头部的线程ID**是否一致，如果一致，无需CAS来抢占锁
   2. 如果不一致，需要查看锁对象头部记录的线程是否存活
      1. 如果**没有存活**，那么锁对象被重置为**无锁**状态（也是一种撤销），然后重新偏向线程2
      2. 如果存活，查找线程1的栈帧信息
         1. 如果线程1还是需要继续持有该锁对象，那么暂停线程1（**STW**），**撤销偏向锁**，**升级为轻量级锁**
         2. 如果线程1不再使用该锁对象，那么将该锁对象设为**无锁**状态（也是一种撤销），然后重新偏向线程2
5. 一旦出现其他线程竞争锁资源时，偏向锁就会被撤销
   1. 偏向锁的撤销**可能需要**等待**全局安全点**，暂停持有该锁的线程，同时检查该线程**是否还在执行该方法**
   2. 如果还没有执行完，说明此刻有**多个线程**竞争，升级为**轻量级锁**；如果已经执行完毕，唤醒其他线程继续**CAS**抢占

**红线流程部分：偏向锁的获取和撤销**

![在这里插入图片描述](https://img-service.csdnimg.cn/img_convert/a38af0f5585877d59c2bdecf47a8511f.png#pic_center)

### 轻量级锁

1. 当有另外一个线程竞争锁时，由于该锁处于**偏向锁**状态
2. 发现对象头Mark Word中的线程ID不是自己的线程ID，该线程就会执行CAS操作获取锁
   1. 如果获取**成功**，直接替换Mark Word中的线程ID为自己的线程ID，该锁会_**保持偏向锁状态**_
   2. 如果获取**失败**，说明当前锁有一定的竞争，将偏向锁**升级**为轻量级锁
3. 线程获取轻量级锁时会有两步
   1. 先把**锁对象的Mark Word**复制一份到线程的**栈帧**中（**DisplacedMarkWord**），主要为了**保留现场**!!
   2. 然后使用**CAS**，把对象头中的内容替换为**线程栈帧中DisplacedMarkWord的地址**
4. 场景
   1. 在线程1复制对象头Mark Word的同时（CAS之前），线程2也准备获取锁，也复制了对象头Mark Word
   2. 在线程2进行CAS时，发现线程1已经把对象头换了，线程2的CAS失败，线程2会尝试使用**自旋锁**来等待线程1释放锁
5. 轻量级锁的适用场景：线程**交替执行**同步块，***绝大部分的锁在整个同步周期内都不存在长时间的竞争***

**红线流程部分：升级轻量级锁**

![在这里插入图片描述](https://img-service.csdnimg.cn/img_convert/9eb895b60fd9adf8975093503bcbf1e4.png#pic_center)

### 自旋锁 / 重量级锁

1. 轻量级锁CAS抢占失败，线程将会被挂起进入阻塞状态
   1. 如果正在持有锁的线程在**很短的时间**内释放锁资源，那么进入**阻塞**状态的线程被**唤醒**后又要**重新抢占**锁资源
2. JVM提供了**自旋锁**，可以通过**自旋**的方式**不断尝试获取锁**，从而_**避免线程被挂起阻塞**_
3. 从JDK 1.7开始，自旋锁默认启用，自旋次数不建议设置过大（意味着长时间占用CPU）
   1. `-XX:+UseSpinning -XX:PreBlockSpin=10`
4. 自旋锁重试之后如果依然抢锁失败，同步锁会升级至重量级锁，锁标志位为10
   1. 在这个状态下，未抢到锁的线程都会**进入Monitor**，之后会被阻塞在**WaitSet**中
5. 在锁竞争不激烈且锁占用时间非常短的场景下，自旋锁可以提高系统性能
   1. 一旦锁竞争激烈或者锁占用的时间过长，自旋锁将会导致大量的线程一直处于**CAS重试状态**，**占用CPU资源**

![在这里插入图片描述](https://img-service.csdnimg.cn/img_convert/f8af04b89755cc28818a1a6a2595ff6f.png#pic_center)

## 7.volatile 关键字

**JMM（Java内存模型）：**

在 JDK1.2 之前，Java 的内存模型实现总是从**主存**（即共享内存）读取变量，是不需要进行特别的注意的。而在当前的 Java 内存模型下，线程可以把变量保存**本地内存**（比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成**数据的不一致**。 

![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-8/0ac7e663-7db8-4b95-8d8e-7d2b179f67e8.png)

要解决这个问题，就需要把变量声明为**`volatile`**，这就指示 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。所以，**volatile关键字除了防止 JVM 的指令重排 ，还有一个重要的作用就是保证变量的可见性。**

![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-8/d49c5557-140b-4abf-adad-8aac3c9036cf.png)

## 四种线程池

线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了大数量超出数量的线程排队等候，等其它线程执行完毕，再从队列中取出任务来执行。他的主要特点为：线程复用；控制大并发数；管理线程。 

- newCachedThreadPool:

  创建一个可根据需要创建新线程的线程池。调用execute将重用以前构造的线程（如果线程可用），如果现有线程没有可用的，则会创建一个新线程并添加到池中。终止并从缓存中移除那些已有60秒中未被使用的线程

- newFixedThreadPool

  创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程

- newScheduledThreadPool

  创建一个固定长度的线程池，而且以延迟或定时的方式来执行任务

- newSingleThreadExecutor

  创建一个单线程的线程池，这个线程池可以在线程死后（或发生异常时）重新启动一个线程来替代原来的线程继续执行下去

线程池的处理过程：

![](http://image.bubuko.com/info/202001/20200111164446068534.png)

提交任务。判断核心线程池是否已满，如果未满，则创建线程，如果已满判断等待队列是否已满，如果未满，则加入队列，如果已满，判断线程池是否已满，如果未满，则创建线程，如果已满，按照拒绝策略进行处理

1. 线程池的拒绝策略

   - AbortPolicy：抛出RejectedExecutionException来拒绝新任务的吃力
   - CallerRunsPolicy：该策略直接在调用者的线程中，运行当前被抛弃的任务
   - DiscardOldestPolicy：丢弃最老的一个请求，并尝试再次提交任务
   - DiscardPolicy：默默低丢弃无法处理的请求

2. 线程池的状态

   线程池有5种状态：running，shutdown，stop，tidying，terminated

   running->调用shutdown方法->shutdown->任务队列为空，并且线程池中执行的任务为空->tidying

   running->调用shuudownNow方法->stop->线程池中执行的任务为空->tidying

   tidying->terminated方法执行完毕->terminated

3. ThreadPoolExecutor 3 个最重要的参数：

   - corePoolSize：核心线程数线程数定义了最小可以同时运行的线程数量。
   - maximumPoolSize：当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
   - workQueue：当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

   ThreadPoolExecutor其他常见参数:

   1. keepAliveTim：当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
   2. unit： keepAliveTime参数的时间单位。
   3. threadFactory：executor 创建新线程的时候会用到。
   4. handler：饱和策略。关于饱和策略下面单独介绍一下。

## 原子类

```java
    // setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
```

AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。

**CAS** 的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址，返回值是 valueOffset。另外 value 是一个 volatile 变量，在内存中可见，因此 JVM 可以保证任何时刻任何线程总能拿到该变量的最新值。

## 双重校验锁实现对象单例（线程安全）

```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

`uniqueInstance` 采用 `volatile` 关键字修饰也是很有必要的， `uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：

1. 为 `uniqueInstance` 分配内存空间
2. 初始化 `uniqueInstance`
3. 将 `uniqueInstance` 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getUniqueInstance`() 后发现 `uniqueInstance` 不为空，因此返回 `uniqueInstance`，但此时 `uniqueInstance` 还未被初始化。

使用 `volatile` 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

## AQS

AbstractQueuedSynchronizer类如其名，抽象的队列式的同步器，AQS定义了一套多线程访问 共享资源的同步器框架，许多同步类实现都依赖于它，如常用的 ReentrantLock/Semaphore/CountDownLatch。 

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/AQS%E5%8E%9F%E7%90%86%E5%9B%BE.png)

它维护了一个 volatile int state（代表共享资源）和一个 FIFO 线程等待队列（多线程争用资源被 阻塞时会进入此队列）。这里 volatile 是核心关键词，具体 volatile 的语义，在此不述。state 的 访问方式有三种: 

1. getState() 
2. setState() 
3. compareAndSetState() 

**AQS 定义两种资源共享方式**

- Exclusive

  （独占）：只有一个线程能执行，如 ReentrantLock。又可分为公平锁和非公平锁：

  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

- **Share**（共享）：多个线程可同时执行，如 Semaphore/CountDownLatch。Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。

## CAS

CAS（Compare And Swap/Set）比较并交换，CAS 算法的过程是这样：它包含 3 个参数 CAS(V,E,N)。V 表示要更新的变量(内存值)，E 表示预期值(旧的)，N 表示新值。当且仅当 V 值等于 E 值时，才会将 V 的值设为 N，如果 V 值和 E 值不同，则说明已经有其他线程做了更新，则当 前线程什么都不做。后，CAS返回当前V的真实值。 CAS 操作是抱着乐观的态度进行的(乐观锁)，它总是认为自己可以成功完成操作。当多个线程同时 使用 CAS 操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。失败的线程不会被挂 起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。基于这样的原理， CAS操作即使没有锁，也可以发现其他线程对当前线程的干扰，并进行恰当的处理。 
