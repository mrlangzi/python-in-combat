# python linecache 模块读取文件用法详解

linecache 模块允许从任何文件里得到任何的行，并且使用缓存进行优化，常见的情况是从单个文件读取多行。

linecache.getlines(filename)  
从名为 filename 的文件中得到全部内容，输出为列表格式，以文件每行为列表中的一个元素,并以 linenum-1 为元素在列表中的位置存储

linecache.getline(filename,lineno)  
从名为 filename 的文件中得到第 lineno 行。这个函数从不会抛出一个异常–产生错误时它将返回”（换行符将包含在找到的行里）。  
如果文件没有找到，这个函数将会在 sys.path 搜索。

linecache.clearcache()  
清除缓存。如果你不再需要先前从 getline() 中得到的行

linecache.checkcache(filename)  
检查缓存的有效性。如果在缓存中的文件在硬盘上发生了变化，并且你需要更新版本，使用这个函数。如果省略 filename，将检查缓存里的所有条目。

linecache.updatecache(filename)  
更新文件名为 filename 的缓存。如果 filename 文件更新了，使用这个函数可以更新 linecache.getlines(filename)返回的列表。

用法举例：

```
# cat a.txt
1a
2b
3c
4d
5e
6f
7g
```

1、获取 a.txt 文件的内容

```
>>> a=linecache.getlines('a.txt')
>>> a
['1a\n', '2b\n', '3c\n', '4d\n', '5e\n', '6f\n', '7g\n']
```

2、获取 a.txt 文件中第 1-4 行的内容

```
>>> a=linecache.getlines('a.txt')[0:4]
>>> a
['1a\n', '2b\n', '3c\n', '4d\n']
```

3、获取 a.txt 文件中第4行的内容

```
>>> a=linecache.getline('a.txt',4)
>>> a
'4d\n'
```

注意：使用 linecache.getlines('a.txt') 打开文件的内容之后，如果 a.txt 文件发生了改变，如你再次用 linecache.getlines 获取的内容，不是文件的最新内容，还是之前的内容，此时有两种方法：

1. 使用 linecache.checkcache(filename) 来更新文件在硬盘上的缓存，然后在执行 linecache.getlines('a.txt') 就可以获取到 a.txt 的最新内容；
2. 直接使用 linecache.updatecache('a.txt')，即可获取最新的 a.txt 的最新内容

另：读取文件之后你不需要使用文件的缓存时需要在最后清理一下缓存，使 linecache.clearcache() 清理缓存，释放缓存。

这个模块是使用内存来缓存你的文件内容，所以需要耗费内存，打开文件的大小和打开速度和你的内存大小有关系。

本文出自 “王伟” 博客，请务必保留此出处 <http://wangwei007.blog.51cto.com/68019/1246214>