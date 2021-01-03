## Nginx

1. [nginx](https://www.songma.com/news/txtlist_i29104v.html)模块一般被分成三大类：handler、filter和upstream。其中upstream模块，将使nginx跨越单机的限制，完成网络数据的接收、处理和转发。

### 负载均衡涉及到的算法

1. 轮询Round Robin， 对所有的后端服务端backend轮训发送请求，默认的分配方式。
2. weight，跟踪backend当前的活跃连接数目，最少的连接数目说明这个backend负载最轻，将请求分配给他。这种方式会考虑到配置中给每个upstream分配的weight权重信息；
3. IP Hash(ip_hash)，每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题
4. Generic Hash(hash),以客户自己设置资源(比方URL)的方式计算hash值完成分配，其可选consistent关键字支持一致性hash特性；

### 会话一致性

1. 原始负载均衡的基本原理：当客户端连接请求过来时，负载均衡系统内部会专门有一张表来记录这些连接的状况:

   [源IP：端口]、[目的IP：端口]、[服务器IP：端口]、空闲超时时间（Idle Timeout）
   这张表的大小一般称之为最大并发连接数，也就是系统同时能够容纳的连接数量。在该表中，设计了一个空闲超时时间的参数，即当该连接在一定时间内无流量通过时，负载均衡会自动删除该连接条目，释放系统资源。

2. 会话保持机制的意义就在于，确保将来自相同客户端的请求，转发至后端相同的服务器进行处理。如果在客户端和服务器之间部署了负载均衡设备，很有可能，这多个连接会被转发至不同的服务器进行处理。如果服务器之间没有会话信息的同步机制，会导致其他服务器无法识别用户身份。

### 后端服务端的动态配置

1. 出问题的backend要能被及时探测并剔除出分配群，而当业务增长的时候可以灵活的增加backend数目

### 基于DNS的负载均衡

### Nginx中的负载均衡

1. 会话一致性：在第一次分配之后，该会话的所有请求都分配到那个相同的backend上面。
   - Cookie Insertion。在backend第一次response之后，会在其头部增加一个session cookie，之后用户端接下来的请求都会带有这个cookie值，Nginx可以根据这个cookie判断需要转发给哪个backend了。
   - Sticky Routes。在backend第一次response之后，会产生一个route信息，route信息通常会从cookie/URI信息中提取。
   - Learn。Nginx会自动监测request和response中的session信息，而且通常需要回话一致性的请求、应答中都会带有session信息，这和第一种方式相比是不用添加cookie，而是动态学习已有的session。
2. backend健康监测
3. 通过DNS设置HTTP负载均衡
4. TCP/UDP流量的负载均衡