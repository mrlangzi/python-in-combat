# Python FAQ3-python 中 的原始(raw)字符串

本篇源自 py2.7.9-docs 的 faq.pdf 中的“3.23 Why can’t raw strings (r-strings) end with a backslash?”  

更准确的说，原始字符串即以r修饰的字符串，不能以奇数个反斜杠结束；

原始字符串被设计用来作为一些处理器(主要是正则表达式引擎)的输入。这种处理器会认为这种未匹配的末端反斜杠是种错误，所以，原始字符串也就不允许以奇数个反斜杠结束。反过来，他们允许你使用斜杠来表示转义，包括\"表示"，\t 表示 TAB 等。当原始字符串用于这些处理器时，这个规则适用。

如果原始字符串不用于正则表达式等处理器，只是简单的代表一个字符串，那么该串中的 \ 就是 \,而不再具有转义的含义，这就是所谓的‘原始’。

下面我会一步步的解释字符串和原始字符串的区别

1.用于单独的字符串表示:

简单字符串中存在 \ 转义行为，而原始字符串中 \n 就是 \n 字符

```
>>> s = "i have\na dream"
>>> r = r'i have\na dream'
>>> print s
i have
a dream
>>> print r
i have\na dream
```

2.原始字符串用于正则表达式中

我们使用 windows 路径来做例子介绍原始字符串的一次转义

```
>>> path = r"\this\is\a\path\"
  File "<stdin>", line 1
    path = r"\this\is\a\path\"    #原始字符串不允许单数个\结尾，不管是用于正则还是普通字串
                             ^
SyntaxError: EOL while scanning string literal
>>> path = r"\this\is\a\path\ "[:-1] 
>>> path
'\\this\\is\\a\\path\\'        #定义了一个待匹配的字符串
>>> reg1 = r'\\this\\is\\a\\path\\' #定义了自然字符串表示的正则表达式
>>> import re
>>> g = re.match(reg1, path)    #使用自然字符串进行匹配
>>> print g.group()
\this\is\a\path\               #匹配到了结果，表示真实的\字符可以被自然字符串以\\匹配上
>>>                            #\\转义的结果就是\
```

3.简单字符串用于正则表达式中

让我们使用上面的 path 变量来制作简单字符串用来匹配的例子

```
>>> reg2 = '\\this\\is\\a\\path\\'
>>> g = re.match(reg2, path)         #竟然报异常了，根据异常的意思是行尾是虚假的转义
Traceback (most recent call last):  #下面我们再探究原因，先把行尾的\\去掉，再次进行匹配
  File "<stdin>", line 1, in <module>
  File "D:\Python27\lib\re.py", line 137, in match
    return _compile(pattern, flags).match(string)
  File "D:\Python27\lib\re.py", line 244, in _compile
    raise error, v # invalid expression
sre_constants.error: bogus escape (end of line)
 
>>> reg2 = '\\this\\is\\a\\path'    
>>> g = re.match(reg, path)         #按照原始字符串的理解，这里应该可以匹配上的，但是没有
>>> print g.group()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'NoneType' object has no attribute 'group'
```

为什么会出现差异，又为什么到处都建议正则匹配时要使用r'字符串'呢？

让我们分析下原始字符串和简单字符串的区别：简单字符串如果想输出‘\’,需要进行转义即'\\'才会输出一个'\'；那原始字符串想要输出'\'，则直接写即可'\'。

这里有些乱，我觉得主要在于 str、repr 在捣乱：

```
>>> print path                     #这里调用str，人们习惯的显示方式
\this\is\a\path\
>>> path                           #这里调用repr，真实的显示方式(比str的显示仅多了一层转义)
'\\this\\is\\a\\path\\'
```

让我们全部将真实的显示方式当做参照物，即
  
path 的真实显示是：'\\this\\is\\a\\path\\'    
简单字符串的正则表达式 reg2 的真实显示是：'\\this\\is\\a\\path'  
原始字符串的正则表达式 reg1 的真实显示是：'\\\\this\\\\is\\\\a\\\\path\\\\'  
从真实的显示来看匹配就容易理解的多了，而且没有了原始和简单字符串之分，都看做是正则引擎应用的串。从上面可以看出 reg2中\\ 只能匹配\，而 path 中是 \\，需要像 reg1 中的 \\\\ 来进行匹配。  
    
追根溯源向来比较绕，还是简单记住使用规则，匹配路径 \ 字符，需要普通字符串输入 4 个斜杠(\\\\)匹配上，而原始字符串仅需要 2 个斜杠(\\)即可匹配上。这也是鼓励使用原始字符串进行正则匹配的原因。
    



本文出自 “无名” 博客，请务必保留此出处 <http://xdzw608.blog.51cto.com/4812210/1607504>