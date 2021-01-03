[TOC]

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
### B树
1. [来源](https://juejin.im/entry/5b0cb64e518825157476b4a9)
2. 特点：
   1. 所有的叶子结点都位于同一层
   2. B 树是关键字数比子树数少一
   3. B+树是节点的子树数和关键字数相同
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

### 面试题

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
    

## MyBatis

1. [来源](https://tech.meituan.com/2018/01/19/mybatis-cache.html)
2. 一级缓存
   - 默认Session级别，在一个MyBatis会话中执行的所有语句，都会共享这一个缓存。
   - Statement级别,缓存只对当前执行的这一个有效
   - 一级缓存的生命周期和SqlSession一致
   - 一级缓存内部设计简单，只是一个没有容量限定的HashMap
   - 一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement。
3. 二级缓存
   1. MyBatis的二级缓存相对于一级缓存来说，实现了SqlSession之间缓存数据的共享
   2. MyBatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
   3. 在分布式环境下，由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据