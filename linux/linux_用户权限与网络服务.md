# linux用户管理

## 运行模式

1.运行模式也可以称为运行级别。在linux中存在一个init进程，初始化进程。PID为1.该进程存在一个配置文件，inittab(运行级别配置文件),位置/etc/inittab。不过在ubuntu下，已经找不到这个文件了，inittab软件包/etc/init，在/etc/init/下rc-sysinit.conf，在该文件末尾（58行执行init.d/rcS脚本进行初始化）。Ubuntu有0123456sS这几种：

- 0，表示为关机级别（不要将默认的运行级别设置为0）
- 1，单用户模式
- 2，多用户模式，不带NFS（network file system）
- 3，full multiuser多用户模式，完全的多用户模式，带网的
- 4，unused保留模式，
- 5，X11，完整的图形化界面模式
- 6，reboot，重启级别。（不要将默认的运行级别设置为6）
- S，单用户恢复模式，运行很少进程以及服务。（一般没必要改这个）

其中运行命令sudo，其实都是调用的init进程，将数字传递给进程，进程去读取配置文件执行对应的操作：

-  init 0 表示关机
-  init 3 切换到不带桌面的模式，永久命令（initdefault，还要改配置文件）。我在Ubuntu下
   - `sudo vim /etc/default/grub`
   - `GRUB_CMDLINE_LINUX_DEFAULT`改为`"quiet splash 3"`
   - `sudo update-grub`以更改配置，重启
-  init 5 切换到图像界面
-  init 6重启电脑

## 配置文件
1. `/ect/profile`：此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行.并从 /etc/profile.d 目录的配置文件中搜集shell的设置.
2. `/etc/bashrc`：为每一个 运行bash shell 的用户执行此文件.当bash shell被打开时,该文件被读取.
3. `~/.bash_profile`：每个用户都可使用该文件输入 专用于 自己使用的shell信息, 当用户登录时,该文件仅仅执行一次 ! 默认情况下,他设置一些环境变量,执行~/.bashrc文件.
4. `~/.bashrc`：该文件包含专用于用户的bash shell的bash信息 ,当登录时以及每次打开新的shell时,该该文件被读取 .
5. `~/.bash_logout`：当每次退出系统(退出bash shell)时,执行该文件
6. `/etc/fstab`是系统分区的配置文件，开机后系统会自动挂载文件中制定的设备;但是光驱U盘这些移动设备是无法开机自动挂起的，否则将造成系统启动失败。
   - noauto是非自动挂起；

## 用户与用户组管理（*）
三个文件

- /etc/passwd存储用户的关键信息root : x : 0 : 0 : root : /root : bin/bash
   - 用户名：密码：用户id：用户组id：注释：家目录：解释器shell
   - 其中:密码x只是占位；解释有时候没有；家目录是用户登录进入系统之后默认的位置；shell当用户进入系统，用户输入指令后，解释器会收集用户输入的指令传递给内核处理。
- /etc/group存储用户组的关键信息，如果查看主组，在passwd文件，查看附加信息在group文件：在一个用户组后面跟着另一个用户。
- /etc/shadow存储用户的密码信息

1.用户管理

- 添加用户：`useradd 选项 用户名`。其中选项为：
   - -g:表示指定用户的用户组，选项的值可为用户组id，也可为组名
   - -G：表示指定用户的用户附加组，选项的值可为用户组id，也可为组名
   - -u:UID，用户的id，系统默认从1000（别的系统从500开始）之后的顺序分配UID，如果不想用系统分配的，可通过该选项进行自定义。
- 修改用户：`usermod 选项 用户名`。选项与上面一致。
   - -g:表示指定用户的用户组，选项的值可为用户组id，也可为组名
   - -G：表示指定用户的用户附加组，选项的值可为用户组id，也可为组名
   - -u:UID，用户的id
   - -l:表示修改用户名`usermod -l newName oldName`
- 设置密码：`passwd 用户名`
- 改密码：`passwd`直接进行设置
- 切换用户：`su 用户名`如果没有用户名，则为root用户
- 删除用户：`userdel 选项 用户名`
   - -r删除用户的同时，删除家目录。如果失败，kill主进程（root...su 用户名这行的pid）

# linux网络服务

## 网络
1、网卡位置：一般linux在/etc/sysconfig/network-script文件夹下。Ubuntu是在/etc/network/interfaces。DEVICE：设备名。TYPE：Ethernet以太网。ONBOOT：自动启动。BOOTPROTO分配IP方式，有DHCP动态分配主机协议。 HWADDR：硬件地址，MAC地址。

2、服务重启：在/etc/init.d中的快捷方式，找network的相关命令，执行`/etc/init.d/network restart`（扩展，如果目录很深，可以在浅的目录创建快捷方式：软连接`ln -s 路径 路径`，当ls -l列出的时候，以l开头，link就是快捷方式）

3、停止某个网卡`ifdown 网卡名`；开启网卡`ifup 网卡名`

## ssh——secure shell服务（重点）
1、该协议2个常见的协议：远程连接协议、远程文件传输。使用默认端口号为22.可以在/etc/ssh/ssh_config的配置文件中修改。`service sshd start/stop/restart`或者跟上面一样`/etc/init.d/sshd start/stop/restart`

2、远程终端。  
终端工具主要帮助运维人员连接远程服务器，常见工具xshell、secureCRT、Putty。使用前先ping,确保两台服务器相通（如果连接被拒绝，有可能并没有安装sshd服务，选择安装就好了）。因此这样就可以在一台电脑运行远程linux命令。

3、ssh远程文件传输  

- filezilla工具。文件->站点管理器->新建站点，协议选择SFTP。FTP服务器有两个端口，其中21端口用于连接，20端口用于传输数据。
- pscp通过命令行传输文件,其中选项的值可以参考帮助。-v、-q啥的
   - 下载`pscp 选项 用户名@linux 主机地址：资源路径 Windows本地地址`
   - 上传`pscp 选项 资源路径 用户名@linux 主机地址：远程路径`

## 自有服务
1、设置主机名   
一般在/etc/sysconfig/network，修改其中的HOSTNAME的值。但是Ubuntu没有这个。只有/etc/hostname文件，直接在里面改就行了，重启后生效。之后如果发现linux本地无法用新的主机名访问浏览器，就在/etc/hosts文件进行修改。

2、chkconfig   
作用：相当于开机启动的管理服务,ubuntu16好像还需要自己安装这个。·`sudo apt install sysv-rc-conf`然后`sudo cp /usr/sbin/sysv-rc-conf /usr/sbin/chkconfig`就可以使用这个命令了。

- 开机启动服务查询：`sudo chkconfig --list` 一堆数字表示：比如在3这个运行模式下启动的linux，默认某某服务不启动/启动
- 删除服务：`chkconfig --del 服务`，但是Ubuntu下--list能用，但--del又不能使用了：`sudo sysv-rc-conf 服务名 on(off)`。然后可以查一下：`chkconfig --list | grep 服务`。
- 添加开机启动服务。不是所有应用都有服务。软件有的没有服务。同样Ubuntu没有`chkconfig --add 服务`
- 设置某个服务在某个级别下开机启动：`sysv-rc-conf --level 连着一起的启动级别或单个35 服务 on/off`

3.ntp服务   
该服务主要用于计算机时间的同步管理。

- 一次性同步(手动)时间：`ntpdate 时间服务器的Ip或域名`；Ubuntu需要单独装ntpdate，然后再绑定。
- 设置时间同步服务：服务名是ntpd服务；但是Ubuntu是用timedatectl status自动设置，不需要人为处理。

4.防火墙   
有软件防火墙跟硬件防火墙之分。centos6.5的防火墙为iptables，centos7是firewalld。而Ubuntu16的自带防火墙是ufw。

- `ufw status`查看是否启动
- `ufw enable`启动ufw防火墙
- `ufw default deny`启动默认防御（阻止外部联接，放行对外联接）
- `ufw allow 53`允许其它主机访问本机53端口，协议包含tcp和udp
- `ufw allow 25/tcp`允许其它主机使用tcp协议访问本机25端口

5、rpm   
对linux下的软件包进行管理：查询是否安装、卸载、安装。然而Ubuntu又不支持。其实apt-get直接就行了。

- 查询：`rpm -qa | grep 关键词`其中`-q`查询,`-a`全部。
- 卸载：`rpm -e 软件名`如果有依赖关系，`rpm -e 软件名 --nodeps`，其中deps就是dependencis的缩写。
- 软件安装：得到安装包，`rpm -ivh 软件包完整名称`,`-i`就是install安装，`-v`进度条，`-h`以#显示进度条。
   - 扩展：挂载U盘找安装包`mount 设备原始地址  要挂载的位置路径`，设备原始地址统一在/Dev下，根据U盘的大小组成原始地址。要挂载的位置路径一般在mnt。解挂载`umount 当前设备的挂载点路径`
   - －qf：查找指定文件属于哪个RPM软件包[Query File]； 
   - －qpi：列出RPM软件包的描述信息[Query Package install package(s)]； 

6、cron/crontab计划任务   
语法：`crontab 选项`

- `-l`——list,列出指定用户的计划任务列表
- `-e`——edit，编辑指定用户的计划任务列表
   - 语法规则：以行为单位，一行则为一个计划——`分 时 日 月 周 需要执行的命令`,其中周取值为0-6,0为周日。
   - 语法规则：`*`表示取值范围中的每一个数字，例子：`0 0 * * * reboot`每天零点零分重启
   - 语法规则：`-`做区间表达式，例子：表示1到7，写作`1-7`
   - 语法规则：`/`表示每多少个，例子：每十分钟执行一次，写作`*/10`
   - 语法规则：`,`表示多个取值，例子：1,点，2点，6点执行，写作`1,2,6`  
   - 例子：每天18:00至23:00之间每30分钟重启：`*/30 18-23 * * *`
   - 例子：每2分钟往root家目录的rt.txt中输入当前的时间信息，使用追加输出：`*/2 * * * * date >> /root/rt.txt`
- `-u`——user，指定的用户名，如果不指定表示当前用户的
- `-r`——remove，删除当前用户的任务书 
  
权限问题：可以用来设置某些用户不允许设置计划任务书。配置文件在/etc/cron.deny，在里面写用户名，一行一个。不过Ubuntu16不知道在哪。白名单：/etc/cron.allow本身不存在，需要自己创建。默认白名单优先级高。

# 权限问题

## 权限
1、读权限：

 - 文件夹，用户是否可以列出目录结构
 - 文件，用户是否可以查看内容

2、写权限：

 - 文件夹，是否可以在文件夹下创建删除复制移动文档
 - 文件，是否可以编辑文件内容

3、 执行权限：一般对于文件而言，特别是脚本文件是否可以执行。

4、查看权限一般`ls -l`等价于`ll`，例如，对于10位字符drwxr-x---：

- 第一个字符文档类型：-文件，d文件夹，l表示快捷键，s套接字。
- 第2-4字符，rwx文件所有者权限, 2为读，3为写，4为执行，-为空，位置顺序不会变化。
- 第5-7位，表示与所有者在同一个组的用户的权限情况。意思一样。
- 第8-10位，除了上面的前2部分的用户之外的其他用户的权限。

5、设置权限：`chmod 选项 权限模式 文档`，选项为`-R`：递归设置权限，权限模式是文档需要设置的权限信息，只有root、文档所有者才能设置权限。

- 字母形式：

   选项 | 字母 | 介绍 |
   | ------ | ------ | ------ |
   谁 | u | 用户 |
   谁 | g | 所属群体 |
   谁 | o | 其他人 |
   谁 | a | 所有人（不指定时默认） |
   作用 | + | 增加权限 |
   作用 | - | 减少权限 |
   作用 | = | 确定权限（设置成这个权限） |
   权限 | r | 读 |
   权限 | w | 写 |
   权限 | x | 执行 |
   
   例子：将-rw---------权限的文件a.txt设置为所有者拥有全部，同组读执行，其他用户只读：
   - `chomd   u+x,g+rx,o+r  ./a.txt`
   - `chomd   u=rwx,g=rx,o=r  ./a.txt`注意这个u=rwx，是设置成。
   
- 数字形式：

   数字 | 权限 | 目录列表 |
   | ------ | ------ | ------ |
   0 | 没有任何权限 | --- |
   1 |  | --x |
   2 |  | -w- |
   3 |  | -wx |
   4 |  | r-- |
   5 |  | r-x |
   6 |  | rw- |
   7 |  | rwx |
   
   上面说的r为4，w为2，x为1。例子：
   - 上面的例子`chmod 754 ./a.txt`
   - 对于这种命令`chmod 731 ./a.txt`的问题是，3=2+1,能写能执行，但是不能读，说明这个命令有问题。
   - 在linux中，删除一个文件，不是看该文件是否有写权限，要看其所在的文件夹是否有写权限。
   
> 文件默认权限是666,unmask 002，就是拿掉002权限，现在就是664
   
2. 有一种`chmod -t`：[如果某个目录设置了sticky bit](http://www.360doc.com/content/14/0216/20/13327838_353033771.shtml)（是在other用户的权限上设置的，设置后可执行位从x变成了t），那么用户在该目录下可以创建文件（当然前提是用户具有写权限和可执行权限，如 果具有可执行权限，设置sticky bit后是t；如果没有可执行权限的话，设置sticky bit后是T），而且可以删除自己创建的文件，但是，不能删除其他用户创建的文件，这样就起到了一种保护作用了。

## 属主与属组设置
1、属主是文件的主人，属组所属的用户组。`ls -l`后输出的有2个用户信息，例如`-rw-rw-r-- 1 xu xu 804 1月   8 11:25 xuxu`，第一个就是属主，后面的就是属组。

2、chown   
作用：更改文档的所属用户   
语法：`chown -R username 文档路径`，其中的-R如果是文件夹就需要。如果将23结合就是`chown -R username:groupname 文档路径`。

3、chgrp（了解）   
作用：更改文档的所属用户组   
语法：`chgrp -R groupname 文档路径`

## sudo扩展
1、reboot、shutdown、init、halt、user等命令，普通用户操作不了，但又想用。可以使用sudo命令（switch user do切换用户）。事先定义某些特殊命令谁可以执行。在/etc/sudoers文件夹下。但是只有root才能查看，但是Ubuntu需要知道root密码，`sudo passwd`。然后输入密码就是设置密码了。然而只能读，即使是root。

- 配置/etc/sudoers，使用`visudo`命令,直接就是这个语句，使用方法与vim一致。但是如果是Ubuntu16，则ctrl+x 退出。ctrl+o 保存后回车再退出。打开后：
   - `root ALL=(ALL:ALL) ALL`，案例：本身test用户不能添加用户，将其设置为可以添加用户并可以修改密码（除root）：需要写命令的完整路径查看指令路径的方式`which 指令`：`xu ALL=(ALL) /usr/sbin/useradd,/usr/bin/passwd`如果是不能改root密码：`xu ALL=(ALL) /usr/sbin/useradd,/usr/bin/passwd,!/usr/bin/passwd root`不过需要注意的是，第一次安装Ubuntu时新建的用户依然可以设置root密码。`sudo -l`查看单独配置的命令
      - root表示能使用sudo命令的用户；如果是用户组，写作`%组名`
      - 第一个ALL表示允许使用sudo的主机；（可以是localhost或者啥）
      - 第二个ALL及第三个ALL表示以什么身份去执行（这里以ALL身份执行）；
      - 第四个ALL表示sudo可执行的命令，即所有命令；在有的系统中也简写做：`root ALL=(ALL) ALL`
## sh
1. 远程执行命令：ssh root@127.0.0.1 "chown ossadm:ossgroup /opt/pub/software"
