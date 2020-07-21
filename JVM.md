1. JVM

   Java虚拟机是运行Java字节码的虚拟机。JVM针对不同系统的特定实现，目的是使用相同的字节码，执行结果相同

   1. .java源文件->编译器javac->.class字节码文件
   2. 字节码文件->JVM->机器码

2. 运行时数据区域

   1. 线程私有：

      线程私有数据区域生命周期与线程相同，依赖用户线程的启动/结束而创建/销毁

      1. 程序计数器：

         程序计数器是一块较小的内存空间，可以看做是当前线程所执行的字节码的行号指示器。字节码解释器通过改变程序计数器来依次读取记录，实现代码的流程控制。在多线程的情况下，程序计数器用于记录当前线程执行的位置，当线程被切换回来的时候能找到该线程上次运行到哪儿

         程序计数器是唯一一个不会出现OutOfMemoryError的内存区域

      2. 虚拟机栈：

         描述java方法执行的内存模型，每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈和出栈的过程

         若Java虚拟机的内存大小不允许动态扩容，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候，就会抛出StackOverflowError错误

         若Java虚拟机栈的内存大小允许动态扩容，且当线程请求栈时内存用完了，无法动态扩展时，会抛出OutOfMemoryError错误

      3. 本地方法栈：

         虚拟机栈为虚拟机执行Java方法服务，而本地方法栈则为虚拟机使用到的Native方法服务

   2. 线程共享

      1. 堆：

         创建的对象和数组都保存在Java堆内存中，也是垃圾收集器进行垃圾收集的最重要内存区域

      2. 方法区：

         用于存储被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码等数据，使用Java堆的永久代来实现方法区

         JDK1.8时，方法区被彻底移除了，取而代之是元空间，元空间使用的是直接内存

3. 堆内存常见分配策略

   1. 对象优先在eden区分配
   2. 大对象直接进入老年代
   3. 长期存活的对象将进入老年代
   4. 动态对象年龄判断

4. 判断对象是否已经死亡

   1. 引用计数法：给对象添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候引用计数器为0的对象就是不可能再被使用的。缺点：无法解决对象之间的相互循环引用的问题
   2. 可达性分析法：通过一系列的"GC roots"对象作为起点搜索。节点所走过的路径称为引用链，如果在"GC roots"和对象之间没有可达路径，则称该对象是不可达的

5. 引用：

   1. 强引用：如果一个对象具有强引用，垃圾收集器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会随意回收具有强引用的对象来解决内存不足问题
   2. 软引用：如果一个对象只具有软引用，如果内存空间足够，垃圾收集器就不会回收它，如果内存空间不足了，就会回收这些对象的内存
   3. 弱引用：如果一个对象只具有弱引用，在垃圾收集器线程扫描它所管辖的内存区域的过程中，一旦发现只具有弱引用的对象，不管当前内存空间是否足够，都会回收它的内存。不过由于垃圾收集器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象
   4. 虚引用：虚引用主要用来跟踪对象被垃圾回收的活动

6. 不可达的对象并非"非死不可"

   要真正宣告一个对象死亡，至少要经历两次标记过程；可达性分析法中不可达的对象被第一次标记并进行一次筛选，筛选的条件是此对象是否有必要执行finalize方法。当对象没有覆盖finalize方法，或finalize方法已经被虚拟机调用过时，虚拟机将两种情况视为没有必要执行

7. 如何判断一个常量是废弃常量

   如果没有如何对象引用该常量的话，说明常量是废弃常量

8. 如何判断一个类是无用的类

   1. 该类所有的实例都已经被回收，也就是Java堆中不会存在该类的所有实例
   2. 加载该类的ClassLoader已经被回收
   3. 该类对应java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

9. 垃圾收集算法

   1. 标记-清除算法：分为标记和清除阶段，首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象
   2. 复制算法：将内存分为大小相同的两块，每次使用中的一块。当这一块内存使用完后，就将还存活的对象复制到另一块，然后再把使用的空间一次清理掉
   3. 标记-整理算法：标记过程和标记-清除算法一样，标记后不是清理对象，而是将存活对象移向内存的一端，然后清除端边界外的对象
   4. 分代收集算法：根据对象存活周期的不同将内存分为几块，新生代使用复制算法，老年代使用标记-清除或标记-整理算法

10. 垃圾收集器

    1. 新生代收集器

       1. Serial收集器：使用复制算法，Serial是一个单线程收集器，只会使用一条垃圾收集线程去完成垃圾收集工作，并且在进行垃圾收集的同时，必须暂停其他所有的工作线程，直到垃圾收集结束
       2. ParNew收集器：使用复制算法，是Serial收集器的多线程版本，除了使用多线程进行垃圾收集之外，其余的行为和Serial收集完全一样，垃圾收集过程中同样也要暂停其他的工作线程
       3. Parallel Scavenge收集器：使用复制算法，也是多线程的垃圾收集器，它重点关注的是程序达到一个可控的吞吐量

    2. 老年代收集器

       1. Serial Old收集器：Serial收集器的老年代版本，使用标记-整理算法，是一个单线程收集器
       2. Parallel Old收集器：Paraller Scavenge收集器的老年代版本，使用多线程和标记-整理算法
       3. CMS收集器：主要目标是获取最短垃圾回收停顿时间，使用多线程和标记-清除算法
          1. 初始标记：暂停所有的工作线程，只标记GC roots直接关联的对象
          2. 并发标记：开启GC和用户线程，进行GC roots跟踪记录可达对象
          3. 重新标记：为修正在并发标记期间，因用户线程继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程
          4. 并发清除：清除GC roots不可达对象，和用户线程一起工作

    3. G1收集器：

       满足GC停顿时间要求的同时，还具备高吞吐量性能特征，采用标记-整理算法

       1. 初始标记、并发标记、最终标记、筛选回收。G1收集器在后台维护一个优
       2. 先列表，每次根据允许的收集时间，优先回收价值最大的Region

11. 类加载过程

    1. 加载：

       1. 通过全类名获取定义此类的二进制字节流
       2. 将字节流所有代表的静态存储结构转换为方法区的运行时数据结构
       3. 在内存中生成一个代表该类的Class对象，作为方法区这些数据的访问入库

    2. 连接：

       1. 验证：确保Class文件的字节流中包含的信息是否符合当前虚拟机的要求
       2. 准备：正式为类变量分配内存并设置类变量的初始值阶段
       3. 解析：虚拟机将常量池中的符号引用替换为直接引用的过程

    3. 初始化：

       初始化阶段是执行类构造器<client>方法的过程，<client>方法是由编译器自动收集类中的类变量的赋值操作和静态语句块中的语句合并而成的

12. 对象的加载

    1. 类加载检查：虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析、初始化过，如果没有，那必须先执行相应的类加载过程

    2. 分配内存：在类加载检查通过后，接下来虚拟机将未新生对象分配内存。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划出出来

    3. 初始化零值：内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值，这一步操作保证了对象的实例字段在Java代码中可以不赋初始值就可以直接使用

    4. 设置对象投：虚拟机要对对象进行必要的设置，例如这个对象是那个类的实例，如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息。这些信息存放在对象头中。另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁

    5. 执行init方法

       执行<init>方法，把对象按照程序员的意愿进行初始化

13. 类加载器

    1. 启动类加载器 Bootstrap ClassLoader：负责加载JAVA_HOME\lib目录中的类
    2. 拓展类加载器 Extension ClassLoader：负责加载JAVA_HOME\lib\ext目录中的类
    3. 应用程序类加载器 Application ClassLoader：负责加载用户路径classpath上的类库

14. 双亲委派

    当一个类收到类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父类去完成，每一层次类加载器都是如何，因此所有的加载请求都应该传送到启动类加载器，只有当父类加载器反馈自己无法完成这个请求的时候，子类加载器才会尝试自己去加载