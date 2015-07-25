# python simplejson 模块浅谈

一、背景知识

- JSON:

引用百科描述如下，具体请自行搜索相关介绍：

JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。它基于 JavaScript（Standard ECMA-262 3rd Edition - December 1999）的一个子集。 JSON 采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括 C, C++, C#, Java, JavaScript, Perl, Python 等）。这些特性使 JSON 成为理想的数据交换语言。易于人阅读和编写，同时也易于机器解析和生成(网络传输速度快)。

表示方法：

- 数据在名称/值对中
- 数据由逗号分隔
- 花括号保存对象
- 方括号保存数组

示例：

```
{"programmers":[
{"firstName":"Brett","lastName":"McLaughlin","email":"aaaa"},
{"firstName":"Jason","lastName":"Hunter","email":"bbbb"},
{"firstName":"Elliotte","lastName":"Harold","email":"cccc"}
],
"authors":[
{"firstName":"Isaac","lastName":"Asimov","genre":"sciencefiction"},
{"firstName":"Tad","lastName":"Williams","genre":"fantasy"},
{"firstName":"Frank","lastName":"Peretti","genre":"christianfiction"}
]}
```

- HOWTO-UNICODE:

unicode 标准描述了字符如何对应编码点(code point),使用 16 进制表示 00 00.  
PYTHON 中，basestring 派生了 unicode 类型和 str 类型  
unicode 字符串是一个编码点序列，该序列在内存中会被表示成一组字节(0-255)，str 是指 8 字节流。  
unicode 字符串可以通过 encode 函数转换为 str；str 可以通过 decode 转换为 unicode。编解码类型一般是 utf-8  

示例：

```
>>> u"中国".encode('utf-8')
'\xe4\xb8\xad\xe5\x9b\xbd'    #将 unicode 字符串编码为 str
>>> '\xe4\xb8\xad\xe5\x9b\xbd'.decode('utf-8')
u'\u4e2d\u56fd'               #将 str 解码为 unicode 字符串
```

从文件中读和写入文件的操作都应该是操作的 8 位字节流，如果将 unicode 字符串写入文件，需要进行编码操作；如果从文件中读 unicode 字符串，首先读取出来的是 8 位字节流需要进行解码操作。

一般功能代码中都直接操作 unicode 字符串，而只在写数据或读数据时添加对应的编解码操作。

- 序列化和反序列化

当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为对象。

**把对象转换为字节序列的过程称为对象的序列化，比如把一个字典对象以某种格式(JSON)写到文件中；把字节序列恢复为对象的过程称为对象的反序列化，比如读取某种格式化(JSON)的文件，构造一个字典对象。**

根据 HOWTO-UNICODE 的知识，把网络可以看做是一个文件，发送方写数据到网络时需要进行编码，接收方读取数据时需要进行解码。也就是说**序列化的同时会进行编码，反序列化的同时会进行解码**。
    
二、simplejson

simplejson 是 json 标准模块的扩展（基础功能相同），是 pypi 提供的拓展模块，需要另行安装。不过可以使用 python 自带的 json 库，基本是相同的使用方法(提供的接口功能基本一致)。在 python 的 library 文档中将 JSON 归为网络数据控制类，很好的说明了他们的用途，主要用于网络数据控制，编解码等。但是也具有其他的用途，比如可以用来作为配置文件的读写模块，简单的文件操作等。

它提供的接口很少，容易掌握，而且大多数情况下会使用默认的参数。官方文档中阐明，默认的接口参数和不进行子类化会有更好的性能体现。下面我们对提供的接口进行讨论，并且仅展示必须参数，其他关键字参数将以**kwargs表示；

- simplejson.dump(obj, fp, **kwargs):将 python 对象写到文件中（以 JSON 格式）
- simplejson.dumps(obj, **kwargs)：将 python 对象表示成字符串(JSON 的格式)
- simplejson.load(fp, **kwargs)：从文件中(包含 JSON 结构)读取为 python 对象
- simplejson.loads(s, **kwargs)：从字符串中(包含 JSON 结构)读取为 python 对象
- class simplejson.JSONDecoder：load/loads 的时候调用，将 JSON 格式序列解码为 python 对象
- class simplejson.JSONEncoder：dump/dumps 的时候调用，将 python 对象编码为 JSON 格式序列

联系到上面的基础知识，我们可以知道，**dump 的过程其实就是向文件句柄中写数据，即对象序列化的过程，需要进行编码，只是编码的格式不只是 unicode 和 str 的转换，而是更重要的 python 对象类型和 JSON 对象类型之间的转换。同理，load 的过程其实就是从文件句柄中读数据，即反序列化生成对象的过程，需要进行解码，只是解码的格式不只是 str 和 unicode 的转换，而是更重要的 JSON 对象类型和 python 对象类型之间的转换。**

下面是 JSON 对象类型和 Python 对象类型之间的对应关系：

|JSON	|Python 2	|Python 3|
|:------|:----------|:-------|
|object	|dict	|dict|
|array|	list|	list|
|string|	unicode|	str|
|number (int)|	int, long|	int|
|number (real)	|float|	float|
|true	|True|	True|
|false|	False|	False|
|null	|None|	None|

下面以一个例子来结束本文，例子中附带注释：

```
#coding:utf-8
import simplejson as json
 
#simplejson.dump(**kwargs)
fp = open('./text.json', 'w+')
json.dump([1,2], fp)         ##将 python 数组进行序列化，保存到文件中
fp.seek(0)
print "----dump----\n", u'使用 dump 将 python 数组对象保存在一个包含 JSON 格式的文件中，文件内容为：\n', fp.read()
print 
fp.close()         
 
#simplejson.dumps(**kwargs)
r_dumps = json.dumps({"中国 obj":[1,2], "obj2":[3,4]})  #将 python 字典进行序列化，保存到字符串中
print "----dumps----\n", u'使用 dumps 将 python 字典对象转换为一个包含 JSON 格式的字符串，字符串结果为：\n', r_dumps
print
 
#simplejson.load(**kwargs)
#如果 json 文档格式有错误，将会抛出 JSONDecoderError 异常
fp = open('./text.json', 'r')
r_load = json.load(fp)           #将文件中的内容转换为 python 对象
print "----load----\n", u"使用 load 读取一个包含 JSON 数组格式的文件后，得到一个 python 对象，类型是：", type(r_load)
print 
#simplejson.loads(**kwargs)
#如果 json 文档格式有错误，将会抛出 JSONDecoderError 异常
 
#将字符串中的内容转换为一个 python 对象
r_loads = json.loads('''{"programmers":[
{"firstName":"Brett","lastName":"McLaughlin","email":"aaaa"},
{"firstName":"Jason","lastName":"Hunter","email":"bbbb"},
{"firstName":"Elliotte","lastName":"Harold","email":"cccc"}
],
"authors":[
{"firstName":"Isaac","lastName":"Asimov","genre":"sciencefiction"},
{"firstName":"Tad","lastName":"Williams","genre":"fantasy"},
{"firstName":"Frank","lastName":"Peretti","genre":"christianfiction"}
]}''')
print "----loads----\n", u"使用 loads 读取一个包含 JSON 字典格式的字符串后，得到一个 python 对象，类型是：", type(r_loads)
print
```

运行之后的结果显示：

```
----dump----
使用 dump 将 python 数组对象保存在一个包含 JSON 格式的文件中，文件内容为：
[1, 2]
----dumps----
使用 dumps 将 python 字典对象转换为一个包含 JSON 格式的字符串，字符串结果为：
{"obj2": [3, 4], "\u4e2d\u56fdobj": [1, 2]}
----load----
使用 load 读取一个包含 JSON 数组格式的文件后，得到一个 python 对象，类型是： <type 'list'>
----loads----
使用 loads 读取一个包含 JSON 字典格式的字符串后，得到一个 python 对象，类型是： <type 'dict'>
```

本文出自 “无名” 博客，请务必保留此出处 <http://xdzw608.blog.51cto.com/4812210/1612389>