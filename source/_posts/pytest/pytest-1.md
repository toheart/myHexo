---
title: pytest测试入门--简单使用
date: 2019-06-14 15:57:08
tags:
 - 测试
 - pytest
 - python

categories:
 - 测试
 - pytest
---

# pytest测试入门--简单使用

## 安装

```shell
pip install pytest
```



## 初步使用

```shell
mkdir pytest_demo
cd pytest_demo
vim test_1.py
cat test_1.py
```

测试使用到的代码如下：

```python
import pytest

def test_1():
    assert 1 == 1

@pytest.mark.hello
def test_hello():
    strinf = "hello world"
    assert strinf == "hello world"

@pytest.mark.hello
def test_exception():
    assert 1 != 2

def test_error():
    assert 1 == 2
```



**执行命令**

```shell
pytest
```

**执行结果**

![test_1](/assets/picture/pytest/test_1.png)



**为什么什么参数都不加，但是确实将代码执行成功了？**

答： 这里就需要引入pytest搜索路径了，标准搜索规则如下：

* 从一个或者多个目录开始查找，可以在命令行指定文件名或目录名，如果没有指定则在当前目录下查找；
* 在该目录和所有子目录下递归查找测试模块；
* 测试模块指：文件名为test_*.py 或  *\_test.py;
* 在测试模块中查找以test_*的函数名；
* 查找名字以Test开头的类，首先筛选掉包含\__init__函数的类，再查找勒种以test\_*开头的类方法；

以上。

当然可以在pytest.ini中设置。（后续）



## pytest常用命令

```shell
# 获取命令行帮助
pytest --help
```

### --collect-only选项

​	只会展示哪些测试用例会被运行；

![collect_only](/assets/picture/pytest/collect_onnly.png)



### -k 选项

​	-k 选项可以使用表达式来获取希望执行的用例。

![_k](/assets/picture/pytest/k.png)	



### -m 选项

​	在pytest中，我们可以使用marker来对测试进行分组，-m选项则是快速选中分组并运行。

![choice_m](/assets/picture/pytest/choice_m.png)

​	下面警告是需要在pytest.ini中注册marker。



### -v 选项

 	输出结果会更加详细，如：

![choice_v](/assets/picture/pytest/choice_v.png)



### -s 与 --capture=\<method\>

​	-s 等价于 --capture=no， 允许终端在测试运行时输出结果；

​	即，程序中的print()语句，可以使用-s选项来捕获输出；

> method:
>
> ​	no: 如上；
>
> ​	fd: fd为文件描述符，如1,2，则会被输出至临时文件；
>
> ​	sys： 将stdout/stderr输出至内存。



### --tb=\<style\> 选项

​	决定捕捉到失败时输出信息的显示方式：

> style:
>
> ​	short: 仅输出一行以及系统判定内容；
>
> ​	line： 只使用一行输出所有错误信息；
>
> ​	no: 直接屏蔽全部回溯信息；

**short**

![tb_short](/assets/picture/pytest/tb_short.png)

**line**	

![tb_line](/assets/picture/pytest/tb_line.png)

**no**

![tb_no](/assets/picture/pytest/tb_no.png)



### --duration=\<N\> 选项

​	加快测试节奏，统计测试过程中那几个阶段是最慢的，包括每个测试的call, setup, teardown过程；

建议使用：

> pytest --duration=0 -vv

-vv,能够显示详细信息。 0则是将所有阶段耗时从长到短排序；

![duration](/assets/picture/pytest/duration.png)

以上。

接下来则介绍pytest中setup,teardown,以及fixture的使用；