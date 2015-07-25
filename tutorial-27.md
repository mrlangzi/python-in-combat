# Python 批量更新 nginx 配置文件

工作需要检查线上所有服务器 ngxin 的 host 配置，是否都添加禁止访问目录中带 /.svn/ 和以 tar.gz、tar、zip、等结尾 url，如果没有则添加，由于线上 Nginx 服务器将近百台，每台的 nginx 配置至少 10 几个，手工检查太慢了，本人也不想浪费太多时间做这些无用功。故用 python 写了一个检测脚本。来完成这些无聊事情。

想用 python 完成这些事情，思路大概为：先备份每台服务器原来的配置，然后遍历每台服务器内所有 host 配置，正则匹配 host 配置看是否已经添加相关配置，如有则跳过，遇到有但不全或没有的则在对应的位置上插入相应的配置。除文本插入时我遇到一个问题外，其他比较简单。因为 python 目前没有对文本进行插入操作的模块。整个插入过程需要自己一步一步实现。

当时我考虑文本插入思路：把原配置文件载入内存，正则找到要插入的位置，以此中心把原文件分成两部分，开辟新内存空间按顺序先存放第一部内容，然后存放出插入的内容，然后存放第二部内容。最后 flush 到硬盘。即通俗说法把原配置根据正则进行重定向。

我了解实现这种重定向有两个方法，第一种: 打开原文件，根据规则重定向到新建的文件上。第二种：使用 fileinput 对原文件重定向。

下面是代码，实现过程大概为：先判断目录 nginx 配置目录，如果存在则备份，然后遍历所有配置文件，匹配每个配置文件是否已添加相关内容，如果没有添加或者之前添加了部分内容，找到配置文件第一个花括号“}”，下一行插入相应的内容。

```
#!/usr/bin/python
#coding:utf-8
#Author by Qfeian@20140215
"""
First backup file and then update all nginx configure
in the path "/usr/local/nginx/conf/vhosts".
Insert rule is to find the configure of the first "}"
after inserting content
"""
import fileinput
import os
import re
import sys
import string
import time
from subprocess import call
s_all = """
    location ~* ^(.*)\/\.svn\/ {
                deny all;
        }
        location ~* \.(tar|gz|zip|tgz|sh)$ {
                deny all;
        }
"""
s_svn = """
    location ~* ^(.*)\/\.svn\/ {
                deny all;
        }
"""
s_tar = """
        location ~* \.(tar|gz|zip|tgz|sh)$ {
                deny all;
        }
"""
def file_insert(fname, str):
        r = ur'}'
        f = open(fname)
        old = f.read()
        num = int(re.search(r, old).start())
        f_input = fileinput.input(fname, inplace=1)
        #for line in fileinput.input(fname, inplace=1):
        for line in f_input:
            if r in line:
                print line.rstrip()
                print "\n"+str+"\n"
                f.seek(num+2)
                print f.read()
                break
            else:
                print line.rstrip()
        f.close()
        f_input.close()
        #print "OK! The %s configure has been sucessfull added!" % fname
        print "Ok! 配置文件%s已经添加成功！" % fname
def file_list(f_dir):
    #Check the content of configure,and add the correcsponding content
    rsvn = ur'\\\/\\.svn\\\/'
    rtar = ur'\(tar\|gz\|zip\|tgz\|sh\)'
    if os.path.exists(f_dir): #check dir
        for f_name in os.listdir(f_dir): #list the dir all configure file
           f_path = os.path.join(f_dir,f_name) #Get the filename full path
           f1 = open(f_path)
           f1_old = f1.read()
           f1.close()
           if re.findall(rsvn, f1_old) and re.findall(rtar, f1_old):
               #print "Notice!  %s have been added ,ignore this configure ...." % f_path
               print "Notice! %s 已添加过相关的配置，忽略此配置文件" % f_path
               continue
           elif re.findall(rsvn, f1_old):
               file_insert(f_path, s_tar)
           elif re.findall(rtar, f1_old):
               file_insert(f_path, s_svn)
           else:
               file_insert(f_path, s_all)
    else:
        print "Warnning! dir %s isn't exists!" % f_dir
def file_backup(f_dir):
    # If the file has been backed up with a minute before,will not back up
    bak_dir = "/data/nginx_config_bak/" + time.strftime('%Y%m%d_%H_%M')
    if not os.path.exists(bak_dir):
        os.makedirs(bak_dir)
        if os.path.exists(f_dir):
            cp = "cp -rp"
            cmd = "%s %s %s" % (cp, f_dir, bak_dir)
            call(cmd, shell=True) #backup the configure
if __name__ == "__main__":
    f_dir = "/usr/local/nginx/conf/vhosts"
    file_backup(f_dir)
    file_list(f_dir)
    print "-"*90,"\n 所有 Nginx 配置文件更新完成 !\n"
```

参考：

<http://ryanstd.iteye.com/blog/480781>

<http://docs.python.org/2/library/fileinput.html>

本文出自 “master” 博客，请务必保留此出处 <http://qfeian.blog.51cto.com/1162350/1359465>