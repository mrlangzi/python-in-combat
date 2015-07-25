# python 中用 string.maketrans 和 translate 巧妙替换字符串

python 中用 string.maketrans 和 translate 巧妙替换字符串

将 nginx 日志中字符串 [2013-07-03T00:29:40-05:00] HTTP  格式化为："2013-07-03 00:29:40-05:00"

整条日志如下：

92.82.22.46 - - [2013-07-03T00:29:40-05:00] "GET /images/mask_bg.png HTTP/1.1" 200 195      "http://www.chlinux.net/" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)" "-"

将[2013-07-03T00:29:40-05:00] 替换成为："2013-07-03 00:29:40-05:00"

把 [] 换成"",然后把 T 替换成空格

做法如下：

```
>>> s='''92.82.22.46 - - [2013-07-03T00:29:40-05:00] "GET /images/mask_bg.png HTTP/1.1" 200 195 "http://www.chlinux.net/" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)" "-"'''
>>> table = string.maketrans('[]','""')
>>> s.translate(table)
'92.82.22.46 - - "2013-07-03T00:29:40-05:00" "GET /images/mask_bg.png HTTP/1.1" 200 195 "http://www.chlinux.net/" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)" "-"'
>>> s.translate(table).replace('T', ' ',1)#替换掉第一个T为空格
'92.82.22.46 - - "2013-07-03 00:29:40-05:00" "GET /images/mask_bg.png HTTP/1.1" 200 195 "http://www.chlinux.net/" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)" "-"'
也可以这样：
>>> table = re.sub('\[|\]','"',s).replace('T', ' ',1)
>>>print table
'92.82.22.46 - - "2013-07-03 00:29:40-05:00" "GET /images/mask_bg.png HTTP/1.1" 200 195 "http://www.chlinux.net/" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)" "-"'
```

本文出自 “王伟” 博客，请务必保留此出处 <http://wangwei007.blog.51cto.com/68019/1242206>