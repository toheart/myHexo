---
title: centos上安装python环境
date: 2019-06-13 20:47:16
tags:
 - centos
 - python
 - mysql
 - nginx

categories:
 - 运维
 - centos
---



## 1、安装python-pip

​	首先安装epel扩展源：

​		yum -y install epel-release

​	更新完成之后，安装pip:

​		yum -y install python-pip

## 2、安装python依赖包：

​	初次直接运行：

​		pip install -r requirement.list

​	如果是裸的centos绝对会报错！

​	然后开始漫长的改错安装。

```
### 错误一：EnvironmentError: mysql_config not found
```

​	原因：缺少mysql驱动导致，所以加上mysql就行

​	`yum -y install mysql-devel`



### 错误二：error: command 'gcc' failed with exit status 1

​	原因：没有gcc命令（c语言编译器），没有 那就安就行了

​	`yum -y install gcc`

​	但是还是会再次报错：然后 我们需要安装下 

​	`yum -y install python-devel`

以上，就安装完了依赖包。



## 3、安装gunicorn

​	没有别的就一个：

​	`pip install gunicorn`



## 4、安装supervisor 

​	安装命令

​	`easy_install supervisor`

​	验证是否成功：echo_supervisord_conf

​	然后mkdir /etc/supervisor

​		echo_supervisord_conf > /etc/supervisor/supervisord.conf	

​	现在有配置文件还是不够，我们需要扩展，所以

  		mkdir   /etc/supervisor/config.d 

​	修改/etc/supervisor/supervisord.conf的最下面的一行include

​		files = /etc/supervisor/config.d/*.conf



​	最基本的配置：

```ini
[program:tomcat]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run
directory=xxxx
autostart=true
autorestart=true
startsecs=5
priority=1
stopasgroup=true
killasgroup=true
```



## 5、安装nginx

​	安装各种依赖项：

### 1、查看内核版本，看看是否高于2.6。#2.6版本以上内核才支持epoll

### 2、安装GCC编译器

`yum -y install gcc`

### 3、安装C++编译器

`yum -y install gcc-c++`

### 4、安装PCRE库  为了支持正则表达式

`yum install -y pcre pcre-devel`

#### 5、安装zlib库

`yum install -y zlib zlib-devel`

### 6、安装OpenSSL

`yum install -y openssl openssl-devel`

### 7、安装nginx

`yum -y install nginx `

以上依赖环境全部安装完成。



全部都安装的命令：

```shell
yum -y install gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel nginx
```



## 6、设置系统时间

下载工具：ntp

```shell 
yum -y install ntp
ntpdate -u asia.pool.ntp.org
rm -rf /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```



## 7、设置supervisor为开机自启动

1、vim /lib/systemd/system/supervisord.service 

2、

```vim
[Unit]
Description=Process Monitoring and Control Daemon
After=rc-local.service



[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
ExecReload=supervisorctl reload
sysVStartPriority=99



[Install]
WantedBy=multi-user.target
```



3、systemctl enable supervisord.service 



## 8、mysql远程授权访问

```shell
 GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "password";
 flush privileges;

 
[mysqld] 
skip_name_resolve 

```





## 9、locale 设置为UTF-8

1. locale -a  查看当前安装的编码
2. 如果没有 则安装

```shell
yum -y install kde-l10n-Chinese telnet &&    yum -y reinstall glibc-common &&    yum clean all  &&       localedef -c -f UTF-8 -i zh_CN zh_CN.utf8

```

3. 重新docker或者物理机
4. vi /etc/profile

```shell
export LC_ALL='zh_CN.utf-8'
```

5. 确认

```shell
python -c "import sys; print(sys.getfilesystemencoding())"

```



## 10、源码安装java

1. 下载jdk.tar.gz
2. 解压

```shell
tar zxvf jdk-8u151-linux-x64.tar.gz
mv jdk1.8.0_151/ /usr/local/java/

```

3. 配置环境变量

```shell
vi /etc/profile #在文件最后加入以下几行
	export JAVA_HOME=/usr/local/java
	export JRE_HOME=${JAVA_HOME}/jre
	export CLASS_PATH=.:$JAVA_HOME/lib/dt,jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	export PATH=$PATH:$JAVA_HOME/bin:%JRE_HOME/bin
source /etc/profile #重新加载配置

```

4. 验证

```shell 
java -version

```



## 11、安装mysql

1.卸载  先停掉mysql进程   没有安装过的可以直接跳过

​        pkill -9 mysqld

​       	rpm -qa|grep -i mysql

​      用命令 yum -y remove

​      	yum -y remove mysql-community-client-5.6.38-2.el7.x86_64

​      卸载不掉的用 rpm -ev 

​      依次卸载 直到没有

2.下载mysql的repo源 这个安装的mysql5.7.20  /**纠正一下，这源下载的是最新的版本  ****/

   [root@localhost ~]# **cd /usr/local/src/**
   [root@localhost src]# **wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm** 

   [root@localhost src]# **rpm -ivh mysql57-community-release-el7-8.noarch.rpm** 

   [root@localhost src]#  **yum -y install mysql-server** 

（**也可以指定安装目录**     **yum** **--installroot=/usr/local/mysql --releasever=/** **-y install mysql-server**  ）我没试，这样装环境变量配置都不用你管，装上直接启动就行。安装路径是默认的。

   一路 y 

根据步骤安装就可以了，

默认配置文件路径： 
配置文件：/etc/my.cnf 
日志文件：/var/log/var/log/mysqld.log 
服务启动脚本：/usr/lib/systemd/system/mysqld.service 
socket文件：/var/run/mysqld/mysqld.pid

  配置  my.cnf        vim /etc/my.cnf

```
[mysqld]
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.

# join_buffer_size = 128M

# sort_buffer_size = 2M

# read_rnd_buffer_size = 2M

datadir=/var/lib/mysql

socket=/var/lib/mysql/mysql.sock

server_id = 1

expire_logs_days = 3

# Disabling symbolic-links is recommended to prevent assorted security risks

symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

```

 

不过安装完成后，密码为随机密码，需要重置密码。

**4.**  启动mysql服务

service mysqld restart

 重置密码

​    **[root@localhost ~]#** **grep "password" /var/log/mysqld.log**    

​    

​    可以看到  输入 mysql -u root -p   密码 进入      第一次登陆 ，需要重置密码 要不什么也不能操作 

​       接下来重置密码：5.7.20 为了安全密码           必须包含 数字字母符号

​      踩过的坑啊，设置了好几次。还有这ip不能是% 不知道为什么  反正第一次设置成%没成功  登上去之后再改就可以改了。

​       把密码改简单的方法 http://blog.csdn.net/z13615480737/article/details/78907697

​      alter user 'root'@'localhost' identified by 'Root!!2018';  

​     最后记得刷新权限；

​     flush privileges 

​    也可以 直接再添加新用户     

​    CREATE USER ‘root‘@‘%‘ IDENTIFIED BY ‘您的密码‘;

​    grant all on *.* to 'root001'@'%' identified by 'Root@@'  with grant option;

   增加root用户指定可以任意IP登录，如果想限制只能让指定IP登录请把%替换成IP地址

   问题：如果发现找不到密码！！！！！

   解决:只能通过忘记密码的方式修改密码!!! 在安装的过程中发现找不到密码？？？折腾了好长时间 通过修改密码找回之后发现、原来之前安装的数据库在了，就没有生产新的数据库！！用的还是之前的配置。

2.看mysql启动了没？初始化数据库了没？  一般直接启动 数据库 就可以 **用****grep "password" /var/log/mysqld.log**    看到随机密码了

修改MySQL的登录设置：

\#vi /etc/my.cnf

在[mysqld]的段中加上一句：skip-grant-tables 保存并且退出vi。

重新启动mysqld

重新启动mysqld
\#/etc/init.d/mysqld restart ( service mysqld restart )
use mysql 
update user set password=password("12345") where user="root";
mysql 5.7的数据库没有了password字段 用的是authentication_string字段
mysql> update mysql.user set authentication_string=password('root') where user='root' ;
flush privileges;
修改密码之后在改回来



# FAQ

## 1. yum ImportError:No module named sqlitecachec

解决方法：

查看你安装的

python-iniparse-*.e*.noarch.rpm

yum-*.centos.0.1.noarch.rpm

yum-metadata-parser-*.x86_64.rpm

yum-plugin-fastestmirror-*.noarch.rpm

```shell
rpm -ivh --nodeps  xxx.rpm

```

## 2. nginx 代理 tomcat 502 bad gateway

解决办法：

```shell
 /usr/sbin/setsebool -P httpd_can_network_connect true 

```

[详情查看](https://stackoverflow.com/questions/25235453/nginx-proxy-server-localhost-permission-denied)









 