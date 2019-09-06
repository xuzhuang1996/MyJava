- [计算机网络](#计算机网络)
- [数据库](#数据库)
   - [mysql索引](#mysql索引)
- [操作系统](#操作系统)
- [spring](#spring)
- [集合框架](#集合框架)
- [JVM](#jvm)
  - [类加载](#类加载)
- [异常](#异常)
- [IO](#io)
- [Nginx](#nginx)
- [Redis](#redis) 
- [问答题](#题) 

# 计算机网络
## 浏览器从输入网址到显示都发生了些什么？
1. 输入网址www.baidu.com 。 如果没有指明是https默认http协议。
2. 浏览器查找域名的IP地址。即应用层DNS解析获取IP地址
   -  浏览器缓存：浏览器会在一定时间内缓存DNS记录（2min-30min不等）
   -  系统缓存：操作系统也会缓存一定的DNS记录。浏览器缓存找不到的时候会查找系统里的缓存记录。一般在hosts文件
   -  本地域名服务器ISP DNS：查询被发送到本地域名服务器，如果依然解析不了，将作为代理，进行迭代或者递归查询：
       - 客户端查询根服务器，找到com域名解析服务器。
       - 客户端查询com顶级域名服务器，找到baidu.com权威域名服务器。
       - 客户端查询baidu.com权威域名服务器，获得www.baidu.com的IP地址。
3. 根据IP地址，向web服务器发送HTTP请求request。
4. 传输层首先需要建立TCP连接，采用三次握手确认建立一个连接。然后才能将应用层发送的报文分成段segment，交给网络层。
5. 网络层将segment封装到datagram。利用ARP协议将IP地址解析为MAC地址。然后路由器利用路由表中的目的网络的下一步路径信息进行转发，在网络中传递。
6. 链路层，将根据MAC地址将数据进行传输。这时，客户端发送请求的过程的阶段才结束。
7. 服务端接收数据
8. 服务端处理请求
9. 服务端返回一个HTTP响应请求
10. 浏览器开始渲染拿到的HTML文件

## http请求内容
1. Host: www.study.com  // 请求的地址域名和端口，不包括协议
2. Connection: keep-alive  // 连接类型，持续连接
3. User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.96 Safari/537.36  //浏览器的用户代理信息
4. Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,`*/*`;q=0.8     //浏览器支持的请求类型
5. Accept-Encoding: gzip, deflate, sdch   //浏览器能处理的压缩代码
5. Accept-Language: zh-CN,zh;q=0.8,en;q=0.6  //浏览器当前设置语言
## 转发(Forward)和重定向(Redirect)的区别
1. 重定向的执行过程：Web服务器向浏览器发送一个http响应->浏览器接受此响应后再发送一个新的http请求到服务器->服务器根据此请求寻找资源并发送给浏览器。它可以重定向到任意URL，不能共享request范围内的数据。
2. 重定向是在客户端发挥作用.
3. 通过浏览器重新请求地址，在地址栏中可以显示转向后的地址。
4. 转发过程：Web服务器调用内部方法在容器内部完成请求和转发动作->将目标资源发送给浏览器，它只能在同一个Web应用中使用，可以共享request范围内的数据。
5. 转发是在服务器端发挥作用，通过forward()方法将提交信息在多个页面间进行传递.
6. 转发是在服务器内部控制权的转移，客户端浏览器的地址栏不会显示出转向后的地址

## HTTP请求,响应报文格式
- HTTP请求报文主要由请求行、请求头部、请求正文3部分组成
- HTTP响应报文主要由状态行、响应头部、响应正文3部分组成
- https://blog.csdn.net/a19881029/article/details/14002273

## Http版本区别
1. http 1.0
   - 短连接。每一个请求建立一个TCP连接，请求完成后立马断开连接。这将会导致2个问题：
     - 连接无法复用，每次请求都经历三次握手和慢启动。
     - head of line blocking会导致带宽无法被充分利用
2. http 1.1
   - 长连接。通过http pipelining实现。多个http 请求可以复用一个TCP连接，服务器端按照FIFO原则来处理不同的Request
   - 增加connection header。该header用来说明客户端与服务器端TCP的连接方式，若connection为close则使用短连接，若connection为keep-alive则使用长连接
   - 身份认证
   - 状态管理
   - Cache缓存等机制相关的请求头和响应头
3. http 2.0
   - 多路复用 (Multiplexing).在 HTTP/1.1 协议中浏览器客户端在同一时间，针对同一域名下的请求有一定数量限制。超过限制数目的请求会被阻塞。而HTTP/2 的多路复用(Multiplexing) 则允许同时通过单一的 
   - 二进制分帧。HTTP/2在 应用层(HTTP/2)和传输层(TCP or UDP)之间增加一个二进制分帧层。

-----------------------------------------------------------------------------------
# 数据库
## 属性
- 实体完整性： 每一个表中的主键字段不能为空或者重复的值。实体完整性指表中行的完整性。要求表中的所有行都有唯一的标识符，称为主关键字；
- 参照完整性：要求通过定义的外关键字和主关键字之间的引用规则来约束两个关系之间的联系。包括更新规则，删除规则，和插入规则；
- 域完整性：针对某一具体关系数据库的约束条件，它保证表中某些列不能输入无效的值；
- 引用完整性：被引用表中的主关键字和引用表中的外部关键字之间的关系，如被引用行是否可以被删除等；
## ACID
1. Atomic（原子性），指整个数据库事务是不可分割的工作单位。事务中任何一个SQL语句执行失败，那么已经执行成功的SQL语句也必须撤销，数据库状态应该退回到执行事务前的状态。
2. Consistency（一致性），指数据库事务不能破坏关系数据的完整性以及业务逻辑上的一致性。比如对于入账出账操作是不会有总资金的变化的。
3. Isolation（隔离性），表示各个事务之间不会互相影响，数据库一般会提供多种级别的隔离。实际上多个事务是并发执行的，但是他们之间不会互相影响。
4. Durability（持久性），持久性表示一旦一个事务成功了，那么他的改变是永久性的被记录和操作。

## 数据库范式
1. 第一范式（1NF）：当关系模式R的所有属性都不能在分解为更基本的数据单位时，称R是满足第一范式
   - 比如电话属性，座机跟手机都属于电话，但是表中不能将其合在一起，而是分成2个属性。
2. 第二范式（2NF）：符合1NF，并且，非主属性完全依赖于码。所谓完全依赖是指不能存在仅依赖主关键字一部分的属性，
3. 第三范式（3NF）：符合2NF，并且，消除传递依赖（在"A → B → C"的决定关系，则C传递函数依赖于A）,即任何非主属性不依赖于任一侯选关键字
4. [BC范式](https://blog.csdn.net/ai549281110/article/details/40040963)（BCNF）：符合3NF，并且，任何字段都不能传递依赖任一侯选关键字(与第三范式相比，一个是“任何非主属性不能”，一个是“任何字段不能”)。一般关系型数据库设计中，达到BCNF就可以了！若一个关系达到了第三范式，并且它只有一个候选码，或者它的每个候选码都是单属性，则该关系自然达到BC范式。

> 侯选关键字：又叫侯选码，惟一标识一行数据，其真子集不能是侯选关键字，一个表可以存在多个侯选关键字，如用户表的username，userid

## sql优化
1. SQL语句全部大写 （所有SQL语句执行时都会转换成大写）
2. 避免使用星号 * ,用具体字段替代（使用星号会在查询时增加一个查询列的操作）
3. 尽量避免在 where 子句中对字段进行 null 值判断，最好不要给数据库留NULL，尽可能的使用 NOT NULL填充数据库
4. in 和 exists的区别: 如果子查询得出的结果集记录较少，主查询中的表较大且又有索引时应该用in
5. [count()](https://msd.misuland.com/pd/2884250068896976750)函数。COUNT()有两个非常不同的作用：它可以统计某个列值的数量，也可以统计行数。在统计列值时要求列值是非空的（不统计NULL）。**如果在COUNT()的括号中定了列或者列表达式，则统计的就是这个表达式有值的结果数。**COUNT()的另一个作用是统计结果集的行数。当MySQL确认括号内的表达式值不可能为空时，实际上就是在统计行数。最简单的就是当我们使用COUNT(*)的时候，这种情况下通配符*并不像我们猜想的那样扩展成所有的列，实际上，他会忽略所有列而直接统计所有的行数.
   - MyISAM。MyISAM在统计表的总行数的时候会很快，但是有个大前提，不能加有任何WHERE条件。这是因为：MyISAM对于表的行数做了优化，具体做法是有一个变量存储了表的行数
   - Innodb。 5.5及以后，是没有什么区别的。通常，我们将第一个字段（一般是ID）作为主键，那么这个时候COUNT(1)实际统计的就是行数。一个优化方案就是预先建一个小字段并建二级索引专门用来统计行数.
   - 任何情况下`select count(*) from tablename`是最优选择；
   - 尽量减少`select count(*) from tablename where COL = ‘value’`这种查询；
   - 杜绝`select count(COL) from tablename where COL2 = ‘value’`的出现。
1. `limit 1000000 10`如何进行优化
   - 记录每次取出后的最大id 然后 where id = 最大id limit 10；
2. 在使用查询的时候遵循mysql组合索引的"[最左前缀](https://www.jianshu.com/p/9b3406bcb199)":索引where时的条件要按照建立索引的时候字段的排序方式
   - 如果有一个 2 列的索引 (col1, col2)，则已经对 (col1)、(col1, col2) 上建立了索引；
   - 如果有一个 3 列索引 (col1, col2, col3)，则已经对 (col1)、(col1, col2)、(col1, col2, col3) 上建立了索引；
     >比如当 (张三,20,F) 这样的数据来检索的时候，b+ 树会优先比较 name 来确定下一步的所搜方向，如果 name 相同再依次比较 age 和 sex，最后得到检索的数据；但当 (20,F) 这样的没有 name 的数据来的时候，b+ 树就不知道第一步该查哪个节点，因为建立搜索树的时候 name 就是第一个比较因子，必须要先根据 name 来搜索才能知道下一步去哪里查询
3. like 语句的索引问题:
   - 如果通配符 % 不出现在开头，则可以用到索引
   - 在 like “value%” 可以使用索引，但是 like “%value%” 不会使用索引，走的是全表扫描
5. 尽量选择区分度高的列作为索引.表示字段不重复的比例，比例越大我们扫描的记录数越少.区分度太小或重复率过高会使索引失效，进而走全表扫描。
## sql语法：
1. desc\asc.用于降序升序。语法：`field desc`.有时候可能需要在前面加`,`
1. having，通常与GROUP BY语句联合使用，用来过滤由GROUP BY语句返回的记录集。如果想查询平均分高于80分的学生记录可以这样写，这里如果是where就出错：

		//group by 中的参数，必须是select查询的结果中的的参数才可以
		SELECT id, COUNT(course) as numcourse, AVG(score) as avgscore

		FROM student

		GROUP BY id

		HAVING AVG(score)>=80;
2. 连接
   - 内连接: 只连接匹配的行。[可以处理left连接为空的情况](https://leetcode-cn.com/problems/department-highest-salary/)。
   - 左外连接: 包含左边表的全部行（不管右边的表中是否存在与它们匹配的行），以及右边表中全部匹配的行。另外，on表示连接条件，[不要把where中的语句也放在on中，应该单独](https://www.nowcoder.com/practice/6d35b1cd593545ab985a68cd86f28671?tpId=82&tqId=29756&tPage=1&rp=&ru=%2Fta%2Fsql&qru=%2Fta%2Fsql%2Fquestion-ranking)。另外，连接的时候，查出来的数据只保证[on条件上的数据相同，其他相同字段不一定相同](https://www.nowcoder.com/practice/4c8b4a10ca5b44189e411107e1d8bec1?tpId=82&tqId=29761&tPage=1&rp=&ru=%2Fta%2Fsql&qru=%2Fta%2Fsql%2Fquestion-ranking)
     - `SELECT a.,b. FROM luntan LEFT JOIN usertable as b ON a.username=b.username`
   - 右外连接: 包含右边表的全部行（不管左边的表中是否存在与它们匹配的行），以及左边表中全部匹配的行
   - 全外连接: 包含左、右两个表的全部行，不管另外一边的表中是否存在与它们匹配的行。
     - `SELECT a.,b. FROM city as a FULL OUTER JOIN user as b ON a.username=b.username`
   - 交叉连接: 生成笛卡尔积－它不使用任何匹配或者选取条件，而是直接将一个数据源中的每个行与另一个数据源的每个行都一一匹配
     - `SELECT type,pub_name FROM titles CROSS JOIN publishers ORDER BY type`
3. limit,初始行的偏移量为0，即limit的第一个参数类似数组。[求数据中的第二高的数据](https://leetcode-cn.com/problems/second-highest-salary/)
   - `SELECT * FROM tbl LIMIT 5,10;  # Retrieve rows 6-15`
   - `LIMIT row_count OFFSET offset`这个offset是为了与别的数据库进行兼容。用`,`也行.
4. 防止取出的数据为null，使用`IFNULL(语句，NULL)`
5. if函数。mysql中if()函数的用法类似于java中的三目表达式
   
   		update salary
		set sex=(if(sex='m','f','m'))
		
		
6. `group by`与`order by`.
   - `order by`行的排序方式，默认的为升序
   - `group by`分组,必须要用聚合函数
      - group by 可以实现一个最简单的去重查询
      - 分组后的条件使用 HAVING 来限定，WHERE 是对原始数据进行条件限制。这里分组，比如统计网站用户点击方式，用name来作为分组字段，然后对某一用户行为进行统计
      - group by 中的参数，必须是select查询的结果中的的参数才可以
   - [组外排序+组内排序](https://www.nowcoder.com/practice/ae51e6d057c94f6d891735a48d1c2397?tpId=82&tqId=29760&tPage=1&rp=&ru=%2Fta%2Fsql&qru=%2Fta%2Fsql%2Fquestion-ranking)。
   
			group by packageId
			order by activeRate desc
7. `datediff(date1,date2)`输出date1-date2的天数。有正负之分。[用途](https://leetcode-cn.com/problems/rising-temperature/submissions/)
8. `distinct`，筛掉重复的。主要是放的位置，可以放在select后面的列前面。也可以如：`count(distinct student)>=5`
9. `case`.Case函数只返回第一个符合条件的值，剩下的Case部分将会被自动忽略。Case when 相当于一个自定义的数据透视表，group by 是行名，case when 负责列名。
	
		case 
  			when sex = '1' then '男'
  			when sex = '2' then '女'
		else '未知' end
		//实际例子
		update salary
		set sex=(
		    CASE 
			when sex="m" then "f"
		    else
			"m"
		    end
		)
10. case:

		//当colume 与condition 条件相等时结果为result\当满足某一条件时，执行某一result
		case colume 
		    when condition then result
		    when condition then result
		    when condition then result
		else result
		end
		//当满足某一条件时，执行某一result,把该结果赋值到new_column_name 字段中
		case  
		    when condition then result
		    when condition then result
		    when condition then result
		else result
		end new_column_name
		
		
11. `ON DUPLICATE KEY UPDATE`.[该语句](https://www.cnblogs.com/zjdxr-up/p/8319982.html)是基于唯一索引或主键使用，不能写where条件的;比如一个字段a被加上了unique index，并且表中已经存在了一条记录值为1，下面两个语句会有相同的效果：

		INSERT INTO table (a,b,c) VALUES (1,2,3)  
		  ON DUPLICATE KEY UPDATE c=c+1;  

		UPDATE table SET c=c+1 WHERE a=1;
		
12. 求极值：[一般where中嵌套去查询极值](https://www.nowcoder.com/questionTerminal/218ae58dfdcd4af195fff264e062138f)，或者desc排序
13. UNION 操作符用于**合并**两个或多个 SELECT 语句的结果集。如果允许重复的值，请使用 UNION ALL。应用：列出各项清单以及总计。

		SELECT column_name(s) FROM table_name1
		UNION
		SELECT column_name(s) FROM table_name2
## 分库分表
1. [来源](https://www.nowcoder.com/discuss/135748)
2. 数据切分分为两种方式，纵向切分和水平切分
   - 纵向切分
     - 纵向分库，根据业务耦合性，将关联度低的不同表存储在不同的数据库
     - 纵向分表，基于数据库中的列进行，某个表字段较多，可以新建一张扩展表，将不经常用或者字段长度较大的字段拆出到扩展表中。适用：如果一个表中某些列常用，另外一些列不常用
   - 水平切分
     - 库内分表，将同一个表按不同的条件分散到多个数据库或多表中，每个表中只包含一部分数据，从而使得单个表的数据量变小，达到分布式的效果。适用：表中的数据本身就有独立性，例如表中分表记录各个地区的数据或者不同时期的数据，特别是有些数据常用，有些不常用。
     - 分库分表
3. 分库分表带来的问题
   - join操作。水平分表后，虽然物理上分散在多个表中，如果需要与其它表进行join查询，需要在业务代码或者数据库中间件中进行多次join查询，然后将结果合并。
   - `COUNT（*）`，水平分表后，某些场景下需要将这些表当作一个表来处理，那么`count(*)`显得没有那么容易了。
   - order by,分表后，数据分散到多个表中，排序操作无法在数据库中完成，只能由业务代码或数据中间件分别查询每个子表中的数据，然后汇总进行排序。
4. 设定网站用户数量在千万级，但是活跃用户数量只有1%，如何通过优化数据库提高活跃用户访问速度？
   - 可以使用MySQL的分区，把活跃用户分在一个区，不活跃用户分在另外一个区，本身活跃用户区数据量比较少，因此可以提高活跃用户访问速度。
   - 还可以水平分表，把活跃用户分在一张表，不活跃用户分在另一张表，可以提高活跃用户访问速度。

## mysql索引
索引的原理：对要查询的字段建立索引其实就是把该字段按照一定的方式排序；建立的索引只对该字段有用。
### b加树简介
1. 什么是B+树：[B+大致认识](https://blog.csdn.net/qq_26222859/article/details/80631121)
    >节点的度：一个节点含有的子树的个数称为该节点的度；
    >树的度：一棵树中，最大的节点的度称为树的度
2. 特点：
   - 每个父节点的元素都出现在子节点中，是子节点的最大最小值。因此所有叶子节点包含全部元素信息。
   - 只有叶子节点所带的索引元素才指向数据记录，其余中间节点只是索引，没有与任何数据关联。因此，B+树的查询必须最终查到叶子节点，查询性能稳定
   - 所有叶子节点形成有序链表，便于范围查询
2. [B+树插入过程](http://www.cnblogs.com/yangecnu/p/Introduce-B-Tree-and-B-Plus-Tree.html)如下图1：
    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/1-B+树插入过程.gif" width=130% height=130% />
3. [B+的插入原则：](https://blog.csdn.net/sunshine_lyn/article/details/82747596)
    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/B+++.png" width=100% height=100% />
4. 一般在数据库系统或文件系统中使用的B+Tree结构都在经典B+Tree的基础上进行了优化，增加了顺序访问指针。做法：在B+Tree的每个叶子节点增加一个指向相邻叶子节点的指针，就形成了带有顺序访问指针的B+Tree。做这个优化的目的是为了提高区间访问的性能，例如如果要查询key为从18到49的所有数据记录，当找到18后，只需顺着节点和指针顺序遍历就可以一次性访问到所有数据节点，极大提到了区间查询效率。如下图2：

    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/2-B+树优化后.png" width=60% height=60% />
### mysQL索引实现
[来源](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)
#### MyISAM索引实现
1. 非聚集索引，MyISAM索引文件和数据文件是分离的，索引文件仅保存数据记录的地址。如下图3，左边是索引文件，右边是数据文件，一行代表一个数据。MyISAM中索引检索的算法为首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域的值为地址，读取相应数据记录。

    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/3-MyISAM索引实现.png" width=55% height=55% />
2. 假设我们以Col1为主键，则上图是一个MyISAM表的主索引（Primary key）示意，因为通过主键建立的索引，这样主键对应的地址唯一。但如果想建立辅助索引，如在Col2建立索引，因为不是主键所以Col2可能重复。
>聚集索引和非聚集索引的根本区别是表记录的排列顺序和与索引的排列顺序是否一致
#### InnoDB索引实现
1. 聚集索引，在InnoDB中，表数据文件本身就是按B+Tree组织的一个索引结构，这棵树的叶节点data域保存了完整的数据记录。如下图4。一行数据直接在叶子节点里面，因此InnoDB要求表必须有主键。

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/4-InnoDB.png" width=55% height=55% />
2. 如果建立辅助索引，如下图5，Col3作索引，InnoDB的所有辅助索引都引用主键作为data域。辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/5-InnoDB辅助索引.png" width=55% height=55% />

### 行级锁
1. mysql的行级锁是针对索引，InnerDB支持行锁也支持表锁。只有通过**索引条件检索**数据，InnoDB才使用行级锁（insert没有而updata操作具有行级索），否则，InnoDB将使用表锁！行级锁的缺点是：由于需要请求大量的锁资源，所以速度慢，内存消耗大，并且可能导致大量的锁冲突，从而影响并发性能。
2. 在查询语句后面增加for update，数据库会在查询过程中给数据库表增加排他锁。当某条记录被加上排他锁之后，其他线程无法再在该行记录上增加排他锁。其他没有获取到锁的就会阻塞在上述select语句上，可能的结果有2种，在超时之前获取到了锁，在超时之前仍未获取到锁。
3. 行级锁分为共享锁 和 排他锁

#### 回答
1. 索引是如何提高查询速度的？使用了B+树，根据索引的Key值建立B+树，将无序的数据变成相对有序的数据。
2. 例如如果要查询key为从18到49的所有数据记录，根据B+树的特性，当找到18后，只需顺着节点和指针顺序遍历就可以一次性访问到所有数据节点，极大提到了区间查询效率。
3. 使用一个与业务无关的自增字段作为主键。理由：如果表使用自增主键，那么每次插入新的记录，记录就会顺序添加到当前索引节点的后续位置，当一页写满，就会自动开辟一个新的页；如果使用非自增主键（如果身份证号或学号等），由于每次插入主键的值近似于随机，因此每次新纪录都要被插到现有索引页得中间某个位置。
4. 不建议使用过长的字段作为主键：因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大。自增字段作为主键则是一个很好的选择。
5. 覆盖索引：我们知道在InnoDB存储引擎中，如果不是主键索引，叶子节点存储的是主键+列值。最终还是要“回表”。覆盖索引就是select的数据列只用从索引中就能够取得，不必从数据表中读取。
6. MyISAM索引实现不支持事务，用于只读程序提高性能，而InnoDB：支持ACID事务、行级锁、并发。 
7. 在mysql中，查询某字段为空时，切记不可用 = null，而是 is null，不为空则是 is not null
8. varchar和char的使用场景:
   - char的长度是不可变的，而varchar的长度是可变的
   - char的存取速度还是要比varchar要快得多，因为其长度固定，方便程序的存储与查找。
9. 适合与否索引
   - 适合索引：
     - 在经常需要搜索的列上，可以加快搜索的速度
     - 在经常需要排序的列上创建索引，因为索引已经排序，这样查询可以利用索引的排序，加快排序查询时间；
     - 在经常需要根据范围进行搜索的列上创建索引，因为索引已经排序，其指定的范围是连续的。用的是B+树的底层链表。
   - 不适合索引：
     - 对于那些在查询中很少使用或者参考的列不应该创建索引
     - 对于那些只有很少数据值的列也不应该增加索引。例如人事表的性别列
     - 对于那些定义为text, image和bit数据类型的列不应该增加索引
10. [Index Condition Pushdown（索引下推）](https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650124338&idx=1&sn=ac152b7bdf4b1ad16827b7851f6170ff&chksm=f36bad13c41c2405aae50eb6c8a1832bc668274e14182e1ca29c4a59c845060aac149dc0967e&scene=27#wechat_redirect)
    - ICP在用第一个索引获取数据的同时进行筛选后面的数据，实现了where的次选条件中无法直接使用索引的情况下的筛选，实现"非直接索引"过滤条件的筛选,避免了没有ICP优化的时候分两个步骤的实现
    - 如果是非ICP优化查询的话，是两步，第一步是获取数据，第二步是获取的数据进行条件筛选。显然，相比后者，前者可以一步实现索引的查找Seek+filter，效率上更高。
    - [例子](https://yq.aliyun.com/articles/259696)
11. 查看是否使用索引：`explain select * from miui_version_info limit 10;`，就是加一个explain，输出的数据中，type=all即全表查询。
    

# 操作系统
## 进程与线程
1. 概念：进程是操作系统的概念，每当我们执行一个程序时，对于操作系统来讲就创建了一个进程,在这个过程中，伴随着资源的分配和释放。而线程是进程中的一部分，进程包含多个线程在运行。
2. [进程间的几种通信方式](https://zhuanlan.zhihu.com/p/37891272)说一下？
   - 管道（pipe）。半双工。只能用于具有亲缘关系的进程之间的通信。而命名管道（FIFO），允许无亲缘关系进程间的通信。若要数据流从父进程流向子进程，则关闭父进程的读端（fd[0]）与子进程的写端（fd[1]）；反之，则可以使数据流从子进程流向父进程。如下图6：
      <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/6-管道.png" width=72% height=72% />
   - 消息队列。在两个不相关进程间传递数据的一种方式，她独立于发送进程、接受进程而存在。由消息组成的链表，存放在内核中,并由消息队列标识符标识。有足够的权限的线程可以往队列中放置消息，有足够读权限的线程可以从队列中取走消息。每个消息都是一个记录，它由发送者赋予一个优先级。
      - 管道是文件，存放在磁盘上，访问速度慢
      - 匿名管道是跟随进程的，消息队列是跟随内核的，也就是说进程结束之后，匿名管道就死了，但是消息队列还会存在（除非显示调用函数销毁）
      - 管道是数据流式存取，消息队列是数据块式存取.
   - 共享内存。共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问，共享内存是最快的IPC方式。如信号量配合使用，来实现进程间的同步和通信
   - 信号量。信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它通常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源：
      - 系统首先要检测该资源的信号量
      - 若该资源的信号量值大于0，则进程可以使用该资源，此时，进程将该资源的信号量值减1；
      - 若该资源的信号量值为0，则进程进入休眠状态，直到信号量值大于0时进程被唤醒，访问该资源；
      - 当进程不再使用由一个信号量控制的共享资源时，该信号量值增加1，如果此时有进程处于休眠状态等待此信号量，则该进程会被唤醒。
   - 套接字。套接字通信更侧重于不同主机间的进程通信。
   - 信号。信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生

    >临界资源：只能被一个进程同时使用（不可以多个进程共享），要用到互斥。

3. 线程间的几种通信方式
   - 锁机制
   - 信号量机制
   - 信号机制
    > 线程间通信的主要目的是用于线程同步，所以线程没有象进程通信中用于数据交换的通信机制

# spring
[一切框架](https://www.ibm.com/developerworks/cn/java/j-lo-spring-principle/)，都是构建一个数据结构，然后根据这个数据结构设计他的生存环境，并让它在这个环境中按照一定的规律在不停的运动，在它们的不停运动中设计一系列与环境或者与其他个体完成信息交换

## spring三种注入方式
1. 属性注入方法，通过setXXX()方法注入Bean的属性值或者依赖对象
2. 构造函数注入方法，前提是：bean必须提供带参的构造函数
3. 工厂方法注入方法


## spring源码：
1. [来源](https://www.zybuluo.com/dugu9sword/note/382745)
2. 使用spring的好处：
   - 单例模式：有时候一些类是不需要多个对象的，例如数据库连接池的连接对象connection，只需要一个就好了。在spring里面，例如一些bean例如service层，完全可以交给spring容器管理。
   - 注入属性：之前如果不注入，就需要自己每次都new一个对象，现在Bean实现了 BeanFactoryAware 接口，则将容器的引用传入到 Bean 中去，这样，Bean 将获取对容器操作的权限，也就是可以根据bean的名字来获取容器中的对象。
3. 启动过程，Spring的启动过程实际上就是Ioc容器初始化以及载入Bean的过程
   1. 构建 BeanFactory，同时加载BeanDefinitions，以便把用户定义的数据结构转化为 Ioc 容器中的特定数据结构。接着，对已经构建的 BeanFactory 的配置做修改，例如获取用户定义的实现了 BeanPostProcessor 接口的子类，并执行把它们注册到 BeanFactory 对象中的 beanPostProcessors 变量中
   2. 注册可能感兴趣的事件
   3. 创建 Bean 实例对象
   4. 触发被监听的事件
   
## springmvc源码
1. [来源](https://dzone.com/articles/how-spring-mvc-really-works)
1. 客户端以URI形式向Web服务发送请求，直接请求到DispatcherServlet。

2. 为了找到与请求匹配的处理程序，DispatcherServlet浏览了HandlerMapping接口的已注册实现：
   1. `SimpleUrlHandlerMapping`可以通过配置文件找到
      
          /welcome.html=ticketController
          /show.html=ticketController
   2. `RequestMappingHandlerMapping`应用最多，它将请求映射到Controller类的注解了@RequestMapping的方法上。
3. 根据请求路径来匹配到HandlerMapping，获取请求对应的 Handler：这里首先通过请求获取一个处理链，这个mappedHandler就是对Handler的一个封装，里面包括Handler属性、以及拦截器。

        HandlerExecutionChain mappedHandler = getHandler(processedRequest); 
3. 然后开始匹配对应Handler（也就是我们平常说的 Controller 控制器）的适配器HandlerAdapter，接着由 HandlerAdapter调用handle方法来调用真正的Controller完成处理：处理器适配器的出现，是为了调用不同类型的Handler。在源码doDispatch中：
   
    	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
   getHandlerAdapter函数参数为handler，内部foreach对所有适配器进行遍历，每一个适配器都有support方法，输入handler是否是某一类型的实例，来判断是否支持某一类适配器。如SimpleControllerHandlerAdapter这个适配器return (handler instanceof Controller);说明适配器都是写好了的，需要新的bean加载方式就新建新的适配器。另外，HandlerAdapter这个接口有3个方法，主要是里面的handle方法。适配器的作用就是针对不同类型的handler，通过这个handle方法来执行控制器的具体逻辑。      
   
5. 处理器处理业务时的方式
   1. SimpleControllerHandlerAdapter这类适配器，返回内容。返回ModelAndView对象并且不自行呈现视图。Model 是返回的数据对象，View 是个逻辑上的 View。
   
	      public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		    return ((Controller) handler).handleRequest(request, response);
	      }
   2. SimpleServletHandlerAdapter适配器返回null。只是自己处理请求，渲染结果到响应对象
   
              public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{
                    ((Servlet) handler).service(request, response);
		    return null;
	      }
	      
6. 实际的Controller中的方法参数和返回值的处理，由RequestMappingHandlerAdapter来完成。
   1. Controller的方法通常不接受HttpServletRequest和HttpServletResponse参数，而是接收和返回许多不同类型的数据。而这，由RequestMappingHandlerAdapter来完成从HttpServletRequest中解析出参数。
   2. 返回值不需要返回ModelAndView，而是可以转化为json的Entity或者view的名字（具体HTML文件）
      - 当您从hello（）方法返回一个字符串时，ViewNameMethodReturnValueHandler会处理该值
      - 当你从login（）方法返回一个准备好的ModelAndView时，Spring使用了ModelAndViewMethodReturnValueHandler
      - 当在方法上使用@ResponseBody批注时，Spring会转换返回值并自动将其写入HTTP响应，因此出现了RestController:对每个方法的返回值都会直接转换为json,不需在方法前面加@ResponseBody,但是不能返回jsp,html页面，视图解析器无法解析jsp,html页面
6. 当前Spring已经处理了HTTP请求并收到了一个ModelAndView对象，需要呈现用户将在浏览器中看到的HTML页面。解析model将其渲染到view上， 这个model就是一个map,将里面的值一个一个赋值给request。然后发送。
8. 把 View 返回给请求者（浏览器）

>适配器模式在使用的时候，除了处理事件的handle方法，还需要一个support方法来，用以遍历循环，进行适配找到合适的适配器处理。


## spring注解
1. [Autowired](https://www.jianshu.com/p/83a79018580f)
   - 还可以注入List、Map、数组等相同类型bean。源码中doResolveDependency方法调用了resolveMultipleBeans方法：判断注入类型
   - 默认按类型装配
   - 如果我们想使用名称装配可以结合@Qualifier注解进行使用
   
		   @Autowired   
		   @Qualifier("userServiceImpl")   
		   public IUserService userService; 
2. @Resource
   - 默认按照名称进行装配
   - 名称可以通过name属性进行指定，也可以type指定。


## bean的循环引用问题
1. Spring 为了解决单例的循环依赖问题，使用了 三级缓存 ，递归调用时发现 Bean 还在创建中即为循环依赖
1. 基于Java的引用传递，当我们获取到对象的引用时，对象的field或则属性是可以延后设置
1. 三级缓存

		/** 一级缓存：用于存放完全初始化好的 bean **/
		private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);

		/** 二级缓存：存放原始的 bean 对象（尚未填充属性），用于解决循环依赖 */
		private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

		/** 三级级缓存：存放 bean 工厂对象，用于解决循环依赖 */
		private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
1. [解决过程](https://zhuanlan.zhihu.com/p/62382615)：
   1. A 创建过程中需要 B，于是 A 将自己放到三级缓里面 ，去实例化 B
   2. B 实例化的时候发现需要 A，于是 B 先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了！
   3. 然后把三级缓存里面的这个 A 放到二级缓存里面，并删除三级缓存里面的 A
   1. B 顺利初始化完毕，将自己放到一级缓存里面（此时B里面的A依然是创建中状态）
   1. 然后回来接着创建 A，此时 B 已经创建结束，直接从一级缓存里面拿到 B ，然后完成创建，并将自己放到一级缓存里面
   1. 总结：先让最底层对象完成初始化，通过三级缓存与二级缓存提前曝光创建中的 Bean，让其他 Bean 率先完成初始化。

## spring的aop与ioc
1. 传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建；依赖注入是当对象与其他对象发生依赖关系时，所依赖的对象由容器注入。
2. IOC，控制反转，不是什么技术，而是一种设计思想。Ioc意味着将你设计好的对象交给容器控制
3. DI，依赖注入.IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的。
4. 在面向对象编程中，关键的单元是对象，AOP的关键单元是切面。使用AOP技术，可以将一些系统性相关的编程工作，独立提取出来，独立实现，然后通过切面切入进系统。从而避免了在业务逻辑的代码中混入很多的系统相关的逻辑——比如权限管理，事物管理，日志记录等等。
5. spring实现的技术为： JDK提供的动态代理技术 和 CGLIB(动态字节码增强技术) 
   - **jdk是动态生成委托类的接口的实现类，cglib是动态生成委托类的子类**
   - JDK动态代理：实现InvocationHandler接口，通过Proxy类动态的创建了一个代理类。jdk代理只能代理接口，而cglib没有这个限制
   - CGLIB:使用ASM来代理普通类。但是不能代理final。final类不能被继承、没有子类、final类中修饰的方法默认是final。
       >ASM是一个java字节码操控框架，可以以二进制的形式修改已有类.
6. BeanFactory+FactoryBean
   1. FactoryBean,是Bean。在Bean产生以后，还可以自定义继续构造Bean。
      - getObject希望你返回需要注册到Spring容器中去的bean实体
      - getObjectType希望你返回你注册的这个Object的具体类型
      - isSingleton方法希望你返回这个bean是不是单例的，如果是，那么Spring容器全局将只保持一个该实例对象，否则每次getBean都将获取到一个新的该实例对象。
6. JDK代理

		public interface SimpleProxy { 

		    public void simpleMethod1(); 

		    public void simpleMethod2(); 

		}
		//使用时，实现InvocationHandler接口的invoke方法，在里面完成调用目的接口的增强，即调用前调用后
		public class JDKProxy implements InvocationHandler {    
    
		    private Object targetObject;//需要代理的目标对象    

		    public Object newProxy(Object targetObject) {//将目标对象传入进行代理    
			this.targetObject = targetObject;     
			return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(),    
				targetObject.getClass().getInterfaces(), this);//返回代理对象    
		    }    

		    public Object invoke(Object proxy, Method method, Object[] args)//invoke方法    
			    throws Throwable {    
			before();
			Object ret = null;      // 设置方法的返回值    
			ret  = method.invoke(targetObject, args);       //invoke调用需要代理的方法
			after();
			return ret;    
		    }    

		    private void before() {//方法执行前   
			System.out.println("方法执行前 !");    
		    }    
		    private void after() {//方法执行后    
			System.out.println("方法执行后");    
		    }    
		}  
		//生成的代理类。即返回这个代理类，用于调用
		public class $Proxy2 extends java.lang.reflect.Proxy implements SimpleProxy{ 
		    java.lang.reflect.Method m0; 
		    java.lang.reflect.Method m1; 
		    java.lang.reflect.Method m2; 
		    java.lang.reflect.Method m3; 
		    java.lang.reflect.Method m4; 

		    int hashCode(); 
		    boolean equals(java.lang.Object); 
		    java.lang.String toString(); 
		    //这个类的方法将调用 InvocationHandler 的 invoke 方法,完成对原调用的调用，并实现增强。
		    void simpleMethod1(); 
		    void simpleMethod2(); 
		}
## spring事务
1. [结论](http://blog.itpub.net/69900354/viewspace-2565243/)：
   - 如果是编译时异常不会自动回滚，如果是运行时异常，那会自动回滚！
   - 如果本类没有事务的方法调用有事务的方法，没有事务发生。下面实例经过测试（tmall_springboot项目中可以看到），在UserService里面有下面2个方法添加用户：
      
			@Transactional
			public void add(User user) {
				userDAO.save(user);
				int a = 10/0;//加事务后，模拟异常后回滚。
			}
			
			public void Add(User user){
				this.add(user);
			}
       在ForeRESTController的register方法调用`userService.Add(user);`,结果是虽然报错，但数据成功插入，也就是没有事务执行；但是调用`userService.add(user);`，结果是报错且数据没有插入，也就是出现事务。
    - 事务方法调用另一个事务方法，不会发生事务嵌套。[来源](https://www.jianshu.com/p/0da29e4f354a)
    - 上面2种情况解决方案：可以通过在方法内部获得代理对象的方式，通过代理对象调用同类的其他方法。如果配置@EnableAspectJAutoProxy(exposeProxy = true)，则将`this.add(user)`改为`((UserService) AopContext.currentProxy()).add(user);`另外，在tmall_springboot中SpringContextUtil这个工具类也可以直接使用，缓存与事务这种通过AOP来拿到代理对象的都会遇到这个问题。
			
2. Spring事务传播机制
   在当前含有事务方法内部调用其他的方法(无论该方法是否含有事务)，属于Spring事务传播机制的范畴.嵌套事务：嵌套是子事务套在父事务中执行，子事务是父事务的一部分，在进入子事务之前，父事务建立一个回滚点，叫save point，然后执行子事务，这个子事务的执行也算是父事务的一部分，然后子事务执行结束，父事务继续执行。重点就在于那个save point：
      - 如果子事务回滚，父事务会回滚到进入子事务前建立的save point，然后尝试其他的事务或者其他的业务逻辑，父事务之前的操作不会受到影响，更不会自动回滚。
      - 如果父事务回滚，父事务回滚，子事务也会跟着回滚！为什么呢，因为父事务结束之前，子事务是不会提交的
      - 提交：子事务先提交，父事务再提交。子事务是父事务的一部分
   
3. 一些概念   
	  - 基于接口代理(JDK代理):凡是类的方法非public修饰，或者用了static关键字修饰，那这些方法都不能被Spring AOP增强;
	  - 基于CGLib代理(子类代理):凡是类的方法使用了private、static、final修饰，那这些方法都不能被Spring AOP增强
	  - Spring IOC所管理的对象默认都是单例的，那么在使用的时候解决线程安全问题：ThreadLocal。
	  - 脏读：指一个事务读取了一个未提交事务的数据
	  - 不可重复读：事务A多次读取同一数据，事务B在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。
	  - 虚读(幻读)：在事务1中，查询User表id为1的是用户否存在，如果不存在则插入一条id为1的数据。在事务1查询结束后，事务2往User表中插入了一条id为1的数据。由于事务1查询到id为1的用户不存在，因此插入1条id为1的数据。但是由于事务2已经插入了1条id为1的数据，因此此时会报主键冲突，对于事务1 的业务来说是执行失败的，这里事务1 就是发生了幻读
	  - 不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表
  
4. [为了避免上面出现几种情况在标准SQL规范中定义了4个事务隔离级别](https://www.cnblogs.com/huanongying/p/7021555.html)：
	  - 未提交读取Read Uncommitted，允许脏读取但不允许更新丢失，如果一个事务已经开始写数据则另外一个数据则不允许同时进行写操作但允许其他事务读此行数据。事务隔离的最低级别。
	  - 提交读取Read Committed，允许不可重复读取但不允许脏读取。SELECT 命令不会返回尚未提交（Committed） 的数据，也不能返回脏数据
	  - 可重复读取Repeatable Read，禁止 不可重复读取和脏读取。读取数据事务将会禁止写事务（但允许读事务），写事务则禁 止任何其他事务。在此隔离级下，用SELECT 命令读取的数据在整个命令执行过程中不会被更改。此选项会影响系统的效能，非必要情况最好不用此隔离级。
	  - 串行Serializable，提供严格的事务隔离，它要求事务序列化执行，事务只能一个接着一个地执行，但不能并发执行。事务隔离的最高级别，事务之间完全隔离

5. [事务传播机制](https://blog.csdn.net/tiandao321/article/details/75388959)
    - PROPAGATION_REQUIRED，加入当前正要执行的事务不在另外一个事务里，那么就起一个新的事务
    - PROPAGATION_SUPPORTS，如果当前在事务中，即以事务的形式运行，如果当前不再一个事务中，那么就以非事务的形式运行
    - PROPAGATION_MANDATORY，必须在一个事务中运行。也就是说，他只能被一个父事务调用。否则，他就要抛出异常
    - PROPAGATION_REQUIRES_NEW 执行当前新建事务完成以后，上下文事务恢复再执行
    - PROPAGATION_NOT_SUPPORTED当前不支持事务
    - PROPAGATION_NEVER不能在事务中运行
    - PROPAGATION_NESTED
6. 注意：
   1. @Transactional 只能应用到 public 方法才有效。
      - 这是因为在使用 Spring AOP 代理时，Spring 在调用 TransactionInterceptor（在目标方法执行前后进行拦截）之前，DynamicAdvisedInterceptor（CglibAopProxy的内部类）的的 intercept方法或 JdkDynamicAopProxy 的 invoke 方法会间接调用 AbstractFallbackTransactionAttributeSource（Spring 通过这个类获取@Transactional 注解的事务属性配置属性信息）的 computeTransactionAttribute 方法。
      
		      protected TransactionAttribute computeTransactionAttribute(Method method,Class<?> targetClass) {
			// Don't allow no-public methods as required.
			if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
			return null;
		      }
   2. 为解决这自调用以及public两个问题，使用 AspectJ 取代 Spring AOP 代理。有一种是[自注入](https://maskwang520.github.io/2018/04/16/Transaction%E5%BF%85%E7%9F%A5%E5%BF%85%E4%BC%9A/)；另一种就是Aspect。

# 集合框架
[来源](https://segmentfault.com/a/1190000014240704)。

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/collect.jpeg" width=80% height=80% />

## list
1. 特点：有序，存储顺序与取出顺序一致。

### ArrayList
1. 结构图7：

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/7ArrayList.png" width=80% height=80% />
2. 实现了List接口，可以List基本操作；实现了RandomAccess接口，作为一个标识，表示可以快速随机访问元素。
1. 底层数组，线程不安全。将数据存在Object[] elementData中。
2. `add(E e)`方法：
   - 检查一下数组的容量是否足够：size+1即当前增加元素所需要的容量minCapacity，如果是增加的第一个元素，设置minCapacity为默认10。
   - 所需容量minCapacity与当前容量elementData.length比较，如果数组长度不够，扩容1.5倍。如果还小，扩充为minCapacity。返回`Arrays.copyOf(elementData, newCapacity);`而Arrays.copyOf底层是System.arraycopy。
3. `remove(int index)`方法，跟`add(int index,E element)`一样，底层元素移动都是System.arraycopy实现。删除元素时不会减少容量，若减少容量则调用trimToSize()

### 线程安全的ArrayList
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

### LinkedList
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

## Set
1. 特点：元素不可重复。

### HashSet
1. 底层哈希表，即一个数组，其中元素为链表。

### TreeSet
1. 底层红黑树。

### LinkedHashSet
1. 底层哈希表与链表组成。



## Map
1. Collections.unmodifiableMap(cache);返回一个不可修改的视图
1. [红黑树](http://www.sohu.com/a/201923614_466939)特性：
	- 根节点总是黑色的；
	- 每个叶子节点都是红色
	- 如果节点是红色的，则它的父子节点必须是黑色；
	- [从根节点到叶节点或空子节点的每条路径，必须包含相同数目的黑色节点](https://www.cnblogs.com/CarpenterLee/p/5503882.html)
### HashMap
1. 结构图

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/9Hashmap.png" width=80% height=80% />
2. 为何红黑树？我的理解：
   1. 不光是为了存储，更多的是为了建立索引树，便于搜索。
   2. 常见的二叉搜索树，一般容易极端情况退化成链表；而平衡树好，但是每次插入容易旋转啥的，很费时；现在红黑树调整可以分为两类：一类是颜色调整，即改变某个节点的颜色；另一类是结构调整，集改变检索树的结构关系,不完全L树的平衡条件的，即每个节点的左子树和右子树的高度最多差1的二叉查找树。红黑是用非严格的平衡来换取增删节点时候旋转次数的降低。O(㏒(n))
2. 底层数组+链表(拉链法)，即散列表
3. Hashmap的扩容需要满足两个条件：当前数据存储的数量（即size()）大小必须大于等于阈值；当前加入的数据是否发生了hash冲突
3. `put(K key,V value)`方法。
   - 以key计算哈希值,这里面的key.hashCode()是Object方法，就是任何对象都有一个哈希码hashCode。接着计算`hashCode^hashCode>>>16`，才是最终的哈希值。下面解释这样计算的原因。
     
		    static final int hash(Object key) {
		        int h;
		        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
		    }
    - 接着，执行put操作。散列表容量n,而哈希值范围很广，为了将哈希值缩小范围正好对应在[0,n-1]上做数组下标，采用&运算，这样不管哈希值多大，高于n的二进制位全部为0，计算的结果就在[0,n-1]。现在解释`hashCode^hashCode>>>16`，如果很多哈希码高位差异大而低位相同，直接计算`(n - 1) & hash`的值相同的情况会增加，导致碰撞概率增大。而这样计算后，此时的低位实际上是高位与低位的结合，增加了随机性。
     
			final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
                    //table就是存放实体Node的数组，p就是实体即一对KV
			        Node<K,V>[] tab; Node<K,V> p; int n, i;
                    //散列表为空，初始化散列表,resize方法在初始化以及元素个数大于capacity*load factor时调用。
			        if ((tab = table) == null || (n = tab.length) == 0)
			            n = (tab = resize()).length;
                    //没有冲突，直接添加
			        if ((p = tab[i = (n - 1) & hash]) == null)
			            tab[i] = newNode(hash, key, value, null);
			        else {
                        //出现冲突，即要插入的元素，在散列表中的位置有值，记录
                        //如果插入的hash与key都相同，说明是覆盖
				//如果插入的元素是红黑树结构，调用树的插入
				//如果是链表结构(这里就是拉链法的应用)，在该节点处开始找该节点所在的链表，如果链表中存在相同的hash与key，就是覆盖，记录。如果不存在，在链表后加一个节点，同时，如果插入后发现临界值大于TREEIFY_THRESHOLD，将该链表转成红黑树。这里注意的是，是该哈希值下的链表元素个数如果大于8，才转树。
				//最后，如果有记录，进行覆盖操作。
                        
                    }
            }

## ConCurrentHashMap
1. 特性：
   - JDK1.8底层是链表+数组+红黑树
   - 支持高并发的访问和更新，线程安全
   - 检索操作不用加锁，get方法非阻塞
   - key和value都不允许为null
    >Hashtable是在每个方法上都加上了Synchronized完成同步，效率低下。1.8的ConcurrentHashMap通过在部分加锁和利用CAS算法来实现同步。1.7是采用分段锁。
2. [在原先HashMap的基础上采取的方案](https://www.jianshu.com/p/c0642afe03e0)：
   1. put
   
                   //无限循环插入
		   for (Node<K,V>[] tab = table;;) {
			    Node<K,V> f; int n, i, fh;
			    //如果table没有就先初始化，进入下一次循环插入
			    if (tab == null || (n = tab.length) == 0)
				tab = initTable();
			    //如果桶中当前位置没有元素，直接用cas原则插入，无需加锁。
			    //如果CAS成功，说明Node节点已经插入，随后addCount(1L, binCount)方法会检查当前容量是否需要进行扩容。
                            //如果CAS失败，说明有其它线程提前插入了节点，自旋重新尝试在这个位置插入节点。
			    else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
				if (casTabAt(tab, i, null,
					     new Node<K,V>(hash, key, value, null)))
				    break;                   // no lock when adding to empty bin
			    }
			    //说明当前f是ForwardingNode节点，意味有其它线程正在扩容，则一起进行扩容操作。
			    else if ((fh = f.hash) == MOVED)
                		tab = helpTransfer(tab, f);
			    //如果冲突，即table处已经有值，此时值为f，链表中的第一个节点，也就是从这个节点开始拉链,在节点f上进行同步
			    else {
				V oldVal = null;
				synchronized (f) {
				    if (tabAt(tab, i) == f) {
					if (fh >= 0) {
					    binCount = 1;
					    for (Node<K,V> e = f;; ++binCount) {
						K ek;
						if (e.hash == hash &&
						    ((ek = e.key) == key ||
						     (ek != null && key.equals(ek)))) {
						    oldVal = e.val;
						    if (!onlyIfAbsent)
							e.val = value;
						    break;
						}
						Node<K,V> pred = e;
						if ((e = e.next) == null) {
						    pred.next = new Node<K,V>(hash, key,
									      value, null);
						    break;
						}
					    }
					}
					// 如果f是树节点，采用红黑树处理
					else if (f instanceof TreeBin) {
					    Node<K,V> p;
					    binCount = 2;
					    if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
									   value)) != null) {
						oldVal = p.val;
						if (!onlyIfAbsent)
						    p.val = value;
					    }
					}
				    }
				}
				if (binCount != 0) {
				    if (binCount >= TREEIFY_THRESHOLD)
					treeifyBin(tab, i);
				    if (oldVal != null)
					return oldVal;
				    break;
				}
			    }//end of else
			    ........
	           }
    
    
    
# jvm
## 内存区域
1. [运行时数据区](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483910&idx=1&sn=246f39051a85fc312577499691fba89f&chksm=fd985467caefdd71f9a7c275952be34484b14f9e092723c19bd4ef557c324169ed084f868bdb#rd)如下图所示：

   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/1jvm.jpg" width=80% height=80% />
2. [HotSpot虚拟机加载对象](https://blog.csdn.net/yangshangwei/article/details/81252722)：
  - 虚拟机遇到一条new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程。
  - 在类加载检查通过后，接下来虚拟机将为新生对象分配内存。根据堆中内存是否绝对规整，选择空闲列表或者指针碰撞的方式来分配内存。而Java堆是否规整是由所采用的垃圾收集器是否带有压缩整理功能决定。
   	>Java堆内存绝对规整，指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”（ Bump the Pointer）。例子：使用的垃圾收集器是Serial、ParNew。
   
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

		Object obj = new Object();
		SoftReference<Object> sf = new SoftReference<Object>(obj);
		obj = null;
		sf.get();//有时候会返回null
    >可用于实现内存敏感的高速缓存，在内存足够的情况下直接通过软引用取值，无需从繁忙的真实来源查询数据，提升速度；当内存不足时，自动删除这部分缓存数据，从真正的来源查询这些数据。

3. 弱引用（WeakReference）:在垃圾回收器线程扫描它 所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存.

		Object obj = new Object();
		WeakReference<Object> wf = new WeakReference<Object>(obj);
		obj = null;
		wf.get();//有时候会返回null
		wf.isEnQueued();//返回是否被标记为即将回收的垃圾
    >弱引用可用于解决内存泄漏的问题。另外，弱引用主要用于监控对象是否已经被垃圾回收器标记为即将回收的垃圾，可以通过弱引用的isEnQueued方法返回对象是否被垃圾回收器标记。

4. 虚引用（PhantomReference）：垃圾回收时回收，无法通过引用取到对象值。虚引用是每次垃圾回收的时候都会被回收，通过虚引用的get方法永远获取到的数据为null，因此也被成为幽灵引用。

		Object obj = new Object();
		PhantomReference<Object> pf = new PhantomReference<Object>(obj);
		obj=null;
		pf.get();//永远返回null
		pf.isEnQueued();//返回是否从内存中已经删除
    >虚引用主要用于检测对象是否已经从内存中删除。

5. 注意：在程序设计中一般很少使用弱引用与虚引用，使用软引用的情况较多，这是因为软引用可以加速JVM对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生


### 内存回收
1. 如何判断对象是否死亡
   - 引用计数法，给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使用的。缺点：很难解决对象之间相互循环引用的问题，下面的例子就是：
   
			class MyObject{
			    public Object object = null;
			}
		    public class Main {
			    public static void main(String[] args) {
			        MyObject object1 = new MyObject();
			        MyObject object2 = new MyObject();
			 
			        object1.object = object2;
			        object2.object = object1;
			 
			        object1 = null;//设为null就是想回收
			        object2 = null;//但是
			    }
			}
   - 可达性分析算法，通过一系列称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径称为“引用链”，当一个对象到GC Roots没有任何的引用链相连时(从GC Roots到这个对象不可达)时，证明此对象不可用。对于上面的两个对象，虽然彼此引用，但是GC Roots不可达，也会回收。具体GC Roots有：
     - 虚拟机栈中引用的对象，程序中new一个对象，堆上将开辟一块空间，同时将空间的地址作为引用保存到虚拟机栈中，如果对象生命周期结束了，那么引用就会从虚拟机栈中出栈，因此如果在虚拟机栈中有引用，就说明这个对象还是有用的
     - 方法区中静态属性引用的对象
     - 方法区中常量引用的对象
     - 本地方法栈中(Native方法)引用的对象

### 垃圾收集算法
1. 垃圾回收动作何时执行：
   1. 当年轻代内存满时，会引发一次普通GC，该GC仅回收年轻代。
   2. 当年老代满时会引发Full GC，Full GC将会同时回收年轻代、年老代
   3. 当永久代满时也会引发Full GC，会导致Class、Method元信息的卸载。但是永久代不属于堆内存，由虚拟机直接分配
2. 触发OutOfMemoryException的条件
   1. JVM98%的时间都花费在内存回收
   2. 每次回收的内存小于2%
1. 标记-清除算法Mark-Sweep。如下图。缺点是效率低以及标记清除后会产生大量的不连续内存碎片。
   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/4jvm垃圾回收标记清除.png" width=50% height=50% />
2. 复制算法：将内存划分为大小相等的两块，每次只使用其中的一块。当这块内存用完了，就将还存活的对象复制到另一块内存上，然后把已使用过的内存空间一次清理掉。不会产生内存碎片，缺点是内存缩小一半。目前流行。
   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/5jvm复制算法.png" width=50% height=50% />
3. 标记-整理算法Mark-Compact：让所有存活对象都向一端移动，然后直接清理掉端边界以外的所有内存
   <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/6jvm标记整理.png" width=50% height=50% />
4. 分代收集算法Generational Collection：根据对象的存活周期的不同将内存划分为几块，一般就分为新生代和老年代，根据各个年代的特点采用不同的收集算法。新生代（少量存活，容易产生碎片）用复制算法，老年代（对象存活率高）“标记-清理”算法。
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

3. Parallel Scavenge收集器，使用复制算法、并行。
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

### 类加载
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
		
## 异常
1. [算术异常类](https://www.cnblogs.com/cvst/p/5822373.html)：ArithmeticExecption。除以0等。
2. 空指针异常类：NullPointerException
3. 类型强制转换异常：ClassCastException
4. 数组下标越界异常：ArrayIndexOutOfBoundsException
5. 文件未找到异常：FileNotFoundException
6. 字符串转换为数字异常：NumberFormatException
7. java.lang.OutOfMemoryError：内存不足错误。
8. java.lang.StackOverflowError：堆栈溢出错误。当一个应用递归调用的层次太深而导致堆栈溢出时抛出该错误。
9. java.lang.UnsupportedClassVersionError。JDK版本不同异常。换一个打包版本就行
## io

### NIO
1. Non-blocking，与IO的区别：
   - IO是面向流的，NIO是面向缓冲区的。
   - IO流是阻塞的，NIO流是不阻塞的。NIO中，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情,真正的IO操作是内核线程。而IO则是：当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了
   - [NIO的核心是selector中的事件通知](http://www.fanyeong.com/2016/09/19/%E5%AF%B9nio%E7%9A%84%E4%B8%80%E7%82%B9%E7%90%86%E8%A7%A3/).
     - 在BIO中,每来一个连接请求，服务器都将分配一个线程来处理这个连接，这个线程专门用来处理读写等事件，直到连接关闭。阻塞并不是IO读写，而是在IO等待。
     - 而NIO中每次来连接都丢给专门处理连接的线程来将请求注册到Selectors，即一个线程用于接受请求；一个线程池用于（或者自己新建一个线程）处理请求——完成读写数据的操作
   - Selectors（选择器）。选择器用于使用单个线程处理多个通道。线程之间的切换对于操作系统来说是昂贵的。 选择器是一个可以监视多个通道的对象，使用Selector的话，我们必须把Channel注册到Selector上，然后就可以调用Selector的select()方法。这个方法会进入阻塞，直到有一个channel的状态符合条件。当方法返回后，线程可以处理这些事件
#### 原理
1. [来源1](https://tech.meituan.com/2016/11/04/nio.html)
2. [来源2](https://segmentfault.com/a/1190000003063859) 
2. NIO的主要事件有几个：读就绪、写就绪、有新连接到来
3. 首先需要注册当这几个事件到来的时候所对应的处理器,然后在合适的时机告诉事件选择器：我对这个事件感兴趣
4. 用一个死循环选择就绪的事件：
   2. 当用户进程调用了selector.select，没有事件到来，那么整个进程会被block。而同时，kernel会“监视”所有selector负责的socket。
   1. 如果有事件到来，即任何一个socket中的数据准备好了，将执行系统调用（Linux 2.6之前是select、poll，2.6之后是epoll）。新事件到来的时候，会在selector上注册标记位，标示可读、可写或者有连接到来。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程
    >select是阻塞的，无论是通过操作系统的通知（epoll）还是不停的轮询(select，poll)，这个函数是阻塞的。所以你可以放心大胆地在一个while(true)里面调用这个函数而不用担心CPU空转。
5. select，poll，epoll都是IO多路复用的机制，但本质上都是同步I/O，而异步I/O则无需自己负责进行读写.
   1. `int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);`底层数组
      1. select监视writefds、readfds、和exceptfds。
      2. 调用后select函数会阻塞，直到有描述副就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。
      3. 缺点在于单个进程能够监视的文件描述符的数量存在最大限制,Linux上一般为1024
   2. poll,和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。select和poll都需要在返回后，通过遍历文件描述符来获取已经就绪的socket。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。底层链表。
   3. epoll。底层红黑树
      1. `int epoll_create(int size);`,创建一个epoll的句柄，参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议
      2. `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；`对指定描述符fd执行op操作，即红黑树进行增删改
      3. `int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);`返回需要处理的事件数目，如返回0表示已超时。
   4. epoll对上面2个的缺点处理：
      1. 监视的描述符数量不受限制，就是树的大小。
      1. 遍历：epoll事先通过epoll_ctl()来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait() 时便得到通知。此处去掉了遍历文件描述符，而是通过监听回调的的机制。
#### Channel
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


#### Selector（选择器）
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


## redis
1. redis快，基于内存，单线程，基于resp协议。可以将mysql中的数据，根据select语句做管道输入，将数据转入redis。
2. 数据类型：
   - String
     - `SET name "runoob"`
     - `GET name`
   - Hash,适合存储对象
     - `HMSET myhash field1 "Hello" field2 "World"`
     - `HGET myhash field1`
   - List,增删快
     - `lpush runoob redis`
     - `lpush runoob mongodb`
     - `lrange runoob 0 10`    //取值
   - Set,添加、删除,查找的复杂度都是O(1),为集合提供了求交集、并集、差集等操作。内部使用Hash结构，所有的value都指向同一个内部值。
     - `sadd runoob mongodb`   //添加
     - `smembers runoob`    //取值
   - zset(sorted set：有序集合)，不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。
     - `zadd runoob 0 redis`
     - `ZRANGEBYSCORE runoob 0 1000`
3. [zset底层](https://lijh.dev/2019/03/19/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Redis-Zset%E5%8E%9F%E7%90%86/)：Hash+[跳表](https://juejin.im/post/57fa935b0e3dd90057c50fbc)
   - Hash作用：保障元素value的唯一性,可以通过元素value找到相应的score权重值。
   - 跳跃列表的目的在于根据score给元素value排序，即输出为有序集合。

    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/skiplist.png" width=50% height=50% />
4. 跳表特点
   1. 在有序链表的基础上发展起来，最初想模仿二分法查找的方式来使查找时间复杂度为O(log n)。但是新插入一个节点之后，就会打乱原先的链表指针对应关系。
   2. 为了避免这个问题，为每个节点随机出一个层数，插入操作只需要修改插入节点前后的指针，而不需要对很多节点都进行调整。这样新插入一个节点不会影响其它节点
   3. 不选择树：
      - 它们的内存不是很大。
      - 效率几乎一样
      - 更容易实现，调试

### redis做消息队列
1. 通过2个命令实现pub/sub模式的消息通知
   - `publish 主题名称 消息内容`,向指定主题发送一条消息
   - `subscribe 主题名称`,订阅某一主题。
2. key失效的时候会发送一些通知，可以通过订阅某一个主题，接受key失效的消息通知（此通知是redis内部事件处理机制）
   - 首先开启事件通知，修改redis启动的配置文件：安装目录下的redis.windows.conf，查找到`notify-keyspace-events Ex`，取消其注释，然后重启redis服务器。
   - 接着，需要订阅的主题名称：`__keyevent@dbindex__:expired`。例子：存储到的redis服务器是0号数据库，则订阅命令为` subscribe __keyevent@0__:expired`，然后测试`set keyName "xu"`,`expire keyName 5`设置其5秒后过期，`ttl keyName`查看其剩余时间。
    >不过很遗憾，我修改配置文件并没有生效，而是运行命令`config set notify-keyspace-events Ex`才可以



### redis为何那么快
1. redis是基于内存的，一般都是简单的存取操作，内存的读写速度非常快
2. redis是单线程的，省去了很多上下文切换线程的时间，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗
    >官方FAQ表示：因为Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了
3. redis使用非阻塞I/O多路复用技术，可以处理并发的连接
   - 多路，指的是多个socket连接
   - 复用，指的是复用一个线程

    [Redis-client 在操作的时候](https://zhuanlan.zhihu.com/p/50392209)，会产生具有不同事件类型的 Socket。在服务端，有一段 I/O 多路复用程序，将其置入队列之中。然后，文件事件分派器，依次去队列中取，转发到不同的事件处理器中。
    >Redis服务器与客户端通过RESP（REdis Serialization Protocol）协议通信


### 过期数据的删除对 Redis 性能影响

1. [定时删除](https://zhuanlan.zhihu.com/p/57089960)：在设置键过期expire时间的同时，创建一个定时器 Timer，让定时器在键过期时间来临时立即执行对过期键的删除。
   - 对内存友好，对 CPU 不友好。如果过期删除的键比较多的时候，删除键这一行为会占用相当一部分 CPU 性能，会对 Redis 的吞吐量造成一定影响
2. 惰性删除：键过期后不管，每次读取该键时，判断该键是否过期，如果过期删除该键返回空。
   - 对 CPU 友好，内存不友好。如果很多键过期了，但在将来很长一段时间内没有很多客户端访问该键导致过期键不会被删除，占用大量内存空间。
3. 定期删除：每隔一段时间对数据库中的过期键进行一次检查。
   - Redis 会将每一个设置了 expire 的键存储在一个独立的字典中，以后会定时遍历这个字典来删除过期的 key。除了定时遍历外，它还会使用惰性删除策略来删除过期的 key。
   - Redis 默认每秒进行十次过期扫描，过期扫描不会扫描所有过期字典中的 key，而是采用了一种简单的贪心策略:从过期字典中随机选择 20 个 key；删除这 20 个 key 中已过期的 key；如果过期 key 比例超过 1/4，那就重复步骤一
   - 同时，为了保证在过期扫描期间不会出现过度循环，导致线程卡死，算法还增加了扫描时间上限，默认不会超过 25ms
4. 定期删除+惰性删除。如果定期删除没删除掉 Key。并且你也没及时去请求 Key，也就是说惰性删除也没生效。这样，Redis 的内存会越来越高。那么就应该采用内存淘汰机制。在redis.conf文件中，`# maxmemory-policy volatile-lru`用于配内存淘汰策略.其中比较推荐的一种是：allkeys-lru，当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 Key

### 如何应对缓存穿透和缓存雪崩问题
1. 缓存穿透，即黑客故意去请求缓存中不存在的数据，导致所有的请求都怼到数据库上，从而数据库连接异常。
   - 提供一个能迅速判断请求是否有效的拦截机制，比如，利用布隆过滤器，内部维护一系列合法有效的 Key。迅速判断出，请求所携带的 Key 是否合法有效。如果不合法，则直接返回。
2. 缓存雪崩，即缓存同一时间大面积的失效，这个时候又来了一波请求，结果请求都怼到数据库上，从而导致数据库连接异常。
   - 给缓存的失效时间，加上一个随机值，避免集体失效。
   - 双缓存。我们有两个缓存，缓存 A 和缓存 B。缓存 A 的失效时间为20分钟，缓存 B 不设失效时间。自己做缓存预热操作。例如：从缓存 A 读数据库，有则直接返回；A 没有数据，直接从 B 读数据，直接返回，并且异步启动一个更新线程，更新线程同时更新缓存 A 和缓存 B。
   
### 分布式锁
1. 分布式系统中，由于分布式系统的分布性，即多线程和多进程并且分布在不同机器中，synchronized和lock这两种锁将失去原有锁的效果，需要我们自己实现分布式锁。
2. 条件：
   - 互斥性：在任意一个时刻，只有一个客户端持有锁
   - 无死锁：即便持有锁的客户端崩溃或者其他意外事件，锁仍然可以被获取。
   - 容错：只要大部分Redis节点都活着，客户端就可以获取和释放锁
   
3. 基于redis的分布式锁实现
   - SETNX
      - [多个进程执行以下Redis命令](https://juejin.im/entry/5a502ac2518825732b19a595)`SETNX lock.id <current Unix time + lock timeout + 1>`:
	    - 返回1，说明该进程获得锁，SETNX将键 lock.id 的值设置为锁的超时时间，当前时间 +加上锁的有效时间。
	    - 返回0，说明其他进程已经获得了锁，进程不能进入临界区。进程可以在一个循环中不断地尝试 SETNX 操作，以获得锁。
		    
      存在问题：死锁。某个线程获取了锁之后，断开了与Redis 的连接，锁没有及时释放，竞争该锁的其他线程都会hung，产生死锁的情况。于是才设置的时间。但有另外的问题，离线超时后未释放，其他进程中的线程怎么获取，直接del肯定不行，容易重复del.
   - GETSET
     - A已经首先获得了锁 lock.id，然后A断线。B,C都在等待竞争该锁；
     - B,C通过GET读取lock.id的值，比较当前时间和键 lock.id 的值来判断是否超时，发现超时；
     - B检测到锁已超时，即当前的时间大于键 lock.id 的值，B会执行`GETSET lock.id <current Unix timestamp + lock timeout + 1>`设置时间戳，通过比较键 lock.id 的旧值是否小于当前时间，判断进程是否已获得锁；
     - B发现GETSET返回的值小于当前时间，则执行 DEL lock.id命令，并执行 SETNX lock.id 命令，并返回1，B获得锁；
     - C执行GETSET得到的时间大于当前时间，则继续等待。
      
	 解决问题：BC同时发现超时,即BC均GET到key的时间，均大于这个时间，同时去执行GETSET，试图比较返回值与自己GET到的值，如果返回值比自己GET到的值大，说明别的线程已修改，相等，则说明还没有人拿到锁。
     >`SETNX key value`若给定的 key 已经存在，则 SETNX 不做任何动作。SET if Not eXists。设置成功，返回 1 。设置失败，返回 0 。   
     >`GETSET key value`:将给定 key 的值设为 value ，并返回 key 的旧值(old value)。

### redis集群：有3个主节点，每个主节点都有一个从节点作为备用。节点间交换通过Gossip协议
1. 原理：集群里面的槽总共有16384个，根据节点的个数进行平均分配。分配完后各个节点向其他节点发送数据，这样就知道各个节点负责哪些槽。然后就可以用了。
2. 存取：随机在一个节点执行set操作，该节点使用crc16算法对key计算得哈希值，然后对16384求余，判断余数在哪个节点，然后直接自动跳转到这个对应的节点上进行存取操作。
3. 投票：当一台主节点掉线时，集群中其他的主节点将会通过投票的形式在掉线主结点的从结点中选举一台从节点来代替原先的主节点，并更改剩余从节点slave的主节点为新的主节点。即所有master参与,如果半数以上master节点与master节点通信超时。
4. fail：
   - 如果集群任意master挂掉,且当前master没有slave.集群进入fail状态
   - 如果集群超过半数以上master挂掉，无论是否有slave，集群进入fail状态.
5. 哨兵模式。哨兵是一个独立的进程，哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例：
   - 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
   - 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。

### 一致性哈希算法
1. [来源](https://blog.csdn.net/bntX2jSQfEHy7/article/details/79549368):本来是通过`hash(a.png) % 4 = 2`来找到对象存放的服务器，但当分布式服务器中有的坏掉或者新增，则不再是对4进行取模导致结果跟之前不一样。由于哈希数值上是0-2^32，因此一致性Hash算法是对2^32取模。由2^32个点组成的圆环称为Hash环。接着确定各个服务器在圆环的位置，最后，将数据key使用相同的函数Hash计算出哈希值，并确定此数据在环上的位置，从此位置沿环顺时针“行走”，第一台遇到的服务器就是其应该定位到的服务器！
2. 一致性Hash算法的容错性和可扩展性：
   - 现假设Node C不幸宕机，可以看到此时对象A、B、D不会受到影响，只有C对象被重定位到Node D。一般的，在一致性Hash算法中，如果一台服务器不可用，则受影响的数据仅仅是此服务器到其环空间中前一台服务器（即沿着逆时针方向行走遇到的第一台服务器）之间数据，其它不会受到影响
   - 如果在系统中增加一台服务器Node X
   
      <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/pp.png" width=50% height=50% />
 
3. 问题：一致性Hash算法在服务节点太少时，容易因为节点分部不均匀而造成数据倾斜（被缓存的对象大部分集中缓存在某一台服务器上）问题，为了解决这种数据倾斜问题，一致性Hash算法引入了虚拟节点机制，即对每一个服务节点计算多个哈希，每个计算结果位置都放置一个此服务节点，称为虚拟节点。具体做法可以在服务器IP或主机名的后面增加编号来实现。例如上面的情况，可以为每台服务器计算三个虚拟节点，于是可以分别计算 “Node A#1”、“Node A#2”、“Node A#3”、“Node B#1”、“Node B#2”、“Node B#3”的哈希值，于是形成六个虚拟节点：

    <img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/ppp.png" width=50% height=50% />
4. 然而redis并没有使用一致性哈希，文档自己说的。[原因的话](https://zackku.com/redis-cluster/):一致性哈希算法对于数据分布、节点位置的控制并不是很友好
   1. Redis Cluster包含了16384个哈希槽，每个Key通过计算后都会落在具体一个槽位上，而这个槽位是属于哪个存储节点的，则由用户自己定义分配。例如机器硬盘小的，可以分配少一点槽位，硬盘大的可以分配多一点。如果节点硬盘都差不多则可以平均分配。所以哈希槽这种概念很好地解决了一致性哈希的弊端。
   2. 在容错性和扩展性上(添加或故障删除节点)，都是对受影响的数据进行转移，但Redis集群是不会自动进行的，而是需要人工配置的。
   
### 主从复制
https://juejin.im/post/5b625b9be51d4519956759d0


### 持久化

### 哨兵机制
1. [redis使用哨兵机制来实现高可用](https://www.codedump.info/post/20190409-redis-sentinel/)
   - redis使用一组哨兵（sentinel）节点来监控主从redis服务的可用性
   - 一旦发现redis主节点失效，将选举出一个**哨兵节点**作为领导者
   - 哨兵领导者再从剩余的从redis节点中选出一个redis节点作为新的主redis节点对外服务
2. 如何对redis数据节点进行监控：每隔10秒，每个哨兵节点都会向主、从redis数据节点发送info命令，获取新的拓扑结构信息（本节点角色：主或从；主从节点的地址、端口信息）
3. 如何确定一个redis数据节点失效：每隔1秒，每个哨兵节点向主、从数据节点以及其他sentinel节点发送ping命令做心跳探测，这个心跳探测是后续主观判断数据节点下线的依据;当一个哨兵节点认为主节点主观下线时，该哨兵节点需要通过”sentinel is-master-down-by addr”命令向其他哨兵节点咨询该主节点是否下线了，如果有超过半数的哨兵节点都回答了下线，此时认为主节点“客观下线”。
4. 当主节点客观下线时，需要选举出一个哨兵节点做为哨兵领导者，以完成后续选出新的主节点的工作:
   - 每个哨兵节点通过向其他哨兵节点发送”sentinel is-master-down-by addr”命令来申请成为哨兵领导者
   - 而每个哨兵节点在收到一个”sentinel is-master-down-by addr”命令时，只允许给第一个节点投票，其他节点的该命令都会被拒绝
   - 如果一个哨兵节点收到了半数以上的同意票，则成为哨兵领导者
5. 选出新的主节点:过滤掉“不健康”的数据节点;选择复制偏移量最大的从节点

### 题：
1. [TCP协议-如何保证传输可靠性](https://blog.csdn.net/liuchenxia8/article/details/80428157)
2. [跳表](https://www.cnblogs.com/seniusen/p/9870398.html)
3. [非递归且不用额外空间（不用栈），如何遍历二叉树](https://blog.csdn.net/Darfie/article/details/53302104)
4. 操作系统的内存管理机制，待解决
5. [快排](https://www.cnblogs.com/MOBIN/p/4681369.html)
   1. [时间复杂度](https://blog.csdn.net/qq_20746945/article/details/89378662)
6. [其他排序](https://www.cnblogs.com/zhaoshuai1215/p/3448154.html)：
   - 插入排序。当待排序的数据基本有序时，插入排序的效率比较高，只需要进行很少的数据移动。
   - 选择排序。遍历数组，遍历到i时，a0,a1...ai-1是已经排好序的，然后从i到n选择出最小的，记录下位置，如果不是第i个，则和第i个元素交换。
   - 堆排序。
6. [OOM](https://www.jianshu.com/p/2fdee831ed03)：
   - java.lang.OutOfMemoryError:Java heap space：
     - 堆大小设置小了。解决办法：在IDEA的Run/Debug Configturations的VM中填写`-Xmx2014m`
     - 内存泄漏：由于GC无法识别一些已经不再使用的对象，而这些未使用的对象一直留在堆空间中，这种堆积最终会导致OOM。
   - java.lang.OutOfMemoryError:GC overhead limit exceeded。默认情况下，当应用程序花费超过98%的时间用来做GC并且回收了不到2%的堆内存时，会抛出该错误。具体的表现就是你的应用几乎耗尽所有可用内存，并且GC多次均未能清理干净。
   - java.lang.OutOfMemoryError:Permgen space。表明持久代所在区域的内存已被耗尽。不过JDK8已经完全移除持久代空间：
     - 如果只是空间小，`-XX:MaxPermSize=512m`
     - 如果是Redeploy时的OutOfMemoryError，
   - java.lang.OutOfMemoryError:Metaspace。太多的类或太大的类加载到元空间。
     - 如果空间不够，`-XX：MaxMetaspaceSize = 512m`
   - java.lang.OutOfMemoryError:Unable to create new native thread.意味着Java应用程序已达到其可以启动线程数量的极限了。
7. [堆](https://www.jianshu.com/p/6b526aa481b1)
8. 局部变量使用前需要显式地赋值，否则编译通过不了，为什么这么设计：
   - 成员变量赋值与取值的先后顺序具有不确定性，因此交给JVM去初始化
   - 局部变量，其赋值与取值的访问顺序是确定的。这样做为了尽最大可能减少使用者犯错。
9. [二叉搜索树](https://blog.csdn.net/Tommy1295/article/details/82733768)
   - 删除：
     - 节点只有左或者右子节点：
     - 节点有左右子节点：在右子树中查找最小值，将值赋给要删除的节点。然后递归删除最小值。也可以找左子树最大值处理
10. [DJ算法](https://www.cnblogs.com/biyeymyhjob/archive/2012/07/31/2615833.html)
11. Java面向对象
    - 封装：把描述一个对象的属性和行为封装成一个类，把具体的业务逻辑功能实现封装成一个方法，其次封装的意义还有效的保护属性通过访问修饰符私有化属性(成员变量),公有化方法
    - 继承：实现代码的复用，所有的子类所共有的行为和属性抽取为一个父类，所有的子类继承该类可具备父类的属性和行为，继承具有单一性和传递性。
    - 多态：
      - 行为多态：同一个run( ){ }方法，不同的对象调用时会有不同的实现，猫调用时是跑，鱼调用时是游，鸟调用时是飞。
      - 对象多态：同一个对象，可以被造型为不同的类型，比如同一个人对象，可以被造型为儿子，父亲，员工等。


