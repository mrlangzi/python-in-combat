# python Howto 之 logging 模块

本文来源于对 py2.7.9 docs 中 howto-logging 部分加之源代码的理解。官方文档链接如下，我用的是下载的 pdf 版本，应该是一致的：<https://docs.python.org/2/howto/logging.html>

我们不按照文档上由浅入深的讲解顺序，因为就这么点东西不至于有“入”这个动作。

使用 logging 模块记录日志涉及四个主要类，使用官方文档中的概括最为合适：

logger提供了应用程序可以直接使用的接口；  
handler将(logger创建的)日志记录发送到合适的目的输出；  
filter提供了细度设备来决定输出哪条日志记录；  
formatter决定日志记录的最终输出格式。 
    
写 log 的一般顺序为：

一、创建logger：

我们不要通过 logging.Logger 来直接实例化得到 logger，而是需要通过 logging.getLogger("name")来生成 logger 对象。

不是说我们不能实现 Logger 的实例化，而是我们期待的是同一个 name 得到的是同一个 logger，这样多模块之间可以共同使用同一个 logger，getLogger 正是这样的解决方案，它内部使用 loggerDict 字典来维护，可以保证相同的名字作为 key 会得到同一个 logger 对象。我们可以通过实例来验证一下：

```
#test_logger1.py
#coding:utf-8
 
import logging
print logging.getLogger("mydear")    
import test_logger2
test_logger2.run()           #调用文件 2 中的函数，保证两个模块共同处于生存期
 
#test_logger2.py
#coding:utf-8
 
import logging
def run():
    print logging.getLogger("mydear")
```

输出：
<logging.Logger object at 0x00000000020ECF28>  
<logging.Logger object at 0x00000000020ECF28>  

结果表明两个文件中通过"mydear"调用 getLogger 可以保证得到的 logger 对象是同一个。而分别进行 Logger 类的实例化则不能保证。

有了 logger 之后就可以配置这个 logger，例如设置日志级别 setLevel,绑定控制器 addHandler，添加过滤器 addFilter 等。

配置完成后，就可以调用 logger 的方法写日志了，根据 5 个日志级别对应有 5 个日志记录方法，分别为logger.debug，logger.info，logger.warning，logger.error，logger.critical。

二、配置 Logger 对象的日志级别：

logger.setLevel(logging.DEBUG)  #DEBUG 以上的日志级别会被此 logger 处理

三、创建 handler 对象

handler 负责将 log 分发到某个目的输出，存在多种内置的 Handler 将 log 分发到不同的目的地，或是控制台，或是文件，或是某种形式的 stream，或是 socket 等。一个 logger 可以绑定多个 handler，例如，一条日志可以同时输出到控制台和文件中。

以 FileHandler 和 StreamHandler 	为例：

logfile= logging.FileHandler("./log.txt")  #创建一个handler，用于将日志输出到文件中      
console = logging.StreamHandler()         #创建另一个handler，将日志导向流  

handler 对象也需要设置日志级别，由于一个 logger 可以包含多个 handler，所以每个 handler 设置日志级别是有必要的。用通俗的话讲，比如，我们需要处理 debug 以上级别的消息，所以我们将 logger 的日志级别定为 DEBUG；然后我们想把 error 以上的日志输出到控制台，而 DEBUG 以上的消息输出到文件中，这种分流就需要两个 Handler 来控制。

```
logfile.setLevel(logging.DEBUG)
console.setLevel(logging.ERROR)
```

除了对 handler 对象设置日志级别外，还可以指定 formatter，即日志的输出格式。对 handler 对象设置日志格式，说明了可以将一条记录以不同的格式输出到控制台，文件或其他目的地。

```
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

logfile.setFormatter(formatter) #设置 handler 的日志输出格式
```

formatter 创建时使用的关键字，最后会以列表的形式展现，这不是重点。

四、绑定 handler 到 logger 中

至此 handlers 和 logger 已经准备好了，下面我们就将 handlers 绑定到 logger 上，一个 logger 对象可以绑定多个 handler。

```
    logger.addHandler(logfile)  #logger 是通过 getLogger 得到的 Logger 对象
    logger.addHandler(console)
```

五、使用 logger 真正写日志

```
    logger.debug("some debug message.")
    logger.info("some info message.")
```

 看上去，中间步骤(创建 handler，设置日志级别，设置输出格式等)更像是配置 Logger，一旦配置完成则直接调用写日志的接口即可，稍后这些日志将按照先前的配置输出。
    
呜呼，好多内容啊，来点简单的吧.

下面的代码，是最简单的。导入 logging 之后就进行了写日志操作：

```
#coding:utf-8
 
import logging
logging.debug("debug mes")
logging.info("info mes")
logging.warning("warn mes")
```

控制台输出如下：

```
    WARNING:root:warn mes
```

咦？发生了什么情况，为什么只输出了 warning？handler、logger、formatter 去哪儿了？

-_-!说好的最简单的呢？为了让自己讲信用，我尽可能把它解释成“最简单的”。
    
**知识点 1：logger 间存在继承关系**

logger 通过名字来决定继承关系，如果一个 logger 的名字是 "mydest"，另一个 logger 的名字是"mydest.dest1"（getLogger("mydest.dest1")）,那么就称后者是前者的子 logger，会继承前者的配置。上面的代码没有指定 logger，直接调用 logging.debug 等方法时，会使用所有 logger 的祖先类 RootLogger。

从上面的代码运行结果可以猜测出，该 RootLogger 设置的日志级别是 logging.WARN，输出目的地是标准流。从源码可以更清楚的看出来：

```
root = RootLogger(WARNING)  #设置 WARNING 的级别
```

至于 rootLogger 的输出目的地的配置，我们跟踪 logging.debug 的源代码来看一下：

```	
def debug(msg, *args, **kwargs):
    """
    Log a message with severity 'DEBUG' on the root logger.
    """
    if len(root.handlers) == 0:
        basicConfig()
    root.debug(msg, *args, **kwargs)
```

大约可以看到，如果 rootLogger 没有配置 handler，就会不带参数运行 basicConfig 函数(*请看知识点 2)，我们看一下 basicConfig 的源代码：

```
def basicConfig(**kwargs):
    _acquireLock()
    try:
        if len(root.handlers) == 0:
            filename = kwargs.get("filename")
            if filename:
                mode = kwargs.get("filemode", 'a')
                hdlr = FileHandler(filename, mode)
            else:
                stream = kwargs.get("stream")
                hdlr = StreamHandler(stream)
            fs = kwargs.get("format", BASIC_FORMAT)
            dfs = kwargs.get("datefmt", None)
            fmt = Formatter(fs, dfs)
            hdlr.setFormatter(fmt)
            root.addHandler(hdlr)
            level = kwargs.get("level")
            if level is not None:
                root.setLevel(level)
    finally:
        _releaseLock()
```

因为参数为空，所以我们就看出了，该 rootLoger 使用了不带参数的 StreamHandler，也可以看到诸如 format 之类的默认配置。之后我们跟踪 StreamHandler(因为我们想看到日志输出目的地的配置，而 handler 就是控制日志流向的，所以我们要跟踪它)的源代码：

```
class StreamHandler(Handler):
    """
    A handler class which writes logging records, appropriately formatted,
    to a stream. Note that this class does not close the stream, as
    sys.stdout or sys.stderr may be used.
    """
 
    def __init__(self, stream=None):
        """
        Initialize the handler.
 
        If stream is not specified, sys.stderr is used.
        """
        Handler.__init__(self)
        if stream is None:
            stream = sys.stderr  ####
        self.stream = stream
```

不带参数的StreamHandler将会把日志流定位到sys.stderr流，标准错误流同样会输出到控制台

**知识点 2：basicConfig 函数用来配置 RootLogger**

basicConfig 函数仅用来配置 RootLogger，rootLogger 是所有 Logger 的祖先 Logger，所以其他一切 Logger 会继承该 Logger 的配置。

从上面的 basicConfig 源码看，它可以有六个关键字参数，分别为：

filename：执行使用该文件名为 rootLogger 创建 FileHandler，而不是 StreamHandler  
filemode：指定文件打开方式，默认是"a"  
stream：指定一个流来初始化 StreamHandler。此参数不能和 filename 共存，如果同时提供了这两个参数，则 stream 参数被忽略  
format：为 rootLogger 的 handler 指定输出格式  
datefmt：指定输出的日期时间格式  
level：设置 rootLogger 的日志级别    

使用样例：

```
logging.basicConfig(
                   filename = './log.txt',
                   filemode = 'a',
                   #stream = sys.stdout,
                   format = '%(levelname)s:%(message)s',
                   datefmt = '%m/%d/%Y %I:%M:%S',
                   level = logging.DEBUG
                    )
```

**知识点 3 通过示例详细讨论 Logger 配置的继承关系**

首先准备下继承条件：log2 继承自 log1，logger 的名称可以随意，要注意‘.’表示的继承关系。

```
#coding:utf-8
 
import logging
log1 = logging.getLogger("mydear")
log1.setLevel(logging.WARNING)
log1.addHandler(StreamHandler())
log2 = logging.getLogger("mydear.app")
log2.error("display")
log2.info("not display")
```
**level 的继承**

原则：子 logger 写日志时，优先使用本身设置了的 level；如果没有设置，则逐层向上级父 logger 查询，直到查询到为止。最极端的情况是，使用 rootLogger 的默认日志级别 logging.WARNING。

从源代码中看更为清晰， 感谢 python 的所见即所得：

```
def getEffectiveLevel(self):
        """
        Get the effective level for this logger.
 
        Loop through this logger and its parents in the logger hierarchy,
        looking for a non-zero logging level. Return the first one found.
        """
        logger = self
        while logger:
            if logger.level:
                return logger.level
            logger = logger.parent
        return NOTSET
```

**handler 的继承**

原则：**先将日志对象传递给子 logger 的所有 handler 处理，处理完毕后，如果该子 logger 的 propagate 属性没有设置为 0，则将日志对象向上传递给第一个父 Logger，该父 logger 的所有 handler 处理完毕后，如果它的 propagate 也没有设置为 0，则继续向上层传递，以此类推。最终的状态，要么遇到一个 Logger，它的 propagate 属性设置为了 0；要么一直传递直到 rootLogger 处理完毕。**

在上面实例代码的基础上，我们再添加一句代码，即：

```
#coding:utf-8
 
import logging
log1 = logging.getLogger("mydear")
log1.setLevel(logging.WARNING)
log1.addHandler(StreamHandler())
log2 = logging.getLogger("mydear.app")
log2.error("display")
log2.info("not display")
print log2.handlers      #打印log2绑定的handler
```

输出如下：

```
    display
    []
```

说好的继承，但是子 logger 竟然没有绑定父类的 handler，what's wrong?

看到下面调用 handler 的源代码，就真相大白了。可以理解成，这不是真正的(类)继承，只是"行为上的继承":

```
def callHandlers(self, record):
        """
        Pass a record to all relevant handlers.
 
        Loop through all handlers for this logger and its parents in the
        logger hierarchy. If no handler was found, output a one-off error
        message to sys.stderr. Stop searching up the hierarchy whenever a
        logger with the "propagate" attribute set to zero is found - that
        will be the last logger whose handlers are called.
        """
        c = self
        found = 0
        while c:
            for hdlr in c.handlers:         #首先遍历子 logger 的所有 handler
                found = found + 1
                if record.levelno >= hdlr.level:
                    hdlr.handle(record)
            if not c.propagate:             #如果 logger 的 propagate 属性设置为 0，停止
                c = None    #break out 
            else:                           #否则使用直接父 logger
                c = c.parent
        ...
```

额，最简单的样例牵引出来这么多后台的逻辑，不过我们懂一下也是有好处的。

下面，我们将一些零碎的不是很重要的东西罗列一下，这篇就结束了。

1. 几种 LogLevel 是全局变量，以整数形式表示，也可以但是不推荐自定义日志级别，如果需要将 level 设置为用户配置，则获取 level 和检查 level 的一般代码是：

```
#假设 loglevel 代表用户设置的 level 内容
numeric_level = getattr(logging, loglevel.upper(), None)
if not isinstance(numeric_level, int):
    raise ValueError('Invalid log level: %s' % loglevel)
logging.basicConfig(level=numeric_level, ...)
```

2. format 格式，用于创建 formatter 对象，或者 basicConfig 中，就不翻译了

```
    %(name)s            Name of the logger (logging channel)
    %(levelno)s         Numeric logging level for the message (DEBUG, INFO,
                        WARNING, ERROR, CRITICAL)
    %(levelname)s       Text logging level for the message ("DEBUG", "INFO",
                        "WARNING", "ERROR", "CRITICAL")
    %(pathname)s        Full pathname of the source file where the logging
                        call was issued (if available)
    %(filename)s        Filename portion of pathname
    %(module)s          Module (name portion of filename)
    %(lineno)d          Source line number where the logging call was issued
                        (if available)
    %(funcName)s        Function name
    %(created)f         Time when the LogRecord was created (time.time()
                        return value)
    %(asctime)s         Textual time when the LogRecord was created
    %(msecs)d           Millisecond portion of the creation time
    %(relativeCreated)d Time in milliseconds when the LogRecord was created,
                        relative to the time the logging module was loaded
                        (typically at application startup time)
    %(thread)d          Thread ID (if available)
    %(threadName)s      Thread name (if available)
    %(process)d         Process ID (if available)
    %(message)s         The result of record.getMessage(), computed just as
                        the record is emitted
```

3. 写日志接口

```
    logging.warn("%s am a hero", "I")   #1 %格式以参数形式提供实参
    logging.warn("%s am a hero" % ("I",)) #2 直接提供字符串，也可以使用format，template
    logging.warn("%(name)s am a hero", {'name':"I"})  #关键字参数    
    logging.warn("%(name)s am a hero" % {'name':"I"}) #甚至这样也可以
    logging.warn("%(name)s am a hero, %(value)s" % {'name':"I", 'value':'Yes'}) #原来%也能解析关键字参数，不一定非是元组
    如果关键字和位置参数混用呢，%应该不会有什么作为了，最强也就能这样：
    logging.warn("%(name)s am a hero, %()s" % {'name':"I" ,'': 'Yes'})#也是字典格式化的原理
```

4. 配置 logging：

上面已经讲了如果配置 handler，绑定到 logger。如果需要一个稍微庞大的日志系统，可以想象，我们会使用好多的 addHandler，SetFormatter 之类的，有够烦了。幸好，logging 模块提供了两种额外配置方法，不需要写众多代码，直接从配置结构中获悉我们的配置意图

**方式一：使用配置文件**

```
import logging
import logging.config
logging.config.fileConfig('logging.conf')
# create logger
logger = logging.getLogger('simpleExample')
# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')
 
#配置文件logging.conf的内容
[loggers]
keys=root,simpleExample
[handlers]
keys=consoleHandler
[formatters]
keys=simpleFormatter
[logger_root]
level=DEBUG
handlers=consoleHandler
[logger_simpleExample]
level=DEBUG
handlers=consoleHandler
qualname=simpleExample
propagate=0
[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)
[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=
```

**方式二：使用字典**

请参阅 python2.7.9 Library 文档，链接:

<https://docs.python.org/2/library/logging.config.html?highlight=dictconfig#configuration-dictionary-schema>

5. 众多的 handler 满足不同的输出需要

StreamHandler，FileHandler，NullHandler，RotatingFileHandler，TimedRotatingFileHandler，SocketHandler，DatagramHandler，SMTPHandler，SysLogHandler，NTEventLogHandler，MemoryHandler，HTTPHandler，WatchedFileHandler，

其中前三种在 logging 模块中给出，其他的在 logging.handlers 模块中给出。