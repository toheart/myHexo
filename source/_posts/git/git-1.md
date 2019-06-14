---
title: git相关命令
date: 2019-06-13 20:43:43
tags:
 - git

categories:
 - 开发
 - git
---

## git基础命令

### 创建git仓库

> git init

### 将修改文件加入暂存区

> git add xxx

### 查看当前git状态

> git status    
>
> git  status -s   #查看状态情况  

### 如何忽略修改文件

> vim .gitignore  #加入到文件中的会被忽略

### 查看文件的差异

> git diff  #查看当前文件与暂存区文件的差异
>
> git diff --cached  或者 git diff --staged #查看已暂存和上次提交文件的区别

### 提交更新

> git commit  -m  "your message"
>
> git commit -a  #将所有已经跟踪过的文件暂存起来并全部提交 

### 删除文件

> git rm  --cached 

### 移动或重命名文件

> git mv old_file new_file



## 查看日志

### 基本

> git log   #查看所有更新

### 查看每次提交的内容差异

> git log -p -2  #-num 表示最近num次提交的更新

### 查看简略统计信息

> git log --stat   

### log输出格式控制

> git log --pretty=xxxx #常用的有format, oneline
>
> git log --graph #输出图形



## 撤销操作

### 如果暂存区提交后，有几个文件忘记提交了，可以如下操作：

`git commit -m 'initial commit'`

`git add forgotten_file`

`git commit -amend`

如上操作只会提交一次，仅提交第二次结果

### 取消暂存的文件

`git reset HEAD xxxx`

### 撤销对文件的修改

`git checkout -- filename`



## 远程仓库的使用

### 查看远程仓库

`git remote -v`

### 添加远程仓库

`git remote add shortname url`

### 从远程仓库中抓取和拉取

`git fetch [remote-name]`

> 执行完命令，将会拥有远程仓库中所有分支的引用。

### 推送到远程仓库

`git push origin master`

### 查看远程仓库

`git remote show origin`





## **git分支**

### 新建并切换分支

`git checkout -b iss53`



## git的骚操作

### 1、clone下载git下的单个文件夹

​	步骤：

```shell
git  init  #创建git仓库
git config --global core.sparsecheckout true	#设置允许clone的子目录
echo "xxxx" >> .git/info/sparse-checkout 	#设置要克隆的仓库的目录   xxx表示目录名称
git  remote add origin git@xxxxx.git			#添加远程
git pull origin master						#下载
```

### 2、上传后换行符切换的问题



```shell
git  config --global  core.autocrlf input   #将提交上去的代码转成LF换行符
##因为我们部署机器为linux ， 强烈建议设置。
```



### 3、新文件 git add 之后不小心 git rm -f 了

```shell
 git fsck --lost-found 
```



