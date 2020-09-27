# Redis单线程模型

**Redis 基于 Reactor 模式来设计开发了自己的一套高效的事件处理模型**，这套事件处理模型对应的是 Redis 中的文件事件处理器（file event handler）。由于文件事件处理器（file event handler）是单线程方式运行的，所以我们一般都说 Redis 是单线程模型。 

Redis 通过**IO 多路复用程序** 来监听来自客户端的大量连接（或者说是监听多个 socket），它会将感兴趣的事件及类型(读、写）注册到内核中并监听每个事件是否发生。 **I/O 多路复用技术的使用让 Redis 不需要额外创建多余的线程来监听客户端的大量连接，降低了资源的消耗**

文件事件处理器（file event handler）主要是包含 4 个部分：

- 多个 socket（客户端连接）
- IO 多路复用程序（支持多个客户端连接的关键）
- 文件事件分派器（将 socket 关联到相应的事件处理器）
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

 ![img](https://snailclimb.gitee.io/javaguide/docs/database/Redis/images/redis-all/redis%E4%BA%8B%E4%BB%B6%E5%A4%84%E7%90%86%E5%99%A8.png) 

###  文件事件处理器 

①redis是基于reactor模式开发了网络事件处理器，这个处理器叫做 文件事件处理器(file event Handler)。这个文件事件处理器是单线程的，所以redis才叫做单线程模式，采用IO多路复用机制去同时监听多个socket，根据socket上的事件来选择对应的事件处理器来处理这个事件。

②如果被监听的socket准备好执行accept、read、write、close等操作的时候，跟操作对应的文件事件就会产生，这个时候文件处理器就会调用之前关联好的的事件处理器来处理这个事件。

③文件事件处理器是单线程模式运行的，但是通过IO多路复用机制监听多个socket，可以实现高性能的网络通信模型，又可以跟内部其他单线程的模块进行对接，保证了redis内部的线程模型的简单性。

④文件事件处理器的结构包含四个部分：多个socket、IO多路复用程序、文件事件分派器、事件处理器(命令请求处理器、命令回复处理器、连接应答处理器，等等)。

⑤多个socket可能并发的产生不同的操作，每个操作对应不同的文件 事件，但是IO多路复用程序会监听多个socket，但是会将socket放到一个队列中去处理，每次从队列中取出一个socket给事件分派器，事件分派器把socket给对应的事件处理器。

⑥然后一个socket的事件处理完了之后，IO多路复用程序才会将队列中的下一个socket给事件分派器。事件分派器会根据每个socket当前产生的事件，来选择对应的事件处理器来处理。

（2）文件事件
①当socket变得可读时(比如客户端对redis执行write操作，或者close操作)，或者有新的可以应答的socket出现时(客户端redis执行connect操作)，socket就会产生一个AE_READABLE事件。

②当socket变得可写的时候(客户端对redis执行read操作)，socket就会产生一个AE_WRITABLE事件。

③IO多路复用程序可以同时监听AE_READABLE和AE_WRITABLE两种事件，要是一个socket同时差生了这两种事件，那么文件分配器优先处理AE_READABLE事件，然后才是AE_WRITABLE事件。

（3）文件事件处理器
 如果是客户端要连接redis，那么会为socket关联连接应答处理器。
 如果是客户端要写数据到redis，那么会为socket关联命令请求处理器。
 如果是客户端要从redis读数据，那么会为socket关联命令回复处理器。

![线程模型](https://user-gold-cdn.xitu.io/2019/5/23/16ae3498fcb59371?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 （4）客户端与redis通信的一次流程

①在redis启动初始化的时候，redis会将连接应答处理器跟AE_READABLE事件关联起来，接着如果一个客户端跟redis发起连接，此时redis会产生一个AE_READABLE事件，然后由连接应答处理器来处理跟客户端建立连接，创建客户端响应的socket，同时将这个socket的AE_READABLE事件跟命令请求处理器关联起来。

②当客户端向redis发起请求的时候(不管是读请求还是写请求，都一样)，首先就会在socket产生一个AE_READABLE事件，然后由对应的命令请求处理器来处理。这个命令请求处理器就会从socket中读取请求的相关数据，然后执行操作和处理。

③接着redis这边准备好了给客户端的响应数据之后，就会将socket的AE_WRITABLE事件跟命令回复处理器关联起来，当客户端这边准备好读取相应数据时，就会在socket上产生一个AE_WRITABLE事件，会由相应的命令回复处理器来处理，就是将准备好的响应数据写入socket，供客户端读取。

④命令回复处理器写完之后，就会删除这个socket的AE_WRITABLE事件和命令回复处理器的关联关系。

![一次通信过程](https://user-gold-cdn.xitu.io/2019/5/23/16ae3492f13c7f4c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

###  Redis6.0多线程的实现机制 


![img](https://mmbiz.qpic.cn/mmbiz_png/DkA0VYrOHvtWkvO02tIickx6ibOgCywvlibHt0WSa1QYguvtDPUmGffgRave8k5wKbLK359HaJltbObPPiblicUcy3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**流程简述如下：**

1. 主线程负责接收建立连接请求，获取 socket 放入全局等待读处理队列
2. 主线程处理完读事件之后，通过 RR(Round Robin) 将这些连接分配给这些 IO 线程
3. 主线程阻塞等待 IO 线程读取 socket 完毕
4. 主线程通过单线程的方式执行请求命令，请求数据读取并解析完成，但并不执行
5. 主线程阻塞等待 IO 线程将数据回写 socket 完毕
6. 解除绑定，清空等待队列

![img](https://mmbiz.qpic.cn/mmbiz_png/DkA0VYrOHvtWkvO02tIickx6ibOgCywvlibpbYER5698cvSsm69yWHQ7iaUwQ140o9PbN5R9kHtlIzGiaoBN2LbwmyA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**该设计有如下特点：**

1. IO 线程要么同时在读 socket，要么同时在写，不会同时读或写
2. IO 线程只负责读写 socket 解析命令，不负责命令处理

总结 Redis 选择使用单线程模型处理客户端的请求主要还是因为 CPU 不是 Redis 服务器的瓶颈，所以使用多线程模型带来的性能提升并不能抵消它带来的开发成本和维护成本，系统的性能瓶颈也主要在网络 I/O 操作上；而 Redis 引入多线程操作也是出于性能上的考虑，对于一些大键值对的删除操作，通过多线程非阻塞地释放内存空间也能减少对 Redis 主线程阻塞的时间，提高执行的效率。

# Redis数据结构

### string

1. **介绍** ：string 数据结构是简单的 key-value 类型。虽然 Redis 是用 C 语言写的，但是 Redis 并没有使用 C 的字符串表示，而是自己构建了一种 **简单动态字符串**（simple dynamic string，**SDS**）。相比于 C 的原生字符串，Redis 的 SDS 不光可以保存文本数据还可以保存二进制数据，并且获取字符串长度复杂度为 O(1)（C 字符串为 O(N)）,除此之外,Redis 的 SDS API 是安全的，不会造成缓冲区溢出。
2. **常用命令:** `set,get,strlen,exists,dect,incr,setex` 等等。
3. **应用场景** ：一般常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。

### list

1. **介绍** ：**list** 即是 **链表**。链表是一种非常常见的数据结构，特点是易于数据元素的插入和删除并且且可以灵活调整链表长度，但是链表的随机访问困难。许多高级编程语言都内置了链表的实现比如 Java 中的 **LinkedList**，但是 C 语言并没有实现链表，所以 Redis 实现了自己的链表数据结构。Redis 的 list 的实现为一个 **双向链表**，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。
2. **常用命令:** `rpush,lpop,lpush,rpop,lrange、llen` 等。
3. **应用场景:** 发布与订阅或者说消息队列、慢查询。

### hash

1. **介绍** ：hash 类似于 JDK1.8 前的 HashMap，内部实现也差不多(数组 + 链表)。不过，Redis 的 hash 做了更多优化。另外，hash 是一个 string 类型的 field 和 value 的映射表，**特别适合用于存储对象**，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。 比如我们可以 hash 数据结构来存储用户信息，商品信息等等。
2. **常用命令：** `hset,hmset,hexists,hget,hgetall,hkeys,hvals` 等。
3. **应用场景:** 系统中对象数据的存储。

#### set

1. **介绍 ：** set 类似于 Java 中的 `HashSet` 。Redis 中的 set 类型是一种无序集合，集合中的元素没有先后顺序。当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。可以基于 set 轻易实现交集、并集、差集的操作。比如：你可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程。
2. **常用命令：** `sadd,spop,smembers,sismember,scard,sinterstore,sunion` 等。
3. **应用场景:** 需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景

### sorted set

1. **介绍：** 和 set 相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，还可以通过 score 的范围来获取元素的列表。有点像是 Java 中 HashMap 和 TreeSet 的结合体。
2. **常用命令：** `zadd,zcard,zscore,zrange,zrevrange,zrem` 等。
3. **应用场景：** 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

# Redis过期数据删除策略

1. **惰性删除** ：只会在取出key的时候才对数据进行过期检查。这样对CPU最友好，但是可能会造成太多过期 key 没有被删除。
2. **定期删除** ： 每隔一段时间抽取一批 key 执行删除过期key操作。并且，Redis 底层会并通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响。

# Redis内存淘汰机制

Redis 提供 6 种数据淘汰策略：

1. **volatile-lru（least recently used）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）
5. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！

4.0 版本后增加以下两种：

1. **volatile-lfu（least frequently used）**：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰
2. **allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key

# 缓存穿透

缓存穿透是指用户查询数据，在数据库没有，自然在缓存中也不会有。这样就导致用户查询的时候，在缓存中找不到，每次都要去数据库再查询一遍，然后返回空（相当于进行了两次无用的查询）。这样请求就绕过缓存直接查数据库，这也是经常提的缓存命中率问题。 

解决方法：

1.  缓存无效 key 

   如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，长不超过五分钟。通过这个直接设置的默认值存放到缓存，这样第二次到缓冲中获取就有值了，而不会继续访问数据库。

2. 布隆过滤器

   将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力。

 ![image](https://snailclimb.gitee.io/javaguide/docs/database/Redis/images/redis-all/%E5%8A%A0%E5%85%A5%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8%E5%90%8E%E7%9A%84%E7%BC%93%E5%AD%98%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B.png) 

# 缓存雪崩

由于原有缓存失效，新缓存未到期间所有原本应该访问缓存的请求都 去查询数据库了，而对数据库 CPU 和内存造成巨大压力，严重的会造成数据库宕机。从而形成一系列 连锁反应，造成整个系统崩溃。

解决方法：

1. 一般并发量不是特别多的时候，使用多的解决方案是加锁排队。
2. 给每一个缓存数据增加相应的缓存标记，记录缓存的是否失效，如果缓存标记失效，则更新数据缓 存。 
3.  为key设置不同的缓存失效时间