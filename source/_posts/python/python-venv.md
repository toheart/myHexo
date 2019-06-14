---
title: python3快速开始
date: 2019-06-14 18:40:42
tags:
 - python
 - venv
categories:
 - 开发
 - python3
---



# python3快速开始

## 安装

```shell
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz
tar -zxvf Python-3.7.3
./configure --prefix=/usr/local/python --enable-optimizations
make && make install 
```



## 加入环境变量

​	我比较喜欢暴力加入，直接加入/etc/profile中

```shell
echo "PATH=$PATH:/usr/local/python/bin" >> /etc/profile
source /etc/profile
# 检查
python --version
```



## 虚拟环境的搭建

​	从python3起，我们便可以直接使用python自带的venv库，来创建venv环境；

```shell
mkdir venv
cd venv
# -m 指定运行模块
# venv 虚拟环境模块
# 虚拟环境名称
python3 -m venv learn
```

**虚拟环境操作**

```shell
# 直接进入虚拟环境
[root@tly ~] sh ./venv/learn/bin/activate
# 前面带括号则进入了虚拟环境了
(learn) [root@tly ~] 
#退出虚拟环境
deactivate
```



> 如果每次进入项目模块后都要执行
>
> ​	sh ./venv/learn/bin/activate
>
> 进入虚拟环境太过于麻烦，所以我们接下来介绍一个神器

## autoenv的使用

### 安装

​	 **可以使用python安装**

```shell
pip install autoenv
echo "source `which activate.sh`" >> ~/.bashrc
```

​	 **也可以使用git安装**

```shell
git clone git://github.com/kennethreitz/autoenv.git ~/.autoenv
echo 'source ~/.autoenv/activate.sh' >> ~/.bashrc
```

 	**也可以下载安装**

```shell
wget https://github.com/kennethreitz/autoenv/archive/master.zip
unzip master.zip
echo 'source activate.sh' >> ~/.bashrc
```

### 使用

使用很简单，分以下几步：

* 进入项目目录

```shell
cd test
```

* 编写.env文件

```shell
echo 'source /root/venv/learn/bin/activate' >> .env
```

* 退出目录，重启进入，然后就看见一大段输出，选y

以上，完毕；

> 建议，将PYTHONPATH=/path/to/project也加入到.env中，
>
> 并且将项目所需要的环境变量也加入到.env中；
>
> 这样我们每次进入项目时，就可以直接运行命令了。









