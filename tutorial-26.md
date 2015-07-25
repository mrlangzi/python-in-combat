# 关于 python 调用 zabbix api 接口的自动化实例 [结合 saltstack]

## 前言：

这两天一直做一个叫集群配置管理平台的自动化项目，写了有 20 多天了，项目做的还算顺利，只是一堆的接口需要写，有点烦。因为 clusterops 项目到最后肯定是要和监控平台做结合的，这两天也抽时间看了下。以前自己也写过不少类似 zabbix 的接口调用教程，当时看的时候，由于时间有限，也都是草草跑 demo。

请大家多关注下我的独立博客，更多的关于 zabbix 二次开发的话题，<http://xiaorui.cc>

zabbix 的接口挺好理解，任何的程序都可以写，甚至是 linux 的 curl 命令。我这边用 python 的 urllib、urllib2 来搞的，当然会 php 的就更好了，因为 zabbix 的接口是 php 写的，懂 php 可以直接用现成的。

zabbix 官网有大量的接口，你只要会用 zabbix，然后看下 api 的说明，应该就没啥问题了  
<https://www.zabbix.com/documentation/1.8/api>

简单说三个例子，入个门。

获取 KEY

```
!/usr/bin/env python2.7
#coding=utf-8
import json
import urllib2
# based url and required header
url = "http://monitor.example.com/api_jsonrpc.php"
header = {"Content-Type": "application/json"}
# auth user and password
data = json.dumps(
{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
    "user": "Admin",
    "password": "zabbix"
},
"id": 0
})
# create request object
request = urllib2.Request(url,data)
for key in header:
    request.add_header(key,header[key])
# auth and get authid
try:
    result = urllib2.urlopen(request)
except URLError as e:
    print "Auth Failed, Please Check Your Name And Password:",e.code
else:
    response = json.loads(result.read())
    result.close()
    print "Auth Successful. The Auth ID Is:",response['result']
```

获取 hostlist

```
#!/usr/bin/env python2.7
#coding=utf-8
import json
import urllib2
#xiaorui.cc
url = "http://10.10.10.61/api_jsonrpc.php"
header = {"Content-Type": "application/json"}
# request json
data = json.dumps(
{
    "jsonrpc":"2.0",
    "method":"host.get",
    "params":{
        "output":["hostid","name"],
        "filter":{"host":""}
    },
    "auth":"dbcd2bd8abc0f0320fffab34c6d749d3",
    "id":1,
})
# create request object
request = urllib2.Request(url,data)
for key in header:
    request.add_header(key,header[key])
# get host list
try:
    result = urllib2.urlopen(request)
except URLError as e:
    if hasattr(e, 'reason'):
        print 'We failed to reach a server.'
        print 'Reason: ', e.reason
    elif hasattr(e, 'code'):
        print 'The server could not fulfill the request.'
        print 'Error code: ', e.code
else:
    response = json.loads(result.read())
    result.close()
    print "Number Of Hosts: ", len(response['result'])
    for host in response['result']:
        print "Host ID:",host['hostid'],"Host Name:",host['name']
```

添加主机

```
#!/usr/bin/env python2.7
#coding=utf-8
import json
import urllib2
#xiaorui.cc
url = "http://10.10.10.61/api_jsonrpc.php"
header = {"Content-Type": "application/json"}
# request json
data = json.dumps(
{
    "jsonrpc":"2.0",
    "method":"host.create",
    "params":{
        "host": "10.10.10.67","interfaces":
        [{"type": 1,"main": 1,"useip": 1,"ip": "10.10.10.67","dns": "","port": "10050"}],
        "groups": [{"groupid": "2"}],"templates": [{"templateid": "10087"}]
        },
    "auth":"dbcd2bd8abc0f0320fffab34c6d749d3",
    "id":1,
}
)
# create request object
request = urllib2.Request(url,data)
for key in header:
    request.add_header(key,header[key])
# get host list
try:
    result = urllib2.urlopen(request)
except URLError as e:
    if hasattr(e, 'reason'):
        print 'We failed to reach a server.'
        print 'Reason: ', e.reason
    elif hasattr(e, 'code'):
        print 'The server could not fulfill the request.'
        print 'Error code: ', e.code
else:
    response = json.loads(result.read())
    result.close()
    print 'ok'zai
```

原文： <http://rfyiamcool.blog.51cto.com/1030776/1358792>  

我个人觉得 zabbix 的 rest api 难点在于 key 相关的认证，会了之后，再看官网的 api 文档就一目了然了。

## 啥时候用？

在我的集群平台下，我可以把暂时下线的服务器，在平台上去除，但是大家有没有想到，你要是吧主机删掉后，监控端会有一堆的通知发给你，所以，在处理主机的时候，顺便调用 zabbix 的接口，把该主机的监控项目给删掉。

在我通过 saltstack 添加 lvs 后端主机的时候，我也同样可以调用接口，把后端的主机相应的监控都给加进去。

就先这样，有时间再丰富下该文章。

本文出自 “峰云，就她了。” 博客，谢绝转载！