# python 计算文件的行数和读取某一行内容的实现方法

## 一、计算文件的行数

最简单的办法是把文件读入一个大的列表中,然后统计列表的长度.如果文件的路径是以参数的形式filepath传递的,那么只用一行代码就可以完成我们的需求了:

```
count = len(open(filepath,'rU').readlines())
```

如果是非常大的文件,上面的方法可能很慢,甚至失效.此时,可以使用循环来处理:

```
count = -1
for count, line in enumerate(open(thefilepath, 'rU')):
    pass
count += 1
```

另外一种处理大文件比较快的方法是统计文件中换行符的个数 '\n  '(或者包含 '\n' 的字串,如在 windows 系统中):

```
count = 0
thefile = open(thefilepath, 'rb')
while True:
    buffer = thefile.read(8192*1024)
    if not buffer:
        break
    count += buffer.count('\n')
thefile.close( )
```

参数 'rb' 是必须的,否则在 windows 系统上,上面的代码会非常慢.

linecache 是专门支持读取大文件，而且支持行式读取的函数库。 linecache 预先把文件读入缓存起来，后面如果你访问该文件的话就不再从硬盘读取

## 二、读取文件某一行的内容（测试过 1 G 大小的文件，效率还可以）

```
import linecache
count = linecache.getline(filename,linenum)
```

## 三、用 linecache 读取文件内容（测试过 1 G 大小的文件，效率还可以）

```
str = linecache.getlines(filename)
```

str 为列表形式，每一行为列表中的一个元素

本文出自 “王伟” 博客，请务必保留此出处 <http://wangwei007.blog.51cto.com/68019/1242317>