# python 之 MySQLdb 库的使用

在开发的过程中避免不了和数据库的交互，在实际环境中用的最多的 Mysql 数据库，那 python 是怎么和 Mysql 进行交互的呢，python 使用一个叫 MySQLdb 的库来连接 MySQL，好的，下面最要从 MySQLdb 的安装、连接 MySQL、执行 sql 语句、如何取得结果、关闭数据库连接来讲述一下：

1. MySQLdb 的安装

我使用的是 ubuntu 系统，安装方法为：apt-get install python-MySQLdb，这样当在 python 环境执行 import MySQLdb 不报错就是安

```
root@ubuntu:~# python
Python 2.7.4 (default, Apr 19 2013, 18:32:33)
[GCC 4.7.3] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import MySQLdb
>>>
```

2. 如何连接 Mysql

MySQLdb 提供的 connect 方法用来和数据库建立连接,接收数个参数,返回连接对象，如：

```
conn=MySQLdb.connect(host="localhost",user="root",passwd="sa",db="mytable",port=3306)
```

比较常用的参数包括:

host:数据库主机名.默认是用本地主机.  
user:数据库登陆名.默认是当前用户.  
passwd:数据库登陆的秘密.默认为空.  
db:要使用的数据库名.没有默认值,如果在这里设置了 db,则连接时直接连接到 Mysql 的 db 设置的数据库中  
port:MySQL服务使用的TCP端口.默认是3306.  

注：connect 中的 host、user、passwd 等可以不写，只有在写的时候按照 host、user、passwd、db(可以不写)、port 顺序写就可以，注意端口号 port=3306 还是不要省略的为好，如果没有 db 在 port 前面，直接写 3306 会报错

连接成功后，如需切换该用户的其他数据库，使用以下语句：conn.select_db('mysql')形式切换数据库

```
>>> con=MySQLdb.connect('localhost','root','123456',port=3306)
>>> con.select_db('mysql')
>>> cur=con.cursor()
>>> cur.execute('show tables')
24L
>>> cur.fetchall()
(('columns_priv',), ('db',), ('event',), ('func',), ('general_log',), ('help_category',), ('help_keyword',), ('help_relation',), ('help_topic',), ('host',), ('ndb_binlog_index',), ('plugin',), ('proc',), ('procs_priv',), ('proxies_priv',), ('servers',), ('slow_log',), ('tables_priv',), ('time_zone',), ('time_zone_leap_second',), ('time_zone_name',), ('time_zone_transition',), ('time_zone_transition_type',), ('user',))
```

第 1 行：连接数据库     
第 2 行：选择连接 mysql 这个数据库    
第 3 行以下是获取数据库表，语法后面会讲  

3. 怎么操作数据库，MySQLdb 用游标（指针）cursor 的方式操作数据库

因该模块底层其实是调用 C API 的，所以，需要先得到当前指向数据库的指针

```
>>> cur=con.cursor()
```

4. 数据库的操作和结果显示

我们利用 cursor 提供的方法来进行操作，方法主要是 1.执行命令 2.接收结果

ursor 用来执行命令的方法:  
execute(query, args):执行单条 sql 语句,接收的参数为 sql 语句本身和使用的参数列表,返回值为受影响的行数  
executemany(query, args):执行单挑 sql 语句,但是重复执行参数列表里的参数,返回值为受影响的行数

cursor 用来接收返回值的方法:  
fetchall(self):接收全部的返回结果行.  
fetchmany(size=None):接收 size 条返回结果行.如果 size 的值大于返回的结果行的数量,则会返回 cursor.arraysize 条数据.  
fetchone():返回一条结果行.  
scroll(value, mode='relative'):移动指针到某一行.如果 mode='relative',则表示从当前所在行移动 value 条,如果 mode='absolute',则表示从结果集的第一行移动 value 条.  

先来看一下 execute 的增删改查的操作

```
#创建数据库 51ctotest
>>> cur.execute('create database 51ctotest')
#选择数据库 51ctotest
>>>con.select_db('51ctotest')
#创建表 51cto,id 自增
>>>cur.execute('create table if not exists 51cto(id int(11) PRIMARY KEY AUTO_INCREMENT,name varchar(20),age int(3))')
#插入一行数据,只给 name、age 赋值，让 id 自增
#使用 sql 语句,这里要接收的参数都用 %s 占位符.要注意的是,无论你要插入的数据是什 # 么类型,占位符永远都要用 %s，后面的数值为元组或者列表
>>>cur.execute("insert into 51cto(name,age) values(%s,%s)",('fan',25))
#插入多行数据，用 executemany，它会循环插入后面元组中的所有值
>>> cur.executemany("insert into 51cto(name,age) values(%s,%s)",(('te',25),('fei',26),('musha',25)))
3L
#查询
>>> cur.execute('select * from 51cto')
5L
#我们使用了 fetchall 这个方法.这样,cds 里保存的将会是查询返回的全部结果.每条结果都是一个 tuple 类型的数据,这些 tuple 组成了一个 tuple
>>> cur.fetchall()
((1L, 'fan', 25L), (2L, 'fan', 25L), (3L, 'te', 25L), (4L, 'fei', 26L), (5L, 'musha', 25L))
#再次查询，会看到查询不到结果，因为无论 fetchone、fetchall、fetchmany 指针是会发生移动的。所以，若不重置指针，那么使用 fetchall 的信息将只会包含指针后面的行内容。使用 fetchall 把指针挪到了最后，可以用 scroll 手动把指针挪到一个位置
>>> cur.fetchall()      
()
>>> cur.scroll(1,'absolute')
>>> for i in cur.fetchall():
...     print i         
...
(2L, 'fan', 25L)
(3L, 'te', 25L)
(4L, 'fei', 26L)
(5L, 'musha', 25L)
```

这里有必要说一下 scroll：

```
cur.scroll(int,parm)
```

这里参数含义为：

int：移动的行数，整数；在相对模式下，正数向下移动，负值表示向上移动。  
parm：移动的模式，默认 是 relative，相对模式；可接受 absoulte，绝对模式。  

```
#fetchone一次只取一行，指针下移  fetchmany（size）一次去 size 行
>>> cur.scroll(1,'absolute')
>>> cur.fetchone()
(2L, 'fan', 25L)
>>> cur.fetchmany(2)
((3L, 'te', 25L), (4L, 'fei', 26L))
#普通取出是元组的形式，再从里面取值不好取，那怎么取成字典的格式呢，MySQLdb 中有 DictCursor，要做到这点也很简单，那就是建立数据库连接是传递 cusorclass 参数，或者在获取 Cursor 对象时传递 cusorclass 参数即可
>>> cur = con.cursor(cursorclass=MySQLdb.cursors.DictCursor) 
>>> cur.execute('select * from 51cto')
5L
>>> for i in cur.fetchall():
...     print i
...
{'age': 25L, 'id': 2L, 'name': 'fan'}
{'age': 25L, 'id': 3L, 'name': 'te'}
{'age': 26L, 'id': 4L, 'name': 'fei'}
{'age': 25L, 'id': 5L, 'name': 'musha'}
#更新，习惯 %s 的用法
>>> cur.execute('update 51cto set name=%s where id=3',('Mus'))
>>> cur.scroll(2,'absolute')
>>> cur.fetchone()       
{'age': 25L, 'id': 3L, 'name': 'Mus'}
#在执行完插入或删除或修改操作后,需要调用一下 conn.commit() 方法进行提交.这样,数据才会真正保 存在数据库中
>>> con.commit()
#最后关闭游标，关闭连接
>>> cur.close()
>>> con.close()
```

本文出自 “linux 开源-不断的总结....” 博客，请务必保留此出处 <http://fantefei.blog.51cto.com/2229719/1282443>