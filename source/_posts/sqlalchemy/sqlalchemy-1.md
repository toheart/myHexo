---
title: Sqlalchemy实战入门--建表
date: 2019-06-13 20:17:01
tags:
 - sqlalchemy
 - mysql
 - python

categories:
 - 开发
 - sqlalchemy
---



# Sqlalchemy实战入门--建表

​	现在所有的sqlalchemy入门教程都比较笼统，所以自己写一份教程。



## 安装

```shell
#安装mysql连接包
pip install pymysql
#安装sqlalchemy
pip install sqlalchemy 
```



## 创建orm模型

```python
from sqlalchemy import Column
from sqlalchemy import DateTime
from sqlalchemy import INTEGER
from sqlalchemy import String
from sqlalchemy.sql import func
from sqlalchemy.ext.declarative import declarative_base

# 声明映像，映射基类，用来生成ORM映射
Base = declarative_base()

class User(Base):
    # 写明实际数据库中的映射表名，<必须有>
    __tablename__ = 'user_table'
    # 每个字段都需要使用Column来声明
    id = Column(INTEGER, primary_key=True, autoincrement=True)
    name = Column(name='username', type_=String(length=32), nullable=False)
    age = Column(INTEGER, default=20)
    insert_time = Column(DateTime, server_default=func.now())
```

解释其中字段名称：

* Column：表示列字段名，其中需要传递参数如下：

  * name:  数据库表字段名称；

  * type_:   数据库类型，其中常用以下几个：

    * String --> varchar()，其中可以指定length大小，如：

      String(255) --> varchar(255)

    * Integer --> int(11) ， 整型

    * Datetime --> datetime 

    * TEXT ..等等

  * primary_key: 是否为主键；

  * autoincrement: True，为自增，在插入时不用赋值；

  * nullable： 是否可以为空

    * False: 不能为空
    * True: 为空

  * server_default： 数据库端的默认值：

     server_default=func.now() 也就是说 插入时，在数据库执行时使用 insert into (insert_time) values(now())

  * defalut : 在程序端给其赋值为 0 

以上，我们的一张映射表创建成功；



## 使用sqlalchemy查看生成表语句

```python
>>>from sqlalchemy.schema import CreateTable
>>>print(CreateTable(User.__table__)）
CREATE TABLE user_table (
	id INTEGER NOT NULL, 
	username VARCHAR(32) NOT NULL, 
	age INTEGER, 
	insert_time DATETIME DEFAULT now(), 
	PRIMARY KEY (id)
)
```

​	sqlachemy.schema中存在很多对Model视图的操作，这里我们可以用其Createtable ，就可以看下当前表的创建语句了。



## 审查当前模型的参数

​	在我们查询（之后讲）的时候，数据库返回的模型实例对象，那么，当我们在进行API编写的时候非常不方便，如：使用Flask的话，我们返回json字符串，那么我们需要将当前模型转化成dict，然后调用jsonify，这样很不方便，这里便提供两种方式：

​	**使用sqlchemy的inspect**

```python
from sqlalchemy import inspect 
insp = inspect(User)
print(insp.all_orm_descriptors.keys())
#输出：
#['id', 'insert_time', '__mapper__', 'name', 'age']
```

### **提供一个to_dict函数**

```python
def to_dict(cls):
    obj_dict = {}
    if not isinstance(cls, Base):
        print('false')
        return obj_dict
    insp = inspect(User)
    for item in insp.all_orm_descriptors.keys():
        if item != '__mapper__':
            obj_dict[item] = getattr(cls, item)
    return obj_dict
        
user = User(name='aaaa', age=12)
print(to_dict(user))
# 输出
# {'id': None, 'insert_time': None, 'name': 'aaaa', 'age': 12}
```



### 提供一个DictMixin

```python
class DictMixin(object):

    def _set_object(self, data):
        if not isinstance(data, dict):
            return None
        for c in self.__table__.columns:
            setattr(self, c, data.get(c))

    def to_dict(self):
        return {c.name: getattr(self, c.name, None)
                for c in self.__table__.columns}
```



## 创建所有表

### 数据库建立连接

​	使用sqlalchemy连接数据库，我们可以通过需要以下几步：

```python
from sqlalchemy.engine import create_engine
from sqlalchemy.orm import sessionmaker

# 第一步创建连接引擎
engine = create_engine(f'mysql+pymysql://{root}:{password}@{host}:{port}/{database}')
# 第二步创建session工厂函数
sess = sessionmaker(engine)
# 第三步创建本地session进行使用
session = sess()
```

### 使用engine进行原始sql的执行

**第一种**：直接使用sql语句

```python
conn = engine.connect()
rs = conn.execute('select 1')
data = rs.fetchone()[0]
print(data)
```



**第二种**：配合text查询

```python
from sqlalchemy import text
t = text('select * from test0 where t_id=:t_id')
result = conn.execute(t, t_id='aaaaa')
data = result.fetchone()
print(data)
```





## 注意

### sqlalchemy的数据库池的管理

​	默认创建create_engine的时候，其以下几个是默认值管理数据库连接池：

* max_overflow = 10,  连接池的上限，最大超过pool_size多少个；
* pool_size = 5, 数据库连接池的大小；

> 所以数据库连接池最大连接数等于: 15

* pool_timeout = 30,    池中没有线程最多等待的时间，否则报错;
*  pool_recycle=-1,   多长时间对进程池中的线程回收一次；





以上。



