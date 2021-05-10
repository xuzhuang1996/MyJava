## 内存区域

1. [运行时数据区](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483910&idx=1&sn=246f39051a85fc312577499691fba89f&chksm=fd985467caefdd71f9a7c275952be34484b14f9e092723c19bd4ef557c324169ed084f868bdb#rd)如下图所示：

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/1jvm.jpg" width=80% height=80% />

2. [HotSpot虚拟机加载对象](https://blog.csdn.net/yangshangwei/article/details/81252722)：

  - 虚拟机遇到一条new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

  - 在类加载检查通过后，接下来虚拟机将为新生对象分配内存。根据堆中内存是否绝对规整，选择空闲列表或者指针碰撞的方式来分配内存。而Java堆是否规整是由所采用的垃圾收集器是否带有压缩整理功能决定。

     	>Java堆内存绝对规整，指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”（ Bump the Pointer）。例子：使用的垃圾收集器是Serial、ParNew。
          	>
          	>Java堆内存不规整，已使用的内存和空闲的内存相互交错，虚拟机就必须维护一个列表，记录哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为“空闲列表”（Free List）。例子：使用的垃圾收集器是CMS这类基于快速标记。

3. 对象的内存布局:
   - 对象头
     - 第一部分存储对象自身的运行时数据：如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID 、偏向时间戳等
     - 另一部分是类型指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
   - 实例数据
   - 对齐填充，不是必须的，只起到地址对齐的作用

4. 对象的访问定位，即如何使用对象，通过栈上的reference数据来操作堆上的具体对象。由于reference类型在Java虚拟机规范中只规定了一个指向对象引用。而没有规定这个引用应该通过何种方式去定位、访问堆中的对象的具体位置，它取决于Java虚拟机实现。
   -  使用句柄：在Java堆中划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，句柄中包含对象实例数据与类型各自具体地址信息，好处是当reference指向别的对象时，只需改变句柄中的实例数据指针。
      <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/2jvm句柄.png" width=72% height=72% />
   -  直接指针访问：HotSpot的reference中存储的直接就是对象地址。好处是速度更快，节省一次指针定位时间开销
      <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/3jvm直接指针.png" width=72% height=72% />


## 引用类型

1. [强引用](https://www.cnblogs.com/yw-ah/p/5830458.html)：垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。下面代码中，只有当obj这个引用被释放之后，对象才会被释放掉

   Object obj = new Object();

2. 软引用（SoftReference）：如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。sf是对obj的一个软引用，通过sf.get()方法可以取到这个对象。

   ```java
   	Object obj = new Object();
      	SoftReference<Object> sf = new SoftReference<Object>(obj);
      	obj = null;
      	sf.get();//有时候会返回null
   ```

   >可用于实现内存敏感的高速缓存，在内存足够的情况下直接通过软引用取值，无需从繁忙的真实来源查询数据，提升速度；当内存不足时，自动删除这部分缓存数据，从真正的来源查询这些数据。

3. 弱引用（WeakReference）:在垃圾回收器线程扫描它 所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存.

   ```java
   	Object obj = new Object();
   	WeakReference<Object> wf = new WeakReference<Object>(obj);
   	obj = null;
   	wf.get();//有时候会返回null
   	wf.isEnQueued();//返回是否被标记为即将回收的垃圾
   ```

   >弱引用可用于解决内存泄漏的问题。另外，弱引用主要用于监控对象是否已经被垃圾回收器标记为即将回收的垃圾，可以通过弱引用的isEnQueued方法返回对象是否被垃圾回收器标记。

4. 虚引用（PhantomReference）：垃圾回收时回收，无法通过引用取到对象值。虚引用是每次垃圾回收的时候都会被回收，通过虚引用的get方法永远获取到的数据为null，因此也被成为幽灵引用。

   ```java
   	Object obj = new Object();
   	PhantomReference<Object> pf = new PhantomReference<Object>(obj);
   	obj=null;
   	pf.get();//永远返回null
   	pf.isEnQueued();//返回是否从内存中已经删除
   ```

   >虚引用主要用于检测对象是否已经从内存中删除。

5. 注意：在程序设计中一般很少使用弱引用与虚引用，使用软引用的情况较多，这是因为软引用可以加速JVM对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生

## 内存回收

1. 如何判断对象是否死亡

   - 引用计数法，给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使用的。缺点：很难解决对象之间相互循环引用的问题。

   - 可达性分析算法，通过一系列称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径称为“引用链”，当一个对象到GC Roots没有任何的引用链相连时(从GC Roots到这个对象不可达)时，证明此对象不可用。对于上面的两个对象，虽然彼此引用，但是GC Roots不可达，也会回收。具体GC Roots有：
   
     - 虚拟机栈中引用的对象，程序中new一个对象，堆上将开辟一块空间，同时将空间的地址作为引用保存到虚拟机栈中，如果对象生命周期结束了，那么引用就会从虚拟机栈中出栈，因此如果在虚拟机栈中有引用，就说明这个对象还是有用的
     - 方法区中静态属性引用的对象
     - 方法区中常量引用的对象
     - 本地方法栈中(Native方法)引用的对象

### 垃圾收集算法

1. 垃圾回收动作何时执行：

   1. 次要GC，Minor GC,从Young空间（由Eden和Survivor空间组成）收集垃圾,当新生代无法为新生对象分配内存空间的时候，会触发Minor GC,很多对象的生命周期很短,所以选用复制算法，只需要少量的复制成本就可以完成回收
   2. Major GC,老年代的垃圾回收（又称Major GC）,该区域中对象存活率高,通常使用“标记-清理”或“标记-整理”算法
   3. 当永久代满时也会引发Full GC，会导致Class、Method元信息的卸载。

2. 触发OutOfMemoryException的条件

   1. JVM98%的时间都花费在内存回收
   2. 每次回收的内存小于2%

3. 标记-清除算法Mark-Sweep。如下图。缺点是效率低以及标记清除后会产生大量的不连续内存碎片。
   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/4jvm垃圾回收标记清除.png" width=50% height=50% />

4. 复制算法：将内存划分为大小相等的两块，每次只使用其中的一块。当这块内存用完了，就将还存活的对象复制到另一块内存上，然后把已使用过的内存空间一次清理掉。不会产生内存碎片，缺点是内存缩小一半。目前流行。
   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/5jvm复制算法.png" width=50% height=50% />

5. 标记-整理算法Mark-Compact：让所有存活对象都向一端移动，然后直接清理掉端边界以外的所有内存
   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/6jvm标记整理.png" width=50% height=50% />

6. 分代收集算法Generational Collection：根据对象的存活周期的不同将内存划分为几块，一般就分为新生代和老年代，根据各个年代的特点采用不同的收集算法。新生代（少量存活，容易产生碎片）用复制算法，老年代（对象存活率高）“标记-清理”算法。

   >HotSpot还多一个持久代：存放静态文件，例如Java类和方法，持久代对GC没有显著的影响。

### 垃圾收集器

[来源](http://www.cnblogs.com/ityouknow/p/5614961.html)

1. 串行收集器Serial，只使用一个线程去回收，即新生代、老年代均串行回收，工作的时候必须暂停其他所有的工作线程。如下图：

   - 参数控制：-XX:+UseSerialGC

     <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/7jvm串行.png" width=50% height=50% />

2. ParNew收集器，新生代并行，老年代串行。

   - 参数控制：-XX:+UseParNewGC  ParNew收集器

   - 参数控制：-XX:ParallelGCThreads 限制线程数量

     <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/8jvmParNew.png" width=50% height=50% />

3. Parallel Scavenge新生代收集器，使用复制算法、并行。

   - 参数控制：-XX:+UseParallelGC  使用Parallel收集器+老年代串行

   - 老年版Parallel Old 收集器:-XX:+UseParallelOldGC 使用Parallel收集器+ 老年代并行

     >Parallel Scavenge收集器关注点是吞吐量（高效率的利用CPU）。CMS等垃圾收集器的关注点更多的是用户线程的停顿时间（提高用户体验）。所谓吞吐量就是CPU中用于运行用户代码的时间与CPU总消耗时间的比值

4. CMS收集器Concurrent Mark Sweep，获取最短回收停顿时间为目标的收集器。运行过程如下图：

   - 初始标记，暂停所有的其他线程，并记录下直接与GC Roots相连的对象，速度很快

   - 并发标记，通过GC Roots Tracing判断对象是否仍在使用中，也就是可达性分析。此时用户线程可能会不断的更新引用域，无法保证实时性。耗时

   - 重新标记，暂停所有的其他线程，为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录。

   - 并发清除，开启用户线程，同时GC线程开始对为标记的区域做清扫。耗时

     <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/9jvmCMS.png" width=60% height=60% />

    缺点是回收算法“标记-清除”产生大量空间碎片、并发阶段会降低吞吐量。另外参数控制：

   - -XX:+UseConcMarkSweepGC，使用CMS收集器
   - -XX:+UseCMSCompactAtFullCollection， Full GC后，进行一次碎片整理；整理过程是独占的，会引起停顿时间变长
   - -XX:+CMSFullGCsBeforeCompaction  设置进行几次Full GC后，进行一次碎片整理
   - -XX:ParallelCMSThreads  设定CMS的线程数量

以上都是针对下图，而G1不同。

<img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/10jvm分区.png" width=70% height=70% />

5. G1收集器

   1. 特点：

      - 空间整合，G1收集器采用标记整理算法，不会产生内存空间碎片。

      - 可预测停顿，G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为N毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。

      - 使用G1收集器时，Java堆的内存布局与其他收集器有很大差别.其中E、S属于年轻代，O与H属于老年代。H巨型对象，当分配的对象大于等于Region大小的一半的时候就会被认为是巨型对象。H对象默认分配在老年代，可以防止GC的时候大对象的内存拷贝。

        <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/G1.png" width=70% height=70% />


   2. 提供了3种模式回收垃圾
      - young GC，当所有E区被耗尽无法申请内存时，就会触发一次young gc。E区的对象会移动到S区，当S区空间不够的时候，E区的对象会直接晋升到O区。
        - -XX:MaxGCPauseMillis	设置G1收集过程目标时间，默认值200ms
        - -XX:G1NewSizePercent	新生代最小值，默认值5%
        - -XX:G1MaxNewSizePercent	新生代最大值，默认值60%
      - mixed gc，除了回收整个young region，还会回收一部分的old region
        - -XX:InitiatingHeapOccupancyPercent，当老年代大小占整个堆大小百分比达到该阈值时，会触发一次mixed gc
      - full gc，采用Serial GC

   3. mixed gc过程：
      - 初始标记，只是标记一下GC Roots能直接关联到的对象
      - 根分区扫描Root Region Scanning，在初始标记暂停结束后，年轻代收集就完成的对象复制到Survivor的工作。这时为了保证标记算法的正确性，所有新复制到Survivor分区的对象，都需要被扫描并标记成根
      - 并发标记Concurrent Marking，标记线程与应用程序线程并行执行。
      - 再标记，再标记阶段是用来收集并发标记阶段产生新的垃圾(并发阶段和应用程序一同运行)
      - 筛选回收，在筛选回收阶段首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划

   4. 参数配置
      - -XX:+UseG1GC -Xmx32g -XX:MaxGCPauseMillis=200，设置GC的最大暂停时间为200ms.

>-Xmx最大堆,
>
>-Xms：初始堆大小,
>
>-Xmn:年轻代大小,
>
>-XXSurvivorRatio年轻代中Eden区与Survivor区的大小比值,注意，Survivor有2个，这里是一个的比值。

## 类加载

[Java](https://dzone.com/articles/jvm-architecture-explained) 虚拟机使用 Java 类的[方式](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html)如下：Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class 文件）。类加载器负责读取 Java 字节代码，并转换成 java.lang.Class类的一个实例。每个这样的实例用来表示一个 Java 类。通过此实例的 newInstance()方法就可以创建出该类的一个对象。而每个 Java 类都维护着一个指向定义它的类加载器的引用，通过 getClassLoader()方法就可以获取到此引用。

<img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/类加载.png" width=70% height=70% />

1. [加载](https://www.cnblogs.com/chanshuyi/p/the_java_class_load_mechamism.html)：
   - 将字节码转化成一个代表该类型的二进制数据流，加载到内存中
   - 方法区创建一个对应的 Class 对象，这个 Class 对象就是这个类各种数据的访问入口
2. 验证，为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。
   - JVM规范校验
   - 代码逻辑校验
3. 准备，重点。
   - 正式为类变量static分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。
   - 初始化的类型。为变量赋予 Java 语言中该数据类型的零值，而不是用户代码里初始化的值。
4. 解析，虚拟机将常量池内的符号引用替换为内存中的直接引用的过程
5. 初始化，重点。JVM 会根据语句执行顺序对类对象进行初始化
   - 使用new关键字实例化对象的时候、读取或设置一个类的静态字段
   - 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化
   - 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
   - 用户需要指定一个要执行的主类(包含main()方法的类)，虚拟机会先初始化这个类。


### 类加载器

1. 启动类加载器（Bootstrap ClassLoader）将存放在\lib目录的类库加载到虚拟机内存中

2. 扩展类加载器（Extension ClassLoader）：它负责加载\lib\ext

3. 应用程序类加载器AppClassLoader，负责加载用户类路径（ClassPath）上所指定的类库,

4. 双亲委派模型的工作过程： 如果一个类加载器收到了类加载的请求，先把这个请求委派给父类加载器去完成（所以所有的加载请求最终都应该传送到顶层的启动类加载器中），只有当父加载器反馈自己无法完成加载请求时，子加载器才会尝试自己去加载。即，真正完成类的加载工作是通过调用 defineClass来实现的；而启动类的加载过程是通过调用 loadClass来实现的。前者称为一个类的定义加载器defining loader，后者称为初始加载器initiating loader。一个类的定义加载器是它引用的其它类的初始加载器。(一个类引用了其他类，即最终加载这个类的加载器是引用类的初始加载器)如类 com.example.Outer引用了类 com.example.Inner，则由类 com.example.Outer的定义加载器负责启动类 com.example.Inner的加载过程。

   > 方法 loadClass()抛出的是 java.lang.ClassNotFoundException异常；方法 defineClass()抛出的是 java.lang.NoClassDefFoundError异常。在遇到 ClassNotFoundException和 NoClassDefFoundError等异常的时候，应该检查抛出异常的类的类加载器和当前线程的上下文类加载器，从中可以发现问题的所在。

5. [如何打破该机制](https://blog.csdn.net/zhouxcwork/article/details/81566636)：

   - 沿用双亲委派机制自定义类加载器很简单，只需继承ClassLoader类并重写findClass方法即可。此方法要做的事情是读取Test.class字节流并传入父类的defineClass方法即可。java.lang.ClassLoader类的方法 loadClass()会首先调用 findLoadedClass()方法来检查该类是否已经被加载过；如果没有加载过的话，会调用父类加载器的 loadClass()方法来尝试加载该类；如果父类加载器无法加载该类的话，就调用 findClass()方法来查找该类。因此，为了保证类加载器都正确实现代理模式，在开发自己的类加载器时，最好不要覆写 loadClass()方法，而是覆写 findClass()方法。
   - 打破：除了重写findClass方法外还重写了loadClass方法，默认的loadClass方法是实现了双亲委派机制的逻辑，即会先让父类加载器加载，当无法加载时才由自己加载。这里为了破坏双亲委派机制必须重写loadClass方法，即这里先尝试交由System类加载器加载，加载失败才会由自己加载。它并没有优先交给父类加载器，这就打破了双亲委派机制。

6. Java 虚拟机是如何判定两个 Java 类是相同的。Java 虚拟机不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。

   package com.example; 

   ```java
   public class Sample { 
      private Sample instance; 
   
      public void setSample(Object instance) { 
          this.instance = (Sample) instance; 
      } 
   }
       //测试 类是否相同
   public void testClassIdentity() { 
      String classDataRootPath = "C:\\workspace\\Classloader\\classData"; 
      //2个类加载器从同一个文件下加载同一个类的定义
      FileSystemClassLoader fscl1 = new FileSystemClassLoader(classDataRootPath); 
      FileSystemClassLoader fscl2 = new FileSystemClassLoader(classDataRootPath); 
      String className = "com.example.Sample";    
      try {
          Class<?> class1 = fscl1.loadClass(className); 
          Object obj1 = class1.newInstance(); 
          Class<?> class2 = fscl2.loadClass(className); 
          Object obj2 = class2.newInstance(); 
          //加载后的两个对象，试图相互转化，将报错java.lang.ClassCastException
          Method setSampleMethod = class1.getMethod("setSample", java.lang.Object.class); 
          setSampleMethod.invoke(obj1, obj2); 
      } catch (Exception e) { 
          e.printStackTrace(); 
      } 
   }
   ```
   
## 实战
### 解决堆内存问题
1. Windows查进程号jps或者jconsole
2. `jmap –heap $PID`查看堆内存，关注年轻、老年代占用。
3. `jmap –histo $PID | head -10`占用内存最多的对象
4. `jstack $PID`查看线程数是否正常
5. `jstat –gcutil`查看GC情况，如果Full GC过多，值得关注
### 解决CPU高等线程问题
1. top查看CPU高的进程pid
1. jstack pid，查看dump信息。查看是否出现deadlock关键字。没有的话，查看该进程中所有线程名，看运行到哪了。查看相关代码。
2. 由于jstack.log里面线程id是16进制，需要对线程id 做16进程转换：
   1. top -Hp pid查看占有CPU最多的线程的nid。
   2. printf "%x\n" nid，获取线程ID的十六进制。
   3. kill -3 pid打印线程dump信息，然后查看线程ID对应的信息，进行分析。
