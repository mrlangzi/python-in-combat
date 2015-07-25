# 一个脚本讲述 python 语言的基础规范，适合初学者

最近学 python 的人比较多，今天讲一下 python 的基础：python 脚本的规范、缩进、编写功能函数时注意事项等，这些都是自己编程过程中的心得体会。

## 一、python 脚本的规范：

每个脚本都有自己的规范，以下的规范不是强制的，但是规范一下，可以使你的脚本规范、易懂、方便使用。

```
	#!/usr/bin/env python
	# -*- coding: utf-8 -*-
```

这个写在开头，定义脚本编码。现在多数都是 UTF8 格式，所以写脚本尽量用这个编码，遇到中文可以做编码处理，字符串编码处理主要就是 encode 和 decode

```
import os,urllib,MySQLdb,time,platform 
```
   
导入需要的模块。

```
main():  
   pass
```
  
定义函数  

```
if __name__ == "__main__":
   main()
```

这个就是说脚本从这里往下执行，如果是其他的脚本调用这个脚本，这个脚本不至于执行其他的部分

提示：以上是整个脚本中的规范，大家在写脚本的时候尽量这么做。

## 二、python 的缩进

python 的对缩进要求很严格，缩进不对，就会报语法错误；python 中缩进就是一个 tab 键或是 4 个空格，4 个空格比较麻烦，直接一个 tab 键简单，所以没有特别的需求，缩进一般用 tab 键。缩进类似于分层，同一缩进就是相同的层次。见如下实例：

```
if a==0:
   print a
else:
   print b
```

## 三、每一个功能对应一个函数

这一点我认为最重要，每一个功能就写一个函数，这样你的脚本清晰易懂，脚本其他复用这个功能也方便，脚本也不冗余。不建议不要一个函数里面有好多功能，使函数模块化。

## 四、系统命令的引用

引用系统命令的时候，特别是 linux 命令，一定要写命令的全路径，比如：

```
os.popen("/sbin/ifconfig eth0").read()
```

这个你直接

```
os.popen("ifconfig eth0").read()
```

这样也是没有问题的，起码是你手动执行脚本时，这个是会执行的，但是脚本做 cron 的时候，就不会执行了。所以这个要特别注意。

##　五、异常处理

```
try：
   pass
except Exception,e:
  print e
```

其中 e 就是错误错误信息。try 的异常处理这么写就足够用了，还有其他的方法，不常用。

以下是一个获取本地 ip 地址，从数据库查询 ip 的用途，去连接一个 URL，判断这个 URL 是否可以用，并写日志。主要讲了讲 python 操作数据库的常用用法。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import os,urllib,MySQLdb,time,platform
def log_w(text):
    logfile = "/tmp/websocket.log"
    if os.path.isfile(logfile):
        if (os.path.getsize(logfile)/1024/1024) > 100:
            os.remove(logfile)
    now = time.strftime("%Y-%m-%d %H:%M:%S")
    tt = str(now) + "\t" + str(text) + "\n"
    f = open(logfile,'a+')
    f.write(tt)
    f.close()
def get_idcname(ip):
    try:
        conn = MySQLdb.connect(host = '192.168.8.43',port=3306,user = 'read_app',passwd = '123456',charset='utf8',connect_timeout=20)
        cursor = conn.cursor()#查询出的结果是元组形式，元组和列表基本一样
        #cursor = conn.cursor(cursorclass = MySQLdb.cursors.DictCursor)#查询结果是字典形式
        sql = "select host,user from mysql.user where host='%s'" % ip#python中执行sql语句一次只能是一个sql语句，一次只执行一条，如果用分号分开写多条的话是会报错的，如果是多条sql语句可以多写几个sql和cursor.execute()来分开执行
        cursor.execute(sql)#执行sql语句
        #cursor.executemany("""insert into dist_sniffer.sniffer_order_day values(%s,%s,%s,%s,%s,%s,%s,%s,%s) """,values)#执行组合插入数据库的时候可以用这个，每个%s代表一个数据库字段，values是一个元组或是一个列表
        alldata = cursor.fetchall()#接收sql执行结果，如果是写操作的，这个就不用了
        #conn.commit()如果是写操作，需要这个去提交
        cursor.close()
        conn.close()#关闭数据库回话
        return alldata[0][0].encode('UTF8')#如果是写操作的话就没有返回值了。
    except Exception,e:
        return 0
def get_ip():
    os = platform.system()
    if os == "Linux":
        ip = os.popen("/sbin/ifconfig eth0|grep 'inet addr'").read().strip().split(":")[1].split()[0]
    elif os == "Windows":
        import wmi
        c=wmi.WMI()
        network = c.Win32_NetworkAdapterConfiguration (IPEnabled=1)
        for interface in network:
            if interface.DefaultIPGateway:
                ip = interface.IPAddress[0]
                return ip
                #print interface.IPAddress[0],interface.MACAddress,interface.IPSubnet[0],interface.DefaultIPGateway[0],interface.DNSServerSearchOrder[0],interface.DNSServerSearchOrder[1]
                #获取出网的ip地址、MAC地址、子网掩码、默认网关、DNS
def web_status():
    ip = get_ip()
    idc_name = get_idcname(ip)
    url = "http://www.text.com/index.php?idc_ip=%s&idc_name=%s" % (ip,idc_name)
    get = urllib.urlopen(url)
    if get.getcode() == 200:
        aa = int(get.read().strip())
        if aa == 1:
            text = "Webservice return OK"
        else:
            text =  "Webservice return Error"
    else:
        text = "Conect webservice Error"
    print text
    log_w(text)
if __name__ == "__main__":
    web_status()
```

一开始就要养成一个好习惯，这样对以后 python 编程是十分有益的。自己的深切体会。

本文出自 “王伟” 博客，请务必保留此出处 <http://wangwei007.blog.51cto.com/68019/1239422>