## 基础概念
### web 基础
1. 资源
   1. 静态资源：所有用户访问，得到的结果都一样。该类资源可直接被浏览器解析。如HTML、js、CSS、jpg
   2. 动态资源：每个用户访问相同的资源，得到的结果可能不一样。动态资源被访问后，需要先转换为静态资源，然后返回给浏览器解析。如servlet/jsp、PHP。
## Tomcat
### tomcat架构
1. Tomcat 的核心是两个组件：Connector 和 Container。

    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/Tomcat/1.PNG" width=80% height=80% />
2. 连接器，称coyote。作为服务器提供给客户端访问的外部接口，客户端通过coyote与服务器建立连接、发送请求并接受响应。coyote封装了底层网络通信（socket请求以及响应处理），使得Catalina容器与具体的请求协议以及IO操作方式完全解耦。coyote将socket输入转换封装尾request对象，交给Catalina容器进行处理，处理完请求后，Catalina再通过coyote提供的response对象将结果写入输出流。

    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/Tomcat/2.PNG" width=80% height=80% />
3. 连接器组件
   1. EndPoint,通信端点，即监听请求。
   2. Processor，将客户端发来的socket请求，转换为http请求，并对其进行解析，封装成request对象
   3. Adapter，在连接器与容器之间适配。将request转换为servletRequest对象，去调用容器中的方法。
   
     <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/Tomcat/3.PNG" width=80% height=80% />
4. Container
   1. Engine，表示整个Catalina的servlet引擎，管理多个虚拟站点，这里没理解
   2. Host，代表一个虚拟主机，可以给Tomcat配置多个虚拟主机地址，这里没理解.就是可以配置多个地址，比如配置localhost或者www.baidu.com主机地址
   3. Context，代表一个web应用程序。
   4. Wrapper，即具体servlet
### 源码
1. Lifecycle接口，由于所有组件均存在初始化、启动、停止等生命周期方法，拥有生命周期管理的特性，因此基于此抽象成该接口。
   1. init()初始化组件
   2. start()启动组件
   3. stop()停止组件
   4. destory()销毁组件
5. Lifecycle 接口的方法的实现都在其它组件中，组件的生命周期由包含它的父组件控制，所以它的 Start 方法自然就是调用它下面的组件的 Start 方法，Stop 方法也是一样。
   1. 在 Server 中 Start 方法就会调用 Service 组件的 Start 方法:

            //StandardServer.Start
            public void start() throws LifecycleException {
                if (started) {
                    log.debug(sm.getString("standardServer.start.started"));
                    return;
                }
                lifecycle.fireLifecycleEvent(BEFORE_START_EVENT, null);
                lifecycle.fireLifecycleEvent(START_EVENT, null);
                started = true;
                synchronized (services) {
                    for (int i = 0; i < services.length; i++) {
                        if (services[i] instanceof Lifecycle)
                            ((Lifecycle) services[i]).start();
                    }
                }
                lifecycle.fireLifecycleEvent(AFTER_START_EVENT, null);
            }
   2. Server 的 Stop 方法代码如下:
   
            //StandardServer.Stop
            public void stop() throws LifecycleException {
                if (!started)
                    return;
                lifecycle.fireLifecycleEvent(BEFORE_STOP_EVENT, null);
                lifecycle.fireLifecycleEvent(STOP_EVENT, null);
                started = false;
                for (int i = 0; i < services.length; i++) {
                    if (services[i] instanceof Lifecycle)
                        ((Lifecycle) services[i]).stop();
                }
                lifecycle.fireLifecycleEvent(AFTER_STOP_EVENT, null);
             }
2. 请求处理。Tomcat利用Mapper组件实现：请求交付Wrapper容器的Servlet处理。首先Mapper组件保存了Web应用的配置信息（.xml），包括Host容器配置的域名、Context容器的Web应用路径、以及Wrapper容器的Servlet映射路径。如下图：
   1. 在Tomcat的server.xml的中`<Engine defaultHost="localhost" name="Catalina">`首先查找对应的Host，即定位到的资源是`<Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true">`,因为name=localhost，最终定位到的资源在webapps下面，
   2. 接着，根据URL，定位到具体应用。后面的具体Servlet的定位，则取决于应用的web.xml中的配置信息。
   
      <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/Tomcat/5.PNG" width=100% height=100% />
      
3. tomcat中的热加载，conf/Server.xml中的Context标签，通过设置reloadable。原理是StandardContext 的 backgroundProcess 方法中实现，在 ContainerBase 类中定义的内部类 ContainerBackgroundProcessor 被周期调用的，这个类是运行在一个后台线程中，它会周期的执行 run 方法，它的 run 方法会周期调用所有容器的 backgroundProcess 方法，因为所有容器都会继承 ContainerBase 类，所以所有容器都能够在 backgroundProcess 方法中定义周期执行的事件。于是完成热部署

         <Context
             path="/library"
             docBase="D:\projects\library\deploy\target\library.war"
             reloadable="true"
         />
1 热部署：在server.xml -> context 属性中设置`autoDeploy="true"`，

      <Context docBase="xxx" path="/xxx" autoDeploy="true"/> 
1. 区别：
   1. 热加载：服务器会监听 class 文件改变，包括web-inf/class,wen-inf/lib,web-inf/web.xml等文件，若发生更改，则局部进行加载，不清空session ，不释放内存。
   2.  热部署： 整个项目从新部署，包括你从新打上.war 文件。 会清空session ，释放内存。项目打包的时候用的多。
### jasper
1. Tomcat在默认的web.xml中配置了一个JSPServlet的类，用于处理*.jsp\*.jspx结尾的请求。另外，在conf/web.xml文件的末尾<welcome-file-list>写入了默认的页面index.jsp，即输入路径为null时默认请求地址为index.jsp。index_jsp.java在Tomcat的work/localhost路径下
   
   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/Tomcat/6.PNG" width=80% height=80% />
2. 编译结果：
   1. 如果在conf/web.xml中配置了参数scratchdir，则编译后的结果存储在该目录下
   
            <init-param>
                 <param-name>scratchdir</param-name>
                 <param-value>D:/tmp/jsp/</param-value>
            </init-param>
   2. 如果没有配置该选项，则会将编译结果存储Tomcat的安装目录work/Catalina(Engine名称)/localhost(Host)/Context名称
   3. 如果使用IDEA开发集成的Tomcat访问Web中的jsp，则编译结果：`C:\Users\Administrator\.Intel1iJIdea2019.l\system\tomcat\ _project_tomcat\work\Catalina\localhost\jsp_demo_01_wa r_exploded\org\apache\jsp`
   
   
   
   
### Tomcat服务器配置
主要集中于/conf下的文件

1. server.xml，Server作为根节点，`<Server port="8005" shutdown="SHUTDOWN">`表示执行shutdown时监听该行为的端口号为8005：
    1. 接着配置了5个监听器。VersionLoggerListener用于日志输出服务器、操作系统、JVM版本信息，AprLifecycleListener用于加载服务器启动和销毁APR，如果找不到APR库，则输出日志，但不影响Tomcat启动。
    1. Service,创建Service实例。
       - Executor被注释了，表示多个连接器使用单独的线程池，取消注释则表明多个连接器公用一个线程池。
       - Connector，默认创建2个实例，以后支持HTTP，一个支持AJP，port用于创建服务器socket并进行监听。redirectPort,即当前Connector不支持SSL请求，会自动重定向到该端口号。URIEncoding:用于指定编码URI的字符编码，Tomcat8以上默认UTF-8，get请求乱码不用处理，但7及以下默认ISO-8859-1
       - Engine：defaultHost属性表示默认访问该主机下的信息。这里可以自己添加虚拟主机。
       - Context：应用存放docBase路径，或者war包的部署路径。path是web应用的Context路径。即主机地址+path为访问路径。
### web应用配置
1. web.xml
   1. <context-param>标签为ServletContext初始化参数，这样我们可以在javax.servlet.Servletcentext.getInitParameter()方法获取参数。
   
### Tomcat管理配置
1. webapps下的host-manager管理虚拟主机信息，而manager管理web应用。
### JVM配置
1. windows下的bin/catalina.bat文件下进行设置。配置好后，在Tomcat的主页右上角点击Server Status可以查到服务器的JVM信息。linxu应该就是.sh

### Tomcat集群
1. 在一台服务器上，安装2台Tomcat，conf/server.xml中分别修改三处端口号：
   1. Server标签的监听端口8005->8015\8025
   2. Connector标签的http协议的端口8080->8888\9999,这个就是后面连接访问的端口
   3. Connector标签的ajp协议的端口8009->8019\8029
2. 配置nginx，即conf/nginx.conf。当以localhost:99来访问时，访问到了nginx，最终则通过serverpool来找到Tomcat

         #Tomcat集群的nginx配置
         upstream serverpool{
            server localhost:8888;
            server localhost:9999;
         }
         server{
            listen 99;
            server_name localhost;
            # 路径是所有
            location / {
               # 代理地址
               proxy_pass http://serverpool/;
            }
         }
         
1. 测试：更改webapps/ROOT/index.jsp，后面访问的地址是在个，启动：bin/startup.bat。如果出现闪退。[解决方式](https://blog.csdn.net/znn626/article/details/7893555)。访问localhost:99就发现，由nginx负责负载均衡，有时候会访问不同内容。  
   - pause 防止命令框一闪而过
   - run 作用打印出 一闪而过的原因。
1. session复制。如果没有采用IP哈希的方式处理负载均衡，可能出现session不统一的问题。解决该问题：
   1. 需要在conf/server.xml中修改：在Engine标签中添加下面语句，通过TCP的形式进行广播复制

           <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
   2. 在应用程序中的web.xml中添加	`<distributable/>`标签
   3. 该方法仅适用于Tomcat集群数量不多的情况，太多复制耗费带宽，影响性能，基本不用。
1. SSO-单点登录

    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/Tomcat/7.PNG" width=80% height=80% />
    
    
### Tomcat安全
1. 管理页面不能被外部访问，即删除webapps下所有文件。原先的应用现在早被加载到内存了。
2. 监听的8005关闭端口需要禁用命令或者修改指令。
3. 定义错误页面，可以保证用户发生错误不会看到异常堆栈信息。
4. 支持HTTPS。需要生成秘钥库文件放置conf/下，然后配置server.xml，加入Connector标签。

### 性能优化
1. 性能测试
   - 响应时间：为执行某个操作，测试多次，求平均响应时间
   - 吞吐量：在给定时间内，系统支持的事务数量，单位TPS。
2. ApacheBench
   1. `ab -n 1000 -c 100 -p data.json -T application/json http://localhost:9000/course/search.do?page=1spagesize=10`
      - -p,post请求时需要json格式数据
      - -n,在测试会话中所执行的请求个数
      - -c,一次产生的请求个数（并发数）。默认是一次一个。
   2. request per second，吞吐量，服务器并发处理的量化描述。指的是某个并发用户数下单位时间内处理的请求数。越大越好
   
            This is ApacheBench, Version 2.3 <$Revision: 655654 $>
            Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
            Licensed to The Apache Software Foundation, http://www.apache.org/

            Benchmarking 10.51.170.39 (be patient)
            Completed 10000 requests
            Completed 20000 requests
            Completed 30000 requests
            Completed 40000 requests
            Completed 50000 requests
            Completed 60000 requests
            Completed 70000 requests
            Completed 80000 requests
            Completed 90000 requests
            Completed 100000 requests
            Finished 100000 requests

            Server Software:                         //测试服务端类型
            Server Hostname:        10.51.170.39     //测试服务器 HOST
            Server Port:            80               //测试服务端端口号

            Document Path:          /sport/home?_wv=2163715   //The request URI parsed from the command line string.
            Document Length:        62346 bytes               //This is the size in bytes of the first successfully returned document. If the document length changes during testing, the response is considered an error.

            Concurrency Level:      500                       //The number of concurrent clients used during the test
            Time taken for tests:   158.969 seconds           //This is the time taken from the moment the first socket connection is created to the moment the last response is received
            Complete requests:      100000                    //The number of successful responses received
            Failed requests:        0                         //The number of requests that were considered a failure. 
            Write errors:           0                         //he number of errors that failed during write (broken pipe).
            Keep-Alive requests:    100000                    //The number of connections that resulted in Keep-Alive requests
            Total transferred:      6266233588 bytes          //The total number of bytes received from the server. This number is essentially the number of bytes sent over the wire.
            HTML transferred:       6236217388 bytes          //The total number of document bytes received from the server. This number excludes bytes received in HTTP headers
            Requests per second:    629.05 [#/sec] (mean)     //This is the number of requests per second. This value is the result of dividing the number of requests by the total time taken
            Time per request:       794.847 [ms] (mean)       //The average time spent per request. The first value is calculated with the formula concurrency * timetaken * 1000 / done while the second value is calculated with the formula timetaken * 1000 / done
            Time per request:       1.590 [ms] (mean, across all concurrent requests)
            Transfer rate:          38494.01 [Kbytes/sec] received            //The rate of transfer as calculated by the formula totalread / 1024 / timetaken

            Connection Times (ms)
                          min  mean[+/-sd] median   max
            Connect:        0    0   2.1      0      34
            Processing:   296  793 151.5    809    2962
            Waiting:      232  690 153.2    713    2724
            Total:        296  793 152.1    809    2992

            //整个场景中所有请求的响应情况。在场景中每个请求都有一个响应时间，其中 50％的用户响应时间小于 809 毫秒，66％的用户响应时间小于 854 毫秒，最大的响应时间小于 2992 毫秒。对于并发请求，cpu 实际上并不是同时处理的，而是按照每个请求获得的时间片逐个轮转处理的，所以基本上第一个 Time per request 时间约等于第二个 Time per request 时间乘以并发请求数。
            Percentage of the requests served within a certain time (ms)
              50%    809
              66%    854
              75%    881
              80%    899
              90%    947
              95%    998
              98%   1075
              99%   1146
             100%   2992 (longest request)
### JVM调优
1. JVM内存参数
   1. -server,启动server，以服务器模式运行
   2. -Xms,最小堆内存，建议与-Xmx设置相同
   3. -Xmx,最大堆内存，建议设置为可用内存的80%，这个可用内存，因为Linux系统内存中还可能运行了其他的比如Redis等，因此可用内存要排除后剩余的内存。
   4. -XX:MetaspaceSize，元空间初始值
   5. -XX:MaxMetaspaceSize,元空间最大内存，默认无限
   6. -XX:MaxNewSize,新生代最大内存，默认16M
   7. -XX:NewRatio,年轻代：老年代，取值整数，默认2，不建议修改
   8. -XX:SurvivorRatio,Eden:Survivor,取值整数，默认8.不建议修改
2. Linux修改配置：在bin/catalina.sh中加入`JAVA_OPTS="-server-Xms2048m -Xmx2048m -XX:Metaspaces1ze=256m -XX:MaxMetaspaceSize=512m -XX:SurvivorRatlo=8"`,当然当前服务器的内存可以查看，通过free命令查到free可用内存大小后计算得出。
   - 具体查看占用多少内存，可以通过`jmap -head Tomcat的PID`来查看
3. 查看垃圾收集情况：jconsole中的VM概要中，第三栏，第一个垃圾收集器为年轻代，选择copy复制算法，第二个为老年代。第二个指标为收集次数。
4. 连接器Connector配置，在conf/server.xml
   1. maxConnections,到达该值后，服务器接收单不会处理更多的请求，额外的请求将会阻塞直到连接数低于该值。可以通过`ulimit -a`查看服务器限制。对于CPU要求高（计算型），不能设大，要求不高一般2000
   2. maxThreads，最大线程数
   3. acceptCount,最大排队等待数，达到maxConnections后，Tomcat会将后面的请求，存放在任务队列中，acceptCount几句诗任务队列中等待的请求数。即一台Tomcat最大请求数=maxConnections+acceptCount
   4. 直接在Connector标签中添加属性即可。
   
   
## webSocket
1. websocket是H5的新增协议，目的是浏览器与服务器之间建立不受限的双向通信通道，即双方都可以主动发送请求给对方。
2. websocket必须由浏览器先发起。请求头中的Connection为upgrade，于是将请求转化为websocket请求，建立长连接。
3. 该请求与HTTP不同在于：
   1. get请求与地址不是`http://`,而是`ws://`开头
   2. HTTP状态码为101，表明服务器已经识别冰切换为websocket协议。
### Tomcat的websocket
1. Java websocket应用由一系列的websocketEndPoint组成：
   1. 编程式：继承javax.websocket.EndPoint
   2. 注解：定义一个Pojo，添加@ServerEndPoint注解。
   3. EndPoint实例在websocket握手期间创建，最后连接关闭时结束，有这些方法：
      - onOpen,当开启一个新会话时调用，握手成功后调用该方法
      - onClose，会话关闭时调用
      - onError，连接过程异常时调用
