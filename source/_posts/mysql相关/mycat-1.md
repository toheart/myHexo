---
title: mycat入门使用
date: 2019-06-14 18:18:57
tags:
 - mycat
 - 数据库
 - mysql

categories:
 - 运维
 - mysql
---

# mycat入门使用

## 安装

### 1.java的安装

- 下载jdk.tar.gz

```shell
wget http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz
```

- 解压并拷贝

```shell
tar zxvf jdk-8u181-linux-x64.tar.gz
mv jdk1.8.0_151/ /usr/local/java/
```

- 配置环境变量

```shell
vi /etc/profile #在文件最后加入以下几行
export JAVA_HOME=/usr/local/java
export JRE_HOME=${JAVA_HOME}/jre
export CLASS_PATH=.:$JAVA_HOME/lib/dt,jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:%JRE_HOME/bin
source /etc/profile #重新加载配置
```

- 验证

```shell
java -version
```



### 2.mycat的安装

- 下载源代码，解压即可。

### 3.试运行

- 进入mycat的解压目录
- ./bin/mycat console



## 目录结果解释

### 目录结构如下所示：

mycat
├── bin		程序目录，存放了各种脚本命令，比如常用的mycat。
├── catlet	人工智能分片目录
├── conf	各类配置文件的目录
├── lib		存放mycat依赖的jar文件
├── logs	存在mycat所产生的日志文件。
└── version.txt	



## 一些常用命令

进入bin目录下

```shell
./mycat start  	#启动mycat, 后台运行
./mycat stop	#停止mycat
./mycat console	 #启动mycat, 前台运行
./mycat restart	 #重启mycat
./mycat pause	#暂停mycat
./mycat status	#查看mycat的状态

mysql -uroot -p123456 -P8066 -h 127.0.0.1 #连接数据接口的命令
```



## 配置文件的介绍

 mycat主要的配置文件分为以下几个：

server.xml 		系统相关的配置文件

schema.xml		逻辑库，逻辑表，dataHost，dataNode配置文件

rules.xml		分片的规则定义文件



# 数据库读写分离的实例

## 分为以下三步：

### 一、数据库配置主从赋值

首先配置两台机器，一台作为master，一台作为slave

测试环境下我使用的是docker来配置：

​	master: 127.0.0.1:3306

​	slave： 127.0.0.1:3309	

#### 配置master

1、设置mysql用户，为其创建一个用来数据同步的用户。

```shell
mysql>create user repl;
mysql> grant replication slave on *.* to 'repl'@'192.168.1.10' identified by 'repl';
mysql> flush privileges;
```

2、配置my.cnf文件

```shell
# 基本的配置
[mysqld]
server-id=1   #数据库服务的唯一标识
log-bin=mysql-bin   # bin文件前缀

#其他配置文件
# expire_logs_days= 10   日志有效期
# binlog-do-db= test 需要记录二进制日志的数据库
# binlog-ignore-db= mysql 不需要记录二进制日志的数据库
# replicate-ignore-db=mysql 不需要同步的数据库
```

3、重启mysqld

4、进入mysql查看master的状态, 获得pos

```shell
mysql> show master status;
```





#### 配置slave

1、配置my.cnf

```shell
[mysqld]
log-bin=mysql-bin
server-id=3
slave-net-timeout=60

```

2、重启Mysqld

3、进入mysql命令行，开始连接master

```shell
stop slave;
change master to master_host='192.168.1.6',master_user='repl',master_password='repl@sangfor',master_port=3306, master_log_file='master-bin.000033',master_log_pos=154;
start slave;
show slave status;

```

4、验证成功：

​	Slave_IO_Running: Yes	 

​        Slave_SQL_Running: Yes



MYSQL[Got fatal error 1236]的原因和解决办法

参考：https://yq.aliyun.com/articles/27685



### 二、mycat配置读写分离

主要配置schema.xml

```xml
<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
writeType="0" dbType="mysql" dbDriver="native" switchType="2" slaveThreshold="100">
    <heartbeat>show slave status</heartbeat>
    <!-- can have multi write hosts -->
    <writeHost host="hostM1" url="localhost:3306" user="root" password="123456">
    <!-- can have multi read hosts -->
    <readHost host="hostS1" url="localhost:3309" user="root" password="123456"
    weight="1" />
</writeHost>
</dataHost>
或者
<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
writeType="0" dbType="mysql" dbDriver="native" switchType="2" slaveThreshold="100">
    <heartbeat>show slave status</heartbeat>
    <writeHost host="hostM1" url="localhost:3306" user="root" password="123456">
    </writeHost>
    <writeHost host="hostS1" url="localhost:3307" user="root" password="123456">
</writeHost>
</dataHost>

```

以上两种取模第一种当写挂了读不可用，第二种可以继续使用，

启动则Ok了。



### 三、迁移数据库

mysql迁移数据库命令：

​	mysqldump [数据库名] -uroot -p'密码'  --master-data=2 --single-transaction -R --triggers -A > /backup/all.sql

  说明：
​	--master-data=2代表备份时刻记录master的Binlog位置和Position

​	--single-transaction意思是获取一致性快照

​	-R意思是备份存储过程和函数

​	--triggres的意思是备份触发器

​	-A代表备份所有的库

在数据库中新建数据库：

​	CREATE DATABASE OSS_BUSINESS_data DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

导入数据：

​	

# FAQ

## 1、当mycat使用第二种方式的时候，将master断电，重启后master的库数据全部消失。

原因： 不太明确。

解决办法：在主从my.cnf中配置两种方式。 当master断电后，主从手动切换。



## 2、MySQL的slave断了怎么办？

解决方案：

mysql> stop slave;

Query OK, 0 rows affected (0.01 sec)

mysql> set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;

Query OK, 0 rows affected (0.00 sec)

mysql> start slave;

Query OK, 0 rows affected (0.01 sec)

解释：

set GLOBAL SQL_SLAVE_SKIP_COUNTER=1; 表示 跳过一个事物

查看从库及其日志，已经开始正常复制

## 3、mysql主从错误断开 怎样恢复

**mysql主从同步常见异常及恢复方法**

1. 一般的异常只需要跳过一步即可恢复

mysql> stop slave;

mysql> set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;

mysql> slave start;

2. 断电导致主从不能同步时，通主库的最后一个bin-log日志进行恢复

在主库服务器上，mysqlbinlog mysql-bin.xxxx > binxxxx.txt

tail -n 100000  binxxxx.txt > tail-binxxxx.txt

vim tail-binxxxx.txt 打开tail-binxxxx.txt文件找到最后一个postion值

然后在从库上，change host to 相应正确的值

mysql> stop slave;

mysql> change master to master_host='ip', master_user='username', master_password='password', master_log_file='mysql-bin.xxxx', master_log_pos=xxxx;

mysql> slave start;

mysql> show slave status\G

 

3. 主键冲突、表已存在等错误代码如1062,1032,1060等，可以在mysql主配置文件my.cnf或my.ini文件指定

[mysqld]

slave-skip-errors = 1062,1032,1060

------

mysql> stop slave;
mysql> set global sql_slave_skip_counter = 1;
mysql> start slave;
同样的操作可能需要进行多次，也可以设置自动处理此类操作，格式：slave-skip-errors = 错误代码

在从服务器的my.cnf里设置：
slave-skip-errors = 1062



## 4、python操作mycat只操作主库

**测试：**

1、将数据库配置成主从赋值，mycat实现读写分离；

2、使用python对数据库进行操作；

3、主从数据库都打开general_log，查看数据库查询日志。



**实验结果：**

使用mysqldb和pymysql库，不管什么操作，mycat都将操作路由到了主库来执行。



**原因：**

​	使用python第三方库，mysqldb和pymysql都会默认开启事务来执行sql语句。也就是常用的执行方法，如下：

```python
cursor.execute(sql_1)
cursor.execute(sql_2)
#至此sql还没有到数据库中执行
cursor.commit()
#commit后所有sql才会生效

```

​	也就是说，操作中包含了一种事务处理的机制。但是，mycat对于显式事务来说，只会路由到主库上执行，所以就会造成，python操作mycat的时候，mycat总是将操作路由到主库。



**解决办法:**

​	mysql方面开启：autocommit。

​	查询方式以及设置如下：

```mysql
select  @@autocommit;
set @@autocommit=1;

```

​	或者修改配置文件:

1. [mysqld]  
2. init_connect='SET autocommit=0' 

​	一下提供三种第三方库操作数据库的处理方式:

**mysqldb**

```python
#在连接数据库后
conn.autocommit(1)
#并且在执行完execute之后，不用进行commit操作。

```

**pymysql**

```python
conn = pymysql.connect(host='192.168.1.2', user='root', password='1',
                      db='test', port=8066, charset='utf8',autocommit=True)
#并且在执行完execute之后，不用进行commit操作。

```

**sqlalchemy**

```python
#连接的时候
mysql+pymysql://user:password@host:port/db?charset=foo&autocommit=true

```

> 以上操作，建议只对select操作进行。







