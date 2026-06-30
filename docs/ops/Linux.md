# Linux

## 一.开始

### 1.1 VM Ware安装

* 运行VMware-workstation-full-15.5.0-14665864.exe ，一直下一步
* 出现许可证，点击输入许可证将密钥放入即可
* 进入任务管理器——查看性能，检查是否开启虚拟化；未开启需要在BIOS开启VT

==注意：安装完成之后，在网络连接中检查虚拟机是否具有两个虚拟网卡并且是否开启；若未开启，后期会影响虚拟机和操作系统之间的相互通信(共享网络、文件传输)==



### 1.2 初始化虚拟机

1. 打开VMware，新建虚拟机，**选择自定义**
2. 然后下一步选择**稍后安装制作系统**，选择**linux系统**，**Centos7 64位**，然后位置自定义即可
3. **选择2处理器+2内核**即可，**内存选择2GB**，选择**使用网络地址转换**，然后一直下一步即可完成
4. 完成之后可以选择**编辑虚拟机设置**进行重新配置，将CD/DVD选择**使用ISO镜像文件**，找到Centos7系统路径即可
5. 开启虚拟机选择安装Centos 7系统，然后**软件安装**可以选择安装**GHOME（图形化界面的方式）**
6. 最后设置root密码，创建其它账户即可，重启输入密码时不会显示密码

==注意：1.当我们点入到虚拟机以后鼠标箭头将会消失，我们可以同时按住Ctrl+Alt键让鼠标重现==

​			==2.在GHOME桌面点开设置-隐式-锁屏，可以关闭锁屏；若不关闭，锁屏的时候可以由下往上拉可以解锁==



### 1.3 安装Xshell6

#### 1.3.1 安装环境

1. 解压 **Xshell6_Plus破解版.zip**，运行**!绿化.bat**
2. 使用xshell.exe 

==注意：若出现**由于找不到mfc110u.dll,无法继续执行代码,重新安装可能会解决此问题**，请安装`vcredist_x64.exe`和`vcredist_x86.exe`==



#### 1.3.2 连接Xshell

1. 打开xshll，文件—新建; 名称自定义，主机为Linux系统的IP地址，然后点击连接即可

> ifconfig				//可以查看自己的ip地址

==注意:如果发现e开头的没有ip地址，是因为没有开启DHCP配置，则要输入下列命令后再次查看==



> sudo dhclient							

==不是永久的ip地址，每次关机后都要开启，参考1.5配置静态IP地址==



2. 出现密钥，选择接收并保存，输入用户名和密码即可
3. 修改字体大小: 文件—属性—外观



#### 1.3.3 快速使用

>  ssh ip地址					//可以快速登录
>
> ping ip地址 -t			    //可以快速连接重启的虚拟机



### 1.4 安装Xftp

​			运行xftp.exe即可



#### 1.4.1 连接Xftp

1. 打开Xftp，文件—新建; 名称自定义，主机为Linux系统的IP地址，输入用户名和密码，然后点击连接即可



### 1.5 配置静态IP地址

`1.根据自己的环境,使用root权限修改/etc/sysconfig/network-scripts/ifconfig-en...`

```
vim /etc/sysconfig/network-scripts/ifcfg-en....			
```



`2.在打开的文件中,输入a进入修改模式,修改以下配置`

```
BOOTPROTO=static									//改为静态static
#UUID=1008581b-d7b1-498e-b96b-e59217a7dd1c			//注释掉
ONBOOT=yes											//改为yes
IPADDR=192.168.2.102								//添加ip地址
NETMASK=255.255.255.0								//子网掩码
GATEWAY=192.168.2.2									//网关
DNS1=114.114.114.114								//dns
```

==注意：接着按下ESC,输入 :x 表示保存,  :q! 表示退出而不保存==



`3.重启网络服务`

```
systemctl restart network
ip a
```

==注意：接着查看自己en.... 的ip地址==



`4.VMware编辑——虚拟网络编辑器`

```
第一步：找到上面的VMnet8,将下面的子网改为192.168.2.0
第二步:点击上方的NAT设置,将网关ip改为192.168.2.2
```



`5.虚拟机和本机测试网络连接`

```
虚拟机: ping www.baidu.com
本机cmd: ping 192.168.2.102
```



### 1.6 详细配置

​			详细配置见 <FONT COLOR="RED">`VMware安装及CentOS 7镜像安装步骤.docx`</FONT>



### 1.7 Linux常用目录概述

​																					**`Linux中一切皆文件`**

* /bin：存放着经常使用的命令
* /etc：存放所有系统管理所需要的配置文件和子目录
* /opt：给主机额外安装软件所摆放的目录，如mysql，jdk，redis，es等
* /root：超级管理员的目录
* /usr：普通用户目录，很多用户的应用程序和文件被放在这个目录下，类似于Windows 的program files
* /home：用户的主目录，每个用户都有一个目录，目录名以自己的账号命名



### 1.8 快照

​				保留当前系统信息为快照，随时可以恢复，以防系统因安装失败而出现环境问题，类似于游戏中的存档。

详细用法见`3.3.2`



## 二. 概述

### 2.1 路径

* 绝对路径：从**根目录/开始**描述的为绝对路径，要写出完整路径，如 /root/etc/inittab
* 相对路径：从当前位置开始描述的为相对路径，如  inittab/life

> .(1个点)：表示当前目录
>
> ..(2个点)：表示上一级目录，即父级目录



### 2.2 Linux基本命令

#### `0.取消执行命令`

​		使用ctrl+c可以取消执行所有的Linux命令



#### `1.切换用户`

> su  用户名
>
> su - 用户名		//切换用户的同时切换到对应的用户目录

<font color="green">#：表示超级用户，$：表示普通用户</font>



#### `2.查看文件列表`

> ls  									//列出当前目录下的文件
>
> ls  指定名称				//列出指定名称下的文件
>
> ls -a							//列出所有文件包含隐藏文件

==ll查看的更加详细==

#### `3.切换工作目录`

> cd    /					//跳转到根目录
>
> cd  ~					//跳转到当前用户主目录
>
> cd  ..					//返回上一层



#### `4.清屏`

> clear  或  ctrl+L



#### `5.创建文件夹`

> mkdir 	/home/jj/eobard/test		//通过绝对路径创建文件夹(**前提是 eobard文件夹存在，否则会错误**)
>
> mkdir  -p   /home/jj/eobard/test	//通过绝对路径递归创建文件夹(**不用管是否存在eobard，都可以创建**)
>
> mkdir  文件夹名1   文件夹名2   		//以空格隔开表示创建多个文件夹，否则创建单个



#### `6.创建文件`

> touch  文件							//创建一个文件
>
> touch  /home/jj/文件		  //通过绝对路径创建文件
>
> touch  文件1  文件2			//以空格隔开表示创建多个文件



#### `7.重命名`

> mv   旧文件名   新文件名		//将后缀名类型一致的旧文件名改为新文件名；包含了文件夹、文件



#### `8.移动文件`

> mv 文件1    文件夹2				//将后缀名类型不一致的文件夹1移动到文件夹2中



#### `9.复制文件`

> cp  文件  文件夹					//将文件复制到文件夹中
>
> cp  -r  文件夹1  文件夹2	 //将文件夹1的所有文件复制到文件夹2中



#### `10.删除文件`

> rm  -r  文件			//删除文件及其子目录
>
> rm -f   文件			//强制删除文件(force)
>
> rm -rf  文件			//强制删除文件及其子目录

==注意：切忌不能执行 rm -rf  / 表示删除整个根路径，整个Linux系统就没了==



#### `11.查看文件/合并文件`

> cat  文件名									//查看文件的内容
>
> cat  文件1  文件2  >>文件3		//将文件1和文件2的内容追加在文件3的末尾



#### `12.查看文件指定开头/末尾行数`

> head -n 行数 文件名		//查看开头指定 行
>
> tail  -n  行数  文件名			//查看末尾指定行



#### `13.文件搜索命令`

> find * .txt	//查看以.txt结尾的文件

```nginx
find 路径地址 -name  *文件名*		//可以根据路径模糊查询文件名的位置
```





#### `14.归档管理`

> * -z：打包同时压缩
> * -x：解压文件
> * -f：指定压缩后的文件名
> * -c：产生.tar打包文件
>
> 
>
> **压缩：** 
>
> ​			tar	-zcf    xxx.tar.gz   指定文件1 指定文件2			//将指定文件压缩成 xxx.tar.gz
>
> **解压**
>
> ​			tar -zxf  xxx.tar.gz								//将xxx.tar.gz文件解压到当前目录
>
> ​			tar -zxf  xxx.tar.gz  **-C**  **./**文档			//将xxx.tar.gz文件解压到当前目录的文档下

==**./**：表示当前目录； **-C**：表示指定路径==



#### `15.文件权限`

##### 15.1 概念

​				drwxr-xr-x  用户名  组名   

解析： 第一个字母d代表文件夹，- 代表普通文件；从第二个字母开始算，数三个分别代表

* rwx ： 代表用户权限为     读、写、执行
* r-x：    代表用户组权限为 读、执行
* r-x：    代表其它组权限为  读、执行

```
三种权限:
	r=4     读的权限
	w=2    	写的权限
	x=1     执行的权限
```



##### 15.2 字母法

> chmod u+rwx  文件名		//给当前用户添加读、写、执行三种权限
>
> chmod u-rw-   文件名		//给当前用户减少读、写的权限，只有执行



##### 15.3 数字法(推荐)

> chmod  754 文件名		//给当前用户rwx，当前用户组为r-x，其它用户组为--r



#### `16.修改文件拥有者`

>  chown -R  用户名  文件或文件夹		//将指定文件名的所有变为指定用户名拥有



#### `17.查看网络配置`

> ifconfig
>
> ip addr 
>
> ip a



#### `18.下载命令`

> wget  url		//从网络上下载

 

#### `19.开关机`

> root权限:  `init 0关机`    `reboot 重启 ` shutdown -h now 关机



#### `20.防火墙相关`

> systemctl status firewalld   																			//查看防火墙状态
>
> systemctl restart firewalld.service																//重启防火墙
>
> systemctl stop firewalld																					//停止防火墙
>
>  systemctl start firewalld							 													//开启防火墙 
>
>  firewall-cmd --zone=public --list-ports													//查看所有打开的端口
>
> firewall-cmd --zone=public --add-port=xx/tcp --permanent			//永久开放xx端口
>
> firewall-cmd --zone= public --remove-port=xx/tcp --permanent	//永久移除xx端口		
>
> firewall-cmd --reload																							//重新加载防火墙



### 2.3 用户管理

#### 2.3.1 新增用户

> useradd  用户名			//新增用户，需要在root权限下



#### 2.3.2 新增/重置密码

> passwd 用户名			//重置或新增用户名的密码，需要在root权限下



#### 2.3.3 重置用户名

> usermod -l 新用户名 -d /home/新用户名 -m  旧用户名		//需要在root权限下



#### 2.3.4 删除用户

> userdel  用户名				//单纯删除用户
>
> userdel -r  用户名			//删除用户的同时删除/home下面的目录



### 2.4 Vim的使用

#### `1.在线安装vim`

> yum -y  install vim



#### `2.使用编辑器`

```nginx
#输入命令
vi 文件名
```

==注意：如果存在这个文件，则修改；如果不存在这个文件，则新建==



#### `3.编辑命令`

​			任意按下以下两个键，可以进入编辑模式

| 命令 | 作用             |
| ---- | ---------------- |
| i    | 在光标前插入文本 |
| a    | 在光标后附加文本 |
| gg   | 跳到首行         |
| G    | 跳到末行         |



#### `4.退出编辑命令`

​			使用Esc键可以退出插入命令



#### `5.保存、退出、撤销命令`

​			在使用Esc键退出后，再次输入相应命令执行操作

| 命令       | 作用                                  |
| ---------- | ------------------------------------- |
| :w         | 保存修改                              |
| :w  文件名 | 另存为新的文件名                      |
| :wq        | 保存修改并退出                        |
| :q!        | 不保存修改并退出                      |
| /字符      | 查询字符在文件往下查询，按n继续往下找 |
| ?字符      | 查询字符在文件往上查询，按N继续往上找 |
| :set nu    | 设置行号，可以在每行中显示出当前行号  |
| :set nonu  | 取消设置行号                          |



### 2.5 进程管理

#### 2.5.1 基本概念

* 1. 在Linux系统中，每一个程序都有一个自己的进程，每个进程都有一个id
* 2. 进程可以由两种方式存在：前台运行、后台运行



#### 2.5.2 查询当前进程

> **ps  -aux					//查看所有进程**
>
> ps -ef 						//查询所有进程包括父进程
>
> ps -ef|grep xxx	  //查询跟xxx相关的进程

` eg: ps aux |grep java  查询跟java相关的进程`



#### 2.5.3 以进程数查看进程

> pstree  -pu						//查看所有进程
>
> pstree  -pu|grep xxx	//查询跟xxx相关的进程



#### 2.5.4 结束进程

> kill  -9  进程id		//强制结束进程



### 2.6 管道符

​					命令A | 命令B，将命令A的输出作为命令B的操作对象

> 命令a | grep  搜索字符串		//将命令a的执行结果查找指定的搜索字符串



### 2.7 清空文本内容

> `> 文件名`
>
> `echo "" > 文件名`

## 三. 配置开发环境

### 3.1 JDK

#### 3.1.1 配置

`1.首先切换为root,在/opt下创建java文件夹`

> cd /opt
>
> mkdir java



`2.运用Xftp将JDK放入/opt目录下`

==注意：若遇到传输错误，确保是否为root权限，再次退出重传==



`3.将压缩文件解压到当前目录下的java文件夹`

> tar -zxf  jdk......    -C  ./java



`4.配置Java环境`

* 进入编辑模式

> vim  /etc/profile

* 在最后几行配置Java环境

```nginx
#这里是刚刚解压缩的位置,后期要更改jdk版本，只需要改这里即可
JAVA_HOME=/opt/java/jdk1.8.0_11  
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME
export PATH
export CLASSPATH
```

==/etc/profile：所有环境配置的文件==



`5.刷新环境变量`

> source  /etc/profile



`6.测试Java环境`

>  java -version

**<FONT color="green">出现以下则成功</FONT>**

```nginx
java version "1.8.0_11"
Java(TM) SE Runtime Environment (build 1.8.0_11-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.11-b03, mixed mode)
```



#### 3.1.2 部署SpringBoot项目

`1.打包SpringBoot项目(jar包),并通过Xftp上传到Linux任意路径中`

==注意：如果SpringBoot项目的端口号在Linux中没有对外开放，则需要开放端口供Windows访问，见 `3.2.2`==



`2.在刚刚上传的路径执行命令`

> java -jar  打包文件名			



`其它: linux后台运行jar文件`

​		表示后台运行jar文件，并把运行的日志打印在**java_running.txt**(可自定义文件名)中

> #后台启动
>
> nohup java -jar 打包文件名 >java_running.txt  &
>
> #查看后台
>
> ps aux|grep java

```nginx
eg:
nohup java -jar xx.jar --spring.profiles.active=test --server.port=9002  >java_running.txt &
```





### 3.2 Tomcat

#### 3.2.1 配置

`1.切换为root,在/opt下创建tomcat文件夹`

> cd /opt
>
> mkdir tomcat



`2.运用Xftp将tomcat放入/opt目录下`

==注意：若遇到传输错误，确保是否为root权限，再次退出重传==



`3.将压缩文件解压到当前目录下的tomcat文件夹`

> tar -zxf  tomcat......    -C  ./tomcat



`4.启动tomcat`

> **一定要进入tomcat文件夹下的bin文件夹(即/opt/tomcat/apache-tomcat-8.5.72/bin)**,再启动
> [root@localhost bin]#  ./startup.sh 



**<FONT color="green">出现以下则成功</FONT>**

```
Using CATALINA_BASE:   /opt/tomcat/apache-tomcat-8.5.72
Using CATALINA_HOME:   /opt/tomcat/apache-tomcat-8.5.72
Using CATALINA_TMPDIR: /opt/tomcat/apache-tomcat-8.5.72/temp
Using JRE_HOME:        /opt/java/jdk1.8.0_11
Using CLASSPATH:       /opt/tomcat/apache-tomcat-8.5.72/bin/bootstrap.jar:/opt/tomcat/apache-tomcat-8.5.72/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
```

**然后在VMware图形化界面访问http://localhost:8080/ 就可以看到Tomcat的logo**



`5.关闭tomcat`

> **一定要进入tomcat文件夹下的bin文件夹(即/opt/tomcat/apache-tomcat-8.5.72/bin)**,再关闭
> [root@localhost bin]#  ./shutdown.sh 



#### 3.2.2 注意事项

* 1. 如果在根路径/想要开启/关闭tomcat，就应该写出完整路径，但不能忘记开头的./

> 如：/opt/tomcat/apache-tomcat-8.5.72/bin/startup.sh
>
> ​		/opt/tomcat/apache-tomcat-8.5.72/bin/shutdown.sh



* 2. 如果Windows想要访问Linux的tomcat，需要开放防火墙端口并且修改IP访问地址

  `第一步:开放对外的8080访问端口`

> firewall-cmd --zone=public --add-port=8080/tcp --permanent			
>
> firewall-cmd --reload



==注意: 若要想关闭端口，输入以下命令==

> firewall-cmd --permanent --zone=public --remove-port=8080/tcp
> firewall-cmd --reload



​	`第二步:Windows修改IP地址`

```nginx
http://Linux自己的IP地址:8080/
```





### 3.3 Mysql

#### 3.3.1 拍摄快照

​				**在安装Mysql之前，一定要拍摄快照备份系统，防止因为Mysql没装上而损坏系统，下一次可以根据快照恢复到之前的系统**	

> 打开VMware-选择当前的虚拟机右键选择快照-选择拍摄快照



#### 3.3.2 安装

`1.切换为root,在/opt下创建mysql文件夹`

> cd /opt
>
> mkdir mysql



`2.运用Xftp将mysql放入/opt目录下`

==注意：若遇到传输错误，确保是否为root权限，再次退出重传==



`3.开始安装`

```nginx
#查找是否有mariadb相关包,必须卸载,否则安装不上
rpm -qa | grep -i mariadb-lib

#首先要卸载mariadb的相关包
rpm -e postfix-2:2.10.1-6.el7.x86_64

#卸载mariadb
rpm -e mariadb-libs-5.5.44-2.el7.centos.x86_64

#再次查看
rpm -qa | grep -i mariadb

#首先安装依赖的autoconf文件
yum -y install autoconf

#解压
tar -xvf MySQL-5.6.25-1.el7.x86_64.rpm-bundle.tar -C ./mysql

#进入目录安装
cd mysql/
#先安装服务端
rpm -ivh MySQL-server-5.6.25-1.el7.x86_64.rpm 
#再安装客户端
rpm -ivh MySQL-client-5.6.25-1.el7.x86_64.rpm

#关闭防火墙
vi /etc/sysconfig/selinux 
#修改为 SELINUX=disabled
#重启
reboot

#启动mysql服务
systemctl start mysql

#查看root账号的密码,并保存下来  Z4mhApao77K5fbUu
cat /root/.mysql_secret	

#登录并未root用户设置新密码
mysql -uroot -p刚刚获取的密码

#设置自己的root密码
set password=password('自定义密码');
flush privileges;
```

> 若忘记了密码，可以重置密码

> *  vi /usr/my.cnf    					//在空白地方加入	**skip-grant-tables**
> * systemctl restart mysql       //重启mysql服务
> * UPDATE mysql.user SET password=PASSWORD("自定义密码") WHERE user='root';   //更新
> * flush privileges;                   //提交
> * vi /usr/my.cnf                      //将刚刚新增的skip-grant-tables注释掉
> * mysql -uroot -p密码         //再次登录即可



#### 3.3.3 开启远程连接

​					如果Windows想要访问Linux的mysql，则需要先在Linux的mysql中开启远程连接

**<font color="green">1.在mysql语句中，开启远程连接Linux支持</font>**

> grant all privileges  on *.* to 'root' @'%' identified by '自定义的密码';
>
> flush privileges;

**<font color="green">2.退出mysql语句，在Linux系统开放3306端口供外访问</font>**

> firewall-cmd --zone=public --add-port=3306/tcp --permanent			
>
> firewall-cmd --reload

**<font color="green">3.在Windows下使用Navicat等工具新建连接即可，需要更改主机名为Linux的IP地址</font>**



#### 3.3.4 修改mysql编码格式

​			当我们用Windows远程连接Linux MySQL时创建表数据的时候，会提示错误1245，就是不让我们输入中文信息，这时候就要更改Linux系统下MySQL的默认编码格式。

`1.关闭mysql服务`

> systemctl stop mysql



`2.修改my.conf配置文件`

> vi  /usr/my.cnf 

```properties
#在末尾加上以下代码并保存退出
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8
```



`3.重启mysql服务即可`

> systemctl start mysql





### 3.4 Nginx

#### 3.4.1 环境准备

>  1. 安装gcc的环境

```nginx
yum -y install gcc-c++
```



>  2. 安装PCRE环境

```nginx
yum  install -y pcre pcre-devel 
```



>  3.安装zlib

```nginx
yum install -y zlib zlib-devel
```



> 4.安装OpenSSL

```nginx
yum install -y openssl openssl-devel
```



> 5.root权限进入/usr/local目录

```nginx
cd /usr/local
```



> 6.使用XFTP将`nginx-1.16.1.tar.gz`文件上传至该路径



> 7.该目录下创建文件夹 software

```nginx
mkdir software
```



> 8.解压文件到software中

```nginx
tar -zxf nginx-1.16.1.tar.gz -C ./software/
```



> 9.进入software目录，再次创建nginx文件夹

```nginx
cd software/
ll
mkdir nginx
```



> 10.进入解压后的文件路径，执行命令

```nginx
cd nginx-1.16.1/
./configure  --prefix=/usr/local/software/nginx
make
make install
```

==**注意：如果执行make命令报错，说明缺少依赖，继续执行下列命令后再次执行make即可**==

```nginx
yum -y install gcc automake autoconf libtool make 
```



> 11.进入nginx文件夹，出现以下四个文件夹则说明安装成功

```nginx
cd ..
cd nginx
ll
```

**conf  html  logs  sbin**



#### 3.4.2 启动、关闭、重启

##### 3.4.2.1 启动

`1.启动`

```nginx
#进入安装成功的目录
cd /usr/local/software/nginx/sbin
#启动:默认以后台启动
./nginx
#查看进程
ps aux|grep nginx
```



`2.linux系统访问`

​			打开火狐，输入localhost即可看到welcome to nginx		



`3.Windows系统访问`

* 在Linux系统开放80端口

```nginx
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload 
```

* 打开Windows浏览器，输入Linux系统**IP地址:80**( `本机：192.168.2.102:80`)即可看到welcome to nginx		



##### 3.4.2.2 关闭

```nginx
#正常关闭
./nginx -s quit
#强制关闭
./nginx -s stop
```



##### 3.4.2.3 重启

​			修改了配置文件一定要重启nginx

```nginx
#再刷新启动
./nginx -s reload
```



##### 3.4.2.4 注意

>  无论是启动、关闭、重启nginx都要来到`/usr/local/software/nginx/sbin`这个路径再执行



#### 3.4.3 部署静态网站

`1.将自己的静态文件夹(如前端页面)上传到/usr/local/software/nginx/路径下`

> 注意：这里为什么要把静态文件夹放在这个路径下进行部署，是因为在nginx配置文件中，优先查询在nginx这个相对路径目录下的文件，比如它默认跳转的是html文件夹，它也是在nginx路径下的



`2.进入/usr/local/software/nginx/conf路径,修改访问地址的值`

```nginx
	#服务配置
    server {
        listen       80;   		     #端口号,默认是80
        server_name  localhost;		 #服务器名称,默认是localhost

	   #访问地址
        location / {			# / 表示根目录
            root   前端页面;	 # root表示访问的文件夹,默认是html,可以根据自己的文件夹名称修改
            index  index.html index.htm;	#默认访问页面为index.html,可以根据自己默认页面修改
        }
    
    
    
    #如果要部署多个静态项目,则使用下面
        location /second {			# /second 表示访问目录为: ip/second
            alias   静态页面2;	 	 # 访问的文件夹是 静态页面2,这里只能用alias
            index  login.html;		#默认访问页面为login.html
        }
    
    
    

	#错误页面的配置
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```

==**注意：我们只需要改访问地址那一栏的root和index值即可**==



`3.修改了配置文件,重启服务并访问`

```nginx
./nginx -s reload
```



`4.配合花生壳实现内网穿透`



`5.其它`

​		如果要配置多个静态项目部署，则配置多个location或多个server( 端口不要冲突了且要开放对应端口)

```nginx
	#服务配置1
    server {
        listen       80;   		    
        server_name  localhost;		 

	   #访问地址
        location / {			
            root   前端页面;	
            index  index.html index.htm;	
        }

	#错误页面的配置
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
	#服务配置2
    server {
        listen       81;   		    
        server_name  localhost;		 

	   #访问地址
        location / {			
            root   前端页面;	
            index  index.html index.htm;	
        }

	#错误页面的配置
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```



#### 3.4.4 location 优先级匹配

* ` = /xxx`最高优先级，全等匹配

  ```nginx
  eg:  
  location = /user {			
             #省略其它
          }
  ```

  > 输入/user即可跳转到这个location下



* `^~ /xxx`次高优先级，半匹配

  ```nginx
  eg:  
  location ^~ /user {			
             #省略其它
          }
  ```

  > 输入/user/a即可跳转到这个location下



* `~ /正则表达式`低优先级，正则表达式匹配

  ```nginx
  eg:  正则表达式,表示开头为字母、数字、下划线
  location ~ /\w {			
             #省略其它
          }
  ```

  > 输入/user、/1user、/_user 即可跳转到这个location下



* `/`最低优先级

  ```nginx
  eg:  
  location / {			
             #省略其它
          }
  ```

  > 随意输入路径即可跳转到这个location下



#### 3.4.5 负载均衡

##### 3.4.5.1 概念

​			随着用户的增多，业务流量越来越大并且业务逻辑也跟着越来越复杂，单台服务器的性能及单点故障问题就凸显出来了，因此需要多台服务器进行性能的水平扩展及避免单点故障出现。那么如何将不同用户的请求流量分发到不同的服务器上呢？使用多台WEB服务器组成集群，**`前端使用Nginx负载均衡，将请求分散到我们的后端服务器集群中，实现负载的分发。那么会大大提升系统的吞吐率、请求性能、高容灾`**。



##### 3.4.5.2 使用

```nginx
   #自定义负载均衡两个服务器地址
	upstream group{
    	#地址1,可以设置权重来重点请求某个路径
        server 192.168.2.102:8080 weight=5;
    	#地址2
        server 192.168.2.102:8081 weight=2;
        }

    server {
        listen  80;
        server_name  localhost;

        location /fzjh {
        		#使用下列路径来实现负载均衡
                proxy_pass http://group/;	
          }

        }
```





## 四.守护进程

### ubuntu下安装

```bash
sudo apt update
sudo apt-get install screen
```





### 使用

新建会话  

```bash
screen -S 自定义名字
```



查看会话			

```bash
screen -ls
```



进入会话			

```bash 
screen -r  自定义名字
```

eg：  23352.xxx   xxxx  （Detached），可以使用screen -r  23352进入这个会话



退出会话但保持会话还在后台运行(需要在会话中使用)		

```
ctrl+a+d
```



退出并结束当前会话(需要在会话中使用)		

	exit



## 五.脚本
### 大文件分片打包chunkPack
> **假设一个图片训练集5G，里面有上千张图片，直接下载下来会很慢，这时候可以使用分片打包，将每1000个打包成压缩包，依次下载**

```bash
#!/bin/bash
# Author: Eobard Gu
# Date: 2025/7/7 15:30:30

# 需要分片打包的文件路径地址
SRC_DIR=~/live-spoof/datasets/train_data/org_1_300_300有AI人脸/real
# 分片打包压缩的目标保存路径
DST_DIR=~/live-spoof/datasets/train_data/result/real
# 临时记录文件
TMP_LIST=/tmp/tmp_file_list.txt

# 获取SRC_DIR地址中最后一个路径的名称：eg：BASENAME=real
BASENAME=$(basename "$SRC_DIR")  

mkdir -p "$DST_DIR"
cd "$SRC_DIR" || exit

# 获取相对路径列表（仅文件），写入中间文件
find . -type f | sort > "$TMP_LIST"

# 总文件数
TOTAL=$(wc -l < "$TMP_LIST")
echo "总文件数：$TOTAL"

# 分片打包的大小
CHUNK_SIZE=1000
PART=1
START=1

while [ $START -le $TOTAL ]; do
    END=$((START + CHUNK_SIZE - 1))
    if [ $END -gt $TOTAL ]; then
        END=$TOTAL
    fi
    
    TAR_NAME="$DST_DIR/${BASENAME}_part_${PART}.tar"  
    # TAR_NAME="$DST_DIR/real_part_${PART}.tar"
    
    sed -n "${START},${END}p" "$TMP_LIST" | xargs tar -cf "$TAR_NAME"
    
    echo "打包第 $PART 部分：$TAR_NAME，包含文件 $START 到 $END"
    
    PART=$((PART + 1))
    START=$((END + 1))
done

# 清理中间文件
rm "$TMP_LIST"
```



