 [TOC] 

## 语法
### 多个参数
`Object... args`这类使用与String.format，在通配符不知个数的情况下。

### [hashCode](https://www.ibm.com/developerworks/cn/java/j-5things3/index.html?ca=drs-)

1. 支持哈希码的键依赖于可变字段的内容，这样容易产生 bug，解决：永远不要将可变对象类型用作 HashMap 中的键。
2. hashmap的equal和hashcode为什么要同时重写:如果你重载了equals，比如说是基于对象的内容实现的，而保留hashCode的实现不变，那么很可能某两个对象明明是“相等”，而hashCode却不一样。

### 泛型

1. `List<?>`、`List`、`List<Object>`三者区别：

| 引用变量的类型 | 名称                             | 可以接受的类型                           | 能否添加元素           |
| -------------- | -------------------------------- | ---------------------------------------- | ---------------------- |
| `List`         | 原始类型                         | 任何对应List的参数化类型， 包括List      | 可以添加任意类型的元素 |
| `List<?>`      | 通配符类型                       | 以接受任何对应List的参数化类型，包括List | 不能添加任何元素       |
| `List<Object>` | 实际类型参数为Object的参数化类型 | 仅可以接受List和其本身类型               | 可以添加任意类型元素   |

> 泛型就是类型参数化, 好处是编译器会检查参数类型.多态就是多个类由继承(实现接口)得到的一致外观, 好处是简化代码。

2. 通配符和边界
   - `<? extends T>`,上界通配符。能放一切继承自T的类.频繁往外读取内容的,适合用<? extends T >。
   - `<? super T>`,下界通配符。能放T及T继承的类，但不能放T的派生类。经常往里插入的,适合用 <? super T> 。

### 正则

1. 正则表达式过滤出字母、数字和中文
   - 过滤出字母: `[^(A-Za-z)]`
   - 过滤出数字: `[^(0-9)]`
   - 过滤出中文: `[^(\\u4e00-\\u9fa5)]`,不过直接复制这个到idea会自动补/，因此记住这里只有2个/
   - 过滤字母数字：`s.replaceAll("[^A-Za-z0-9]", ""); `

## 集合框架

[继承关系](https://segmentfault.com/a/1190000014240704)

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/collect.jpeg" width=80% height=80% />

### List

1. 特点：有序，存储顺序与取出顺序一致。

#### ArrayList

1. 结构图7：

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/7ArrayList.png" width=80% height=80% />

2. 实现了List接口，可以List基本操作；实现了RandomAccess接口，作为一个标识，表示可以快速随机访问元素。

3. 底层数组，线程不安全。将数据存在Object[] elementData中。

4. `add(E e)`方法：

   - 检查一下数组的容量是否足够：size+1即当前增加元素所需要的容量minCapacity，如果是增加的第一个元素，设置minCapacity为默认10。
   - 所需容量minCapacity与当前容量elementData.length比较，如果数组长度不够，扩容1.5倍。如果还小，扩充为minCapacity。返回`Arrays.copyOf(elementData, newCapacity);`而Arrays.copyOf底层是System.arraycopy。

5. `remove(int index)`方法，跟`add(int index,E element)`一样，底层元素移动都是System.arraycopy实现。删除元素时不会减少容量，若减少容量则调用trimToSize()

#### 线程安全的ArrayList

1. Collections.synchronizedList(list)

   1. 在所有方法都加关键字，另外官方文档是建议我们在遍历的时候加锁处理的。

      List list = Collections.synchronizedList(new ArrayList());
         synchronized (list) {
            Iterator i = list.iterator(); // Must be in synchronized block
            while (i.hasNext())
      	  foo(i.next());
         }
         //因为该方法没加锁
         @Override
         public Iterator<T> iterator() {
             return backingList.iterator();
         }

2. CopyOnWriteArrayList

   1. add(E)时直接用ReentrantLock锁住代码块。拷贝一份新的，指向原数组，并且使用volatile修饰数组来保证修改后的可见性。

      private transient volatile Object[] array;

   2. add(int,E)同样用该锁，然后按index分2部分进行copy出一份新的数组进行相关的操作，在执行完修改操作后将原来集合指向新的集合来完成修改操作；

3. 特性：

   1. synchronizedList适合对数据要求较高的情况，但是因为读写全都加锁，所有效率较低。
   2. CopyOnWriteArrayList效率较高，适合读多写少的场景，因为在读的时候读的是旧集合，所以它的实时性不高。

#### LinkedList

1. 结构图8：

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/8LinkedList.png" width=80% height=80% />

2. 实现了List接口，可以List基本操作；实现了Deque接口，可以操作队列。没有实现RandomAccess接口，不可快速随机访问元素。

3. 底层双向链表，线程不安全。注意：双向链表不是循环链表，只能说节点有前指针与后指针。

4. `add(E e)`方法，往链表最后添加元素。

5. `get(int index)`方法：如果下标小于长度的一半就从头遍历，否则从尾遍历。set方法类似。

### Vector

1. 底层数组，线程安全。扩容是直接扩一倍。

### 总结

1. 查询多用ArrayList，增删多用LinkedList
2. 实现了RandomAccess接口的list，优先选择普通for循环
3. 未实现RandomAccess接口的ist， 优先选择iterator遍历
4. [ArrayList与LinkedList的对比](https://github.com/Snailclimb/JavaGuide/blob/master/%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87/%E7%BE%8E%E5%9B%A2-%E8%BF%9B%E9%98%B6%E7%AF%87.md#%E4%B8%89-%E8%81%8A%E8%81%8A-java-%E4%B8%AD%E7%9A%84%E9%9B%86%E5%90%88%E5%90%A7%EF%BC%81)

### Set

1. 特点：元素不可重复。

#### HashSet

1. 底层哈希表，即一个数组，其中元素为链表。

#### TreeSet

1. 底层红黑树。

#### LinkedHashSet

1. 底层哈希表与链表组成。

### Map

1. Collections.unmodifiableMap(cache);返回一个不可修改的视图
1. [红黑树](http://www.sohu.com/a/201923614_466939)特性：
   - 根节点总是黑色的；
   - 每个叶子节点都是红色
   - 如果节点是红色的，则它的父子节点必须是黑色；
   - [从根节点到叶节点或空子节点的每条路径，必须包含相同数目的黑色节点](https://www.cnblogs.com/CarpenterLee/p/5503882.html)

#### HashMap

1. 结构图

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/9Hashmap.png" width=80% height=80% />

2. 为何红黑树？我的理解：

   1. 不光是为了存储，更多的是为了建立索引树，便于搜索。
   2. 常见的二叉搜索树，一般容易极端情况退化成链表；而平衡树好，但是每次插入容易旋转啥的，很费时；现在红黑树调整可以分为两类：一类是颜色调整，即改变某个节点的颜色；另一类是结构调整，集改变检索树的结构关系,不完全L树的平衡条件的，即每个节点的左子树和右子树的高度最多差1的二叉查找树。红黑是用非严格的平衡来换取增删节点时候旋转次数的降低。O(㏒(n))

3. 底层数组+链表(拉链法)，即散列表

4. Hashmap的扩容需要满足两个条件：当前数据存储的数量（即size()）大小必须大于等于阈值；当前加入的数据是否发生了hash冲突

5. `put(K key,V value)`方法。

   - 以key计算哈希值,这里面的key.hashCode()是Object方法，就是任何对象都有一个哈希码hashCode。接着计算`hashCode^hashCode>>>16`，才是最终的哈希值。下面解释这样计算的原因。

         static final int hash(Object key) {
             int h;
             return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
         }

    - 接着，执行put操作。散列表容量n,而哈希值范围很广，为了将哈希值缩小范围正好对应在[0,n-1]上做数组下标，采用&运算，这样不管哈希值多大，高于n的二进制位全部为0，计算的结果就在[0,n-1]。现在解释`hashCode^hashCode>>>16`，如果很多哈希码高位差异大而低位相同，直接计算`(n - 1) & hash`的值相同的情况会增加，导致碰撞概率增大。而这样计算后，此时的低位实际上是高位与低位的结合，增加了随机性。

#### ConCurrentHashMap

1. 特性：

   - JDK1.8底层是链表+数组+红黑树
   - 支持高并发的访问和更新，线程安全
   - 检索操作不用加锁，get方法非阻塞
   - key和value都不允许为null

    >Hashtable是在每个方法上都加上了Synchronized完成同步，效率低下。1.8的ConcurrentHashMap通过在部分加锁和利用CAS算法来实现同步。1.7是采用分段锁。

2. [在原先HashMap的基础上采取的方案](https://www.jianshu.com/p/c0642afe03e0)。

## NIO流

1. Non-blocking，与IO的区别：
   - IO是面向流的，NIO是面向缓冲区的。
   - IO流是阻塞的，NIO流是不阻塞的。NIO中，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情,真正的IO操作是内核线程。而IO则是：当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了
   - [NIO的核心是selector中的事件通知](http://www.fanyeong.com/2016/09/19/%E5%AF%B9nio%E7%9A%84%E4%B8%80%E7%82%B9%E7%90%86%E8%A7%A3/).
     - 在BIO中,每来一个连接请求，服务器都将分配一个线程来处理这个连接，这个线程专门用来处理读写等事件，直到连接关闭。阻塞并不是IO读写，而是在IO等待。
     - 而NIO中每次来连接都丢给专门处理连接的线程来将请求注册到Selectors，即一个线程用于接受请求；一个线程池用于（或者自己新建一个线程）处理请求——完成读写数据的操作
   - Selectors（选择器）。选择器用于使用单个线程处理多个通道。线程之间的切换对于操作系统来说是昂贵的。 选择器是一个可以监视多个通道的对象，使用Selector的话，我们必须把Channel注册到Selector上，然后就可以调用Selector的select()方法。这个方法会进入阻塞，直到有一个channel的状态符合条件。当方法返回后，线程可以处理这些事件

### 原理

1. [来源1](https://tech.meituan.com/2016/11/04/nio.html)

2. [来源2](https://segmentfault.com/a/1190000003063859) 

3. 文件描述符fd

   1. 文件描述符在形式上是一个非负整数,它是一个索引值，指向内核为每一个进程所维护的**该进程打开文件的记录表**。
   2. 当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。

4. 首先缓存IO的过程：

   1. 等待数据准备。数据会先被拷贝到操作系统内核的缓冲区中
   2. 将数据从内核拷贝到进程中 ，即从操作系统内核的缓冲区拷贝到应用程序的地址空间。

5. 而NIO的工作原理是：当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。因此，用户进程需要不断的主动询问kernel数据好了没有。

6. 而在kernel内核内部，用单个process处理所有IO请求，即不断的轮询所负责的所有socket，当某个socket有数据准备好了，就通知用户进程：

   2. 当用户进程调用了selector.select，没有事件到来，那么整个进程会被block。而同时，kernel会“监视”所有selector负责的socket。
   3. 如果有事件到来，即任何一个socket中的数据准备好了，将执行系统调用（Linux 2.6之前是select、poll，2.6之后是epoll）。新事件到来的时候，会在selector上注册标记位，标示可读、可写或者有连接到来。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程

    >select是阻塞的，无论是通过操作系统的通知（epoll）还是不停的轮询(select，poll)，这个函数是阻塞的。所以你可以放心大胆地在一个while(true)里面调用这个函数而不用担心CPU空转。

7. select，poll，epoll都是**IO多路复用**的机制，但本质上都是同步I/O，而异步I/O则无需自己负责进行读写.

   1. `int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);`底层数组
      1. select监视writefds、readfds、和exceptfds。
      2. 调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。
      3. 缺点在于单个进程能够监视的文件描述符的数量存在最大限制,Linux上一般为1024
   2. poll,和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。select和poll都需要在返回后，通过遍历文件描述符来获取已经就绪的socket。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。底层链表。
   3. epoll。底层红黑树
      1. `int epoll_create(int size);`,创建一个epoll的句柄，参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议
      2. `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；`对指定描述符fd执行op操作，将描述符和感兴趣的事件注册到epoll实例，即红黑树进行增删改
      3. `int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);`阻塞等待IO事件,返回需要处理的事件数目，并会将就绪的描述符存储到events参数中。如返回0表示已超时。
   4. epoll对上面2个的缺点处理：
      1. 监视的描述符数量不受限制，就是树的大小。
      1. 遍历：epoll事先通过epoll_ctl()来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait() 时便得到通知。此处去掉了遍历文件描述符，而是通过监听回调的的机制。

### Channel

[来源](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483966&idx=1&sn=d5cf18c69f5f9ec2aff149270422731f&chksm=fd98545fcaefdd49296e2c78000ce5da277435b90ba3c03b92b7cf54c6ccc71d61d13efbce63#rd)

1. FileChannel： 用于文件的数据读写。使用：
   - 开启FileChannel。无法直接打开抽象类FileChannel，需要通过 InputStream，OutputStream或RandomAccessFile获取FileChannel。
   - 从FileChannel读取数据read()/写入数据write()
   - 关闭FileChannel
2. DatagramChannel： 用于UDP的数据读写
   - 获取DataGramChannel
   - 接收/发送消息
3. SocketChannel： 用于TCP的数据读写，一般是客户端实现

  - 通过SocketChannel连接到远程服务器
  - 创建读数据/写数据缓冲区对象来读取服务端数据或向服务端发送数据
  - 关闭SocketChannel

4. ServerSocketChannel: 允许我们监听TCP链接请求，每个请求会创建会一个SocketChannel，一般是服务器实现

   - 通过ServerSocketChannel 绑定ip地址和端口号

   - 通过ServerSocketChannelImpl的accept()方法创建一个SocketChannel对象用户从客户端读/写数据

   - 创建读数据/写数据缓冲区对象来读取客户端数据或向客户端发送数据

   - 关闭SocketChannel和ServerSocketChannel

     >非阻塞模式.在使用传统的ServerSocket和Socket的时候,很多时候程序是会阻塞的,比如 serversocket.accept()的时候都会阻塞 accept()方法除非等到客户端socket的连接或者被异常中断,否则会一直等待下去;在ServerSocket与Socket的方式中 服务器端往往要为每一个客户端(socket)分配一个线程,而每一个线程都有可能处于长时间的阻塞状态中.而过多的线程也会影响服务器的性能;在JDK1.4引入了非阻塞的通信方式,这样使得服务器端只需要一个线程就能处理所有客户端socket的请求

5. Scatter / Gather

   - Scatter: 从一个Channel读取的信息分散到N个缓冲区中(Buufer)

         ByteBuffer header = ByteBuffer.allocate(128);
          ByteBuffer body = ByteBuffer.allocate(128);
          ByteBuffer [] array = {header,body}
          channel.read(array);
          //read()方法内部会负责把数据按顺序写进传入的buffer数组内。一个buffer写满后，接着写到下一个buffer中。
          //举个例子，假如通道中有200个字节数据，那么header会被写入128个字节数据，body会被写入72个字节数据；

   - Gather: 将N个Buffer里面内容按照顺序发送到一个Channel.

     >无论是scatter还是gather操作，都是按照buffer在数组中的顺序来依次读取或写入的

6. 通道之间的数据传输。以上都是通道与缓冲的传输，通道之间可以传输。在Java NIO中如果一个channel是FileChannel类型的，那么他可以直接把数据传输到另一个channel。

### Selector（选择器）

[来源](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483970&idx=1&sn=d5e2b133313b1d0f32872d54fbdf0aa7&chksm=fd985423caefdd354b587e57ce6cf5f5a7bec48b9ab7554f39a8d13af47660cae793956e0f46#rd)

1. 用于检查一个或多个NIO Channel（通道）的状态是否处于可读、可写。使用Selector的好处在于： 使用更少的线程来就可以来处理通道了， 相比使用多个线程，避免了线程上下文切换带来的开销。

2. SelectionKey表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系.
   - `OP_CONNECT`,连接就绪
   - `OP_ACCEPT`,接收就绪
   - `OP_READ`,读就绪
   - `OP_WRITE`, 写就绪
3. select(),返回的int值表示有多少通道已经就绪
   - int select()：阻塞到至少有一个通道在你注册的事件上就绪了。
   - int select(long timeout)：和select()一样，但最长阻塞时间为timeout毫秒。
   - int selectNow()：非阻塞，只要有通道就绪就立刻返回。
4. 一旦调用select()方法，并且返回值不为0时，则 可以通过调用Selector的selectedKeys()方法来访问已选择键集合 。`Set selectedKeys=selector.selectedKeys(); `,进而可以放到和某SelectionKey关联的Selector和Channel

### 问题：

- System.out.println()是什么？println是PrintStream的一个方法。out是一个静态PrintStream类型的成员变量，System是一个java.lang包中的类，用于和底层的操作系统进行交互。
- File类，它主要用于知道一个文件的属性，读写权限，大小等信息
- RandomAccessFile，它在java.io包中是一个特殊的类，既不是输入流也不是输出流，它两者都可以做到。


### AIO

1. 特点：
   - 读完后通知
   - 不会加快IO，只是读完后进行通知
   - 使用回调函数，进行业务处理。能够胜任那些重量级，读写过程长的任务

## Java8

### Lambda

#### 应用

1. 前提：

   1. 方法的参数或者局部变量类型必须为接口。抽象类都不行
   2. 接口中有且仅有一个抽象方法。（equal方法因为object已实现不算，default修饰的方法也不算）

2. lambda例子

       //最初。匿名的内部类不能访问外部的索引值。如果只是一个值，没有修改过，那是可以访问的，但是如果修改过就不能使用，即需要final，即使不声明，也不能修改。
       ExecutorService executorService = Executors.newFixedThreadPool(10);
       for(int i = 0; i < 5; i++) {
         int temp = i;
         executorService.submit(new Runnable() {
           public void run() {
             //If uncommented the next line will result in an error
             //System.out.println("Running task " + i); 
             //local variables referenced from an inner class must be final or effectively final
       
             System.out.println("Running task " + temp); 
           }
         });
       }
       executorService.shutdown();
       
       //一次演变
       ExecutorService executorService = Executors.newFixedThreadPool(10)；            
       IntStream.range(0, 5)
        .forEach(i -> 
          executorService.submit(new Runnable() {
            public void run() {
              System.out.println("Running task " + i); 
            }
          }));
       executorService.shutdown();
       
       //二次演变
       IntStream.range(0, 5)
                .forEach(i -> executorService.submit(() -> System.out.println("Running task " + i)));

3. 将一个list中所有元素合在一起：不用reduce

       names.stream()
            .collect(Collectors.joining(", "))

4. 尽管lambda 表达式没有任何错误，但它的语法对于当前这个任务而言过于复杂。为了理解 (parameters) -> body 的用途，我们需要进入 body（在 -> 的右侧）来查看该形参发生了什么。如果该lambda表达式没有对该形参执行任何实际操作，则付出的努力就白费了。在此情况下，将传递 lambda 表达式替换为方法引用会比较有益。

       numbers.stream()
              .forEach(e -> System.out.println(e));
       //方法引用  
       numbers.stream()
              .forEach(System.out::println);

   如果 lambda 表达式的目的仅是将一个形参传递给实例方法，那么可以将它替换为实例上的方法引用

         e -> this.increment(e)
         this::increment

   如果传递表达式要传递给静态方法，可以将它替换为类上的方法引用

         e -> Integer.valueOf(e)
         Integer::valueOf

   如果形参是方法调用的目标,跟静态类似

         .map(e -> e.doubleValue())
         className::doubleValue

    一个构造函数调用

         .collect(toCollection(() -> new LinkedList<Double>()));
         .collect(toCollection(LinkedList::new));

    reduce传递2个参数的调用

         .reduce(0, (total, e) -> Integer.sum(total, e)));
         .reduce(0, Integer::sum));

5. lambda中闭包携带状态的方式是：闭包保留着 状态value 的一个副本

#### 原理

1. 实例：

       //目的接口，
       interface Swim{
           void fun();
       }
       public class Lambda {
           public static void main(String[] args) {
               fun(() -> System.out.println("6"));
           }
           /**匿名内部类方式，
           fun(new Swim() {
             @Override
             public void fun() {
                 System.out.println("6");
             }
           });**/
           
           public static void fun(Swim swim){
               swim.fun();
           }
       }

2. lambda会在该类Lambda中生成一个私有静态方法，比如mian方法中第一次使用lambda：`lambda$main$0`，方法中的内容为lambda的表达式内容

       //javap -c -p C:\Users\wd\IdeaProjects\untitled\out\production\untitled\Lambda.class获取如下信息
       public class Lambda {
          public Lambda();
            Code:
               0: aload_0
               1: invokespecial #1                  // Method java/lang/Object."<init>":()V
               4: return
       
          public static void main(java.lang.String[]);
            Code:
               0: invokedynamic #2,  0              // InvokeDynamic #0:fun:()LSwim;
               5: invokestatic  #3                  // Method fun:(LSwim;)V
               8: return
       
          public static void fun(Swim);
            Code:
               0: aload_0
               1: invokeinterface #4,  1            // InterfaceMethod Swim.fun:()V
               6: return
       
          private static void lambda$main$0();
            Code:
               0: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
               3: ldc           #6                  // String 6
               5: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
               8: return
        }

3. 然后在运行中动态生成一个内部类，实现目的接口Swim，并重写抽象方法，重写内容就是调用原先类新增的静态私有方法`Lambda.lambda$main$0();`。具体类生成方式：在`C:\Users\wd\IdeaProjects\untitled\out\production\untitled>`目录下执行命令：`java -Djdk.internal.lambda.dumpProxyClasses com.Lambda`,在正常的Java执行命令下加上这个参数，即可保存lambda动态生成的匿名内部类class数据到一个文件中。

       package com;
       import java.lang.invoke.LambdaForm.Hidden;
       
       // $FF: synthetic class
       final class Lambda$$Lambda$1 implements Swim {
           private Lambda$$Lambda$1() {
           }
       
           @Hidden
           public void fun() {
               Lambda.lambda$main$0();
           }
       }

### Stream

1. [流stream的原理](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/)。这篇文章学习流必须看。

   1. 操作包括：

     - Intermediate：map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered。Intermediate 操作永远是惰性化的。也就是说没有Terminal操作，Intermediate操作并不会执行，比如在Intermediate中打印数据则不会打印出来。
       - arrayList.stream().map(e -> e * 3 + 2).mapToInt(Integer::intValue).sum();
     - Terminal：forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator
     - Short-circuiting：anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit

   2. 一次性使用。terminal操作之后无法再次进行terminal操作。
   3. 可以并行计算parallelStream。缺点是结果打乱之前顺序,以及线程安全问题。
   4. 注意：reduce(),如果没有设置初始种子，返回的是 Optional，也就是可能没有值进行reduce操作。而如果指定了初始值，就返回具体的对象 

2. 流转数组

       Integer[] integers = Stream.of(1, 2, 3, 4, 5).toArray(Integer[]::new);

3. 转Map等

       employees.stream().collect(Collectors.toMap( e -> e.getEmpId(),  e -> e));

4. IntStream数值流，[为了减少Integer与int计算时的拆装箱问题而引入](https://www.jianshu.com/p/e429c517e9cb)。

   1. iterate ： 依次对每个新生成的值应用函数

          //生成流，首元素为 0，之后依次加 2
          Stream.iterate(0, n -> n + 2)

   2. 注意流的使用顺序

          //输出0.1后继续运行
          IntStream.iterate(0, i -> (i + 1) % 2).distinct().limit(6).forEach(System.out::println);
          //输出0.1后停止
          IntStream.iterate(0, i -> (i + 1) % 2).limit(6).distinct().forEach(System.out::println); 

### 函数式编程思想

1. Function接口，一般将函数作为参数进行引用传递，需要关注的是Function本身是一种行为。

   > `::`，[双冒号](https://www.cnblogs.com/tietazhan/p/7486937.html)。`Function<String, String> f = String::toUpperCase`,这里通过这种方式调用成员方法，将其赋值给Function，则默认Function的第一个参数为该类的实例对象。如果toUpperCase有参数，则不能使用该方式调用，除非重载的方法没有参数。

2. [Function<T, R>，T—函数的输入类型，R-函数的输出类型。](https://www.orchome.com/935)lambda的写法就是Function内部写一个方法T为输入，使用的时候调用apply或者compose等就是相当于重写了该方法，获取返回。因为apply方法是必须实现的，而compose、andThen是default关键字修饰。

```
// 我的理解是，compose与andThen的顺序正好相反。即两个function执行顺序相反，同时将前者的输出作为后一个function的输入
Function<Integer, Integer> function1 = (value) -> value * value;
Function<Integer, Integer> function2 = (value) -> value * 2;
function1.compose(function2).apply(10); // 400
function1.andThen(function2).apply(10); // 200
```

3. 当然，对于需要重复使用的Function，可以事前定义好，像写函数一样，在具体使用的时候直接赋值。然后调用时直接用apply方法传参数就好了。

4. 将Function作为函数参数的意义：提前将行为抽象出来，在使用的时候定义具体的行为方式，而不是一个一个直接定义函数。解耦。比如：求年龄>20,求年龄<20岁等过滤时，不用定义函数，用Function实现过滤输出。

```
public class function {
    public static void main(String[] args) {
        // function传递行为
        compute(1, value -> value + 10);
        compute(1, value -> value * 2);
        // 非function
        compute1(1);
        compute2(1);
    }

    public static int compute(int a, Function<Integer, Integer> function){
        return  function.apply(a);
    }

    public static int compute1(int a){
        return a + 10;
    }

    public static int compute2(int a){
        return a * 2;
    }
}
```

5. BiFunction，如果想利用Function接口实现两个输入参数一个输出，做不到，因此有了BiFunction。不过BiFunction只有andThen方法，因为BiFunction只能返回一个输出，compose中将Function作为输入的话，下一个执行的行为是BiFunction，需要2个输入，不可。

6. Predicate接口，针对一个参数的，用于判断lambda的方法体中表达式真假。跟Function接口一样，将判断作为一种行为抽象出来，作为参数传递。

7. Supplier接口，不需要参数，返回一个结果。一般用于工厂方法。

8. BinaryOperator接口，`interface BinaryOperator<T> extends BiFunction<T,T,T>`,用法我觉得跟C++的operator类似。定义两个对象之间的计算得到另一个对象。当然远不止这样的功能。比如在该接口中只有2个静态方法：
   - minBy(Comparator<T> c),返回两个对象间较小的一个。
   - maxBy()

### ForkJoin

1. parallelStream底层使用该框架，将一个大任务拆分为很多小任务来异步执行。包含三个模块：

   1. 线程池ForkJoinPool,继承自AbstractExecutorService,

      <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/Thread/pool.png" width=100% height=100% />

   2. 任务对象ForkJoinTask,实现Future接口。

   3. 执行任务的线程ForkJoinWorkerThread，继承自Thread类。

2. 原理包括分治法以及工作窃取  

3. 案例：

       package com;
       
       import java.util.concurrent.ForkJoinPool;
       import java.util.concurrent.RecursiveTask;
       
       public class ForkJoin {
           public static void main(String[] args) {
               ForkJoinPool forkJoinPool = new ForkJoinPool(); //创建一个线程池
               SumTask sumTask = new SumTask(1, 100000000);  // 创建一个线程池
               forkJoinPool.invoke(sumTask);  //提交任务
           }
       }
       
       // 创建一个求和的任务,该任务需继承RecursiveTask
       class SumTask extends RecursiveTask<Long> {
           //任务量拆分的临界值
           private long THRESHOLD = 3000L;
           //任务计算中的起始值跟结束值，比如计算3000到6000的值
           private long start = 0L;
           private long end = 0L;
       
           public SumTask(long start, long end) {
               this.start = start;
               this.end = end;
           }
       
           @Override
           protected Long compute() {
               long len = end - start;
       
               if (len > THRESHOLD){
                   //如果需要拆分,按2分法的方式进行拆分
                   long middle = (start + end) / 2;
                   SumTask left = new SumTask(start, middle);
                   left.fork();
                   SumTask right = new SumTask(middle + 1, end);
                   right.fork();
       
                   return left.join() + right.join();
               }else{
                   //如果需要计算
                   return (end + start) * len / 2;
               }
           }
       }

### Optional

1. 常见用法

       Optional<String> stringOptional = Optional.of("");
       stringOptional = Optional.empty();
       stringOptional.orElse(""); //如果stringOptional有值，就用自己的值，否则用orElse的值。泛型
       stringOptional.ifPresent(s -> System.out.println(s)); //与lambda的结合，如果有值就调用
       
       //在JDK1.9后改进了 Optional 类增加了 ifPresentOrElse 方法
       Optional.ofNullable(user).ifPresentOrElse(u -> {
          user.getName();
          user.getAge();
       }, () -> {
          System.err.println("user 对象为null");
       });
       
       //
       public String getUpperUserName2(Optional<User>op){
          String upperName=op. map(u->u.getUserName())
                            . map(s->s.toUpperCase())
                            . orE1se("null"); 
          return upperName;
        }

### 日期

1. 原先Date存在线程不安全等问题。现在使用LocalTime时分秒、LocalDate年月日、LocalDateTime具体时间

2. 相比之前，多了一些方法：

       LocalDate now = LocalDate.now();
       LocalDate localDate = LocalDate.of(2010, 1 ,1);
       now.isAfter(localDate); //时间的比较挺常用的
       now.isBefore(localDate);
       // 时间解析格式，有了自带的格式。但是LocalDate没有，需要自己解析
       LocalDateTime now = LocalDateTime.now();
       DateTimeFormatter dtf = DateTimeFormatter.ISO_DATE_TIME;
       String date = now.format(dtf); // 2019-12-09T18:48:13.14
       DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd"); // 或者自定义格式，注意时间方式MM月yyyy年dd天，都是有具体规范。线程安全

3. Duration计算时间的距离

       LocalDateTime now = LocalDateTime.now();
       LocalDateTime end = LocalDateTime.now();
       Duration duration = Duration.between(now, end);
       duration.toDays(); // 相差的天数，这里LocalTime也可以
       duration.toHours();

4. Period计算日期的距离

       LocalDate now = LocalDate.now();
       LocalDate end = LocalDate.now();
       Period period = Period.between(now, end); // 后面时间-前面时间
       period.getMonths();

### Instant

1. 时间戳，用于处理1970.1.1 00:00:00以来的秒与纳秒



## Effictive Java

### 静态工厂方法

1. 适用于基于接口的框架。其中，接口为静态工厂方法提供恰当的返回类型。
2. 例如：OPCEServiceFactory产生实例，而OPCEServiceInf为接口。所有实例都实现了OPCEServiceInf接口。得益于**静态工厂方法返回对象的类可以根据输入参数的不同而不同**。使用这种静态工厂方法需要客户端通过接口而不是实现类来引用返回的对象，这通常是良好的实践。

3. 例如：服务提供者框架JDBC。
   1. 服务接口，它表示实现。即Connection
   2. 提供者注册 API，提供者用来注册实现。即DriverManager.registerDriver
   3. 服务访问 API，客户端使用该 API 获取服务的实例。DriverManager.getConnection
   4. 服务提供者接口，它描述了一个生成服务接口实例的工厂对象。在没有服务提供者接口的情况下，必须对实现进行反射实例化。即Driver接口。即`Class.forName("com.mysql.jdbc.Driver")`
4. 缺点：很难找到对应的实例。需要通过编码规范命名：
   - from —— 类型转换方法，它接受单个参数并返回此类型的相应实例，例如：`Date d = Date.from(instant);`。
   - create 或 newInstance —— 该方法保证每次调用返回一个新的实例，例如：`Object newArray = Array.newInstance(classObject, arrayLen)`;
   - 除非有令人信服的理由，否则不要提供独立于构造方法或静态工厂的公共初始化方法。

### builder模式

1. 优势：builder 的参数可以在构建方法的调用之间进行调整，以改变创建的对象。 builder 可以在创建对象时自动填充一些属性，例如每次创建对象时增加的序列号。
2. 劣势：builder 模式比伸缩构造方法模式更冗长，因此只有在有足够的参数时才值得使用它，比如四个或更多。

### 私有构造方法

1. 工具类（utility classes）不是设计用来被实例化的。因此提供一个私有构造方法。

### 依赖注入优于硬连接资源（hardwiring resources）

1. 许多类依赖于一个或多个底层资源。例如，拼写检查器依赖于字典。不要用单例和静态工具类来实现依赖一个或多个底层资源的类，且该资源的行为会影响到该类的行为；也不要直接用这个类来创建这些资源。而应该将这些资源或者工厂传给构造器（或者静态工厂，或者构建器），通过它们来创建类。这个实践就被称作依赖注人，它极大地提升了类的灵活性、可重用性和可测试性。

### 避免创建不必要的对象

1. 工厂方法 `Boolean.valueOf(String)` 比构造方法 `Boolean(String)` 更可取，后者在 Java 9 中被弃用。构造方法每次调用时都必须创建一个新对象，而工厂方法永远不需要这样做，在实践中也不需要。

### 关闭资源使用try

1. try-with-resources。要使用这个构造，资源必须实现 `AutoCloseable` 接口。

2. 例如：

   ```java
   // try-with-resources with a catch clause
   static String firstLineOfFile(String path, String defaultVal) {
       try (BufferedReader br = new BufferedReader(
              new FileReader(path))) {
           return br.readLine();
       } catch (IOException e) {
           return defaultVal;
       }
   }
   ```

3. 由于底层物理设备发生故障，对 `readLine` 方法的调用可能会引发异常，并且由于相同的原因，调用 `close` 方法可能会失败。 在这种情况下，第二个异常完全冲掉了第一个异常。 在异常堆栈跟踪中没有第一个异常的记录，这可能使实际系统中的调试非常复杂——通常这是你想要诊断问题的第一个异常。

### 组合优于继承

1. 父类的实现可能会从发布版本不断变化，如果是这样，子类可能会被破坏，即使它的代码没有任何改变。 因此，一个子类必须与其超类一起更新而变化.
2. 存在继承的情况下，方法向超类方向集中，数据向子类方向集中

### 接口优于抽象类

1. 你可以通过提供一个抽象的骨架实现类（abstract skeletal implementation class）来与接口一起使用，将接口和抽象类的优点结合起来。 接口定义了类型，可能提供了一些默认的方法，而骨架实现类在原始接口方法的顶层实现了剩余的非原始接口方法。 继承骨架实现需要大部分的工作来实现一个接口。 这就是模板方法设计模式。

### 接口仅用来定义类型

1. 当类实现接口时，该接口作为一种类型（type），可以用来引用类的实例。因此，一个类实现了一个接口，因此表明客户端可以如何处理类的实例。**为其他目的定义接口是不合适的**。

### 重载

1. **重载（overloaded）方法之间的选择是静态的，而重写（overridden）方法之间的选择是动态的**。 

## 异常

1. IllegalArgumentException。检查参数有效性。一般应该在执行计算之前显式检查方法的参数。
2. IllegalStateException。非法状态异常。例如，如果在某个对象被正确地初始化之前，调用者就企图使用这个对象，就会抛出这个异常。如果我们在一个方法中需要抛出IllegalStateException，那么说明这个方法具有**implicit dependencies**
3. **不要直接重用 Exception、RuntimeException、Throwable 或者 Error。** 对待这些类要像对待抽象类一样。
4. 抛出与抽象对应的异常。
   1. 如果方法抛出的异常与它所执行的任务没有明显的联系，这种情形将会使人不知所措。
   2. 为了避免这个问题， **更高层的实现应该捕获低层的异常，同时抛出可以按照高层抽象进行解释的异常。**这种做法称为异常转译。

### 异常种类

1. [算术异常类](https://www.cnblogs.com/cvst/p/5822373.html)：ArithmeticExecption。除以0等。
2. 空指针异常类：NullPointerException
3. 类型强制转换异常：ClassCastException
4. 数组下标越界异常：ArrayIndexOutOfBoundsException
5. 文件未找到异常：FileNotFoundException
6. 字符串转换为数字异常：NumberFormatException
7. java.lang.OutOfMemoryError：内存不足错误。
8. java.lang.StackOverflowError：堆栈溢出错误。当一个应用递归调用的层次太深而导致堆栈溢出时抛出该错误。
9. java.lang.UnsupportedClassVersionError。JDK版本不同异常。换一个打包版本就行
10. [异常捕获后何时抛出？何时自己处理？](https://juejin.im/post/5ae66791f265da0b92655c5d)
    1. 尽量将异常统一抛给上层调用者，由上层调用者统一之时如何进行处理。如果在每个出现异常的地方都直接进行处理，会导致程序异常处理流程混乱，不利于后期维护和异常错误排查。由上层统一进行处理会使得整个程序的流程清晰易懂。 


## 小工具

1. 界面输入密码繁琐。然后将其粘贴到书签栏，形成一个书签。

   ```javascript
   javascript:$('#name').val('admin');
   $('#pwd').val('123456');
   $('#submitDataverify').click();
   ```


## 参考

1. [Effictive Java](https://www.bookstack.cn/read/effective-java-3rd-chinese/docs-README.md)
2. [郑教练博客](http://arganzheng.life/)
