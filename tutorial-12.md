# Python 编写的 socket 服务器和客户端

服务器端：

```
#!/usr/bin/python
import socket
host='127.0.0.1'
port=8123
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.bind((host,port))
s.listen(2)
try:
       while True:
               conn,add=s.accept()
               while True:
                       data2=''
                       data1=conn.recv(3)
                       if data1=='EOF':
                               conn.send('hello clietn1')
                               break
                       if data1=='FOE':
                               conn.send('hello client2')
                               break
                       data2+=data1
                       print data2
except KeyboardInterrupt:
       print "you have CTRL+C,Now quit"
       s.close()
```

注：服务器端一次只接收 3 个字节的数据，我让读取进入循环，然后不断累加到 data2 中，当读取到 EOF 时，退出打印 data2，当读取 FOE 时，退出打印 data2，（EOF 和 FOE 是客户端发送完数据时发送的结束符），当接收到 CTRLC+C 时，关闭 socket

客户端 1：

```
#!/usr/bin/env python
import socket
import os
ss=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
ss.connect(('127.0.0.1',8123))
#f=open('aa','wb')
ss.sendall('hello serverdddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd')
os.system('sleep 1')
ss.send('EOF')
data=ss.recv(1024)
print "server dafu %s"%data
ss.close()
```

客户端 2：

```
#!/usr/bin/env python
import socket
import os
ss=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
ss.connect(('127.0.0.1',8123))
#f=open('aa','wb')
ss.sendall('wokao sile')
os.system('sleep 1')
ss.send('FOE')
data=ss.recv(1024)
print "server dafu %s"%data
ss.close()
```

本文出自 “linux 开源-不断的总结....” 博客，请务必保留此出处 <http://fantefei.blog.51cto.com/2229719/1274582>