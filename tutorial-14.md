# python 监控文件或目录变化

我们经常会遇到监控一个文件或目录的变化，如果有变化，把文件上传备份至备份主机，并且我们还要监控上传过程是否有问题等，根据此需求，查阅了相关的一些材料，编写如下脚本实现这个功能：

```
#!/usr/bin/env python
#coding=utf-8
#######################
#
#Status wd gs/ccs sql file changed
#date:2013-08-26  王伟
#文件有变化上传至备份主机，上传之后验证文件是否正确
#
#######################
import paramiko,os,sys,datetime,time,MySQLdb
from pyinotify import WatchManager, Notifier, ProcessEvent, IN_DELETE, IN_CREATE,IN_MODIFY
'''
CREATE TABLE `wddel_log.status_sql` (
  `ip` varchar(16) NOT NULL COMMENT '机器IP',
  `tar_name` varchar(50) NOT NULL COMMENT '备份文件名字',
  `md5` varchar(50) NOT NULL COMMENT '备份文件 MD5',
  `flag` int(2) NOT NULL COMMENT '0:成功;1:失败',
  `error_log` varchar(100) NOT NULL COMMENT '错误日志',
  `uptime` datetime NOT NULL COMMENT '更新时间',
  KEY `ip` (`ip`),
  KEY `uptime` (`uptime`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8'''#日志表创建脚本
GM_path='/home/asktao/'
center_hostname='192.168.1.100'
center_username='root'
center_password='123456'
center_port=63008
def log2db(ip,tar_name,md5,flag,error='0'):#删除日志入库
    try:
        tar_name = os.path.split(tar_name)[1]
        now  = time.strftime("%Y-%m-%d %H:%M:%S")
        conn = MySQLdb.connect(host = '192.168.1.104',user = 'root',passwd = '1q2w3e4r',charset='utf8',connect_timeout=20)
        cursor = conn.cursor()
        sql = "SELECT ip FROM wddel_log.status_sql WHERE ip='%s'" % ip
        cursor.execute(sql)
        res = cursor.fetchall()
        if len(res)==0:
            inster_sql = "insert into wddel_log.status_sql VALUES('%s','%s','%s',%s,'%s','%s')" % (ip,tar_name,md5,flag,error,now)
            cursor.execute(inster_sql)
            conn.commit()
        else:
            update_sql = "UPDATE wddel_log.status_sql SET md5='%s',flag='%s',error_log='%s',uptime='%s' WHERE ip='%s'" % (md5,flag,error,now,ip)
            cursor.execute(update_sql)
            conn.commit()
        cursor.close()
        conn.close()
    except Exception,e:
        print e
def find_ip():#获取本地 eth0 的 IP 地址
    ip = os.popen("/sbin/ip a|grep 'global eth0'").readlines()[0].split()[1].split("/")[0]
    if "192.168." in ip:
        ip = os.popen("/sbin/ip a|grep 'global eth1'").readlines()[0].split()[1].split("/")[0]
    return ip
def md5sum(file_name):#验证 sql 打包文件的 MD5
    if os.path.isfile(file_name):
        f = open(file_name,'rb')
        py_ver = sys.version[:3]
        if py_ver == "2.4":
            import md5 as hashlib
        else:
            import hashlib
            md5 = hashlib.md5(f.read()).hexdigest()
            f.close()
            return md5
    else:
        return 0
def center_md5(file_name):#上传至备份中心的文件的 MD5
    try:
        s=paramiko.SSHClient()
        s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        s.connect(hostname = center_hostname,port=center_port,username=center_username, password=center_password)
        conm = "/usr/bin/md5sum %s" % file_name
        stdin,stdout,stderr=s.exec_command(conm)
        result = stdout.readlines()[0].split()[0].strip()
        s.close()
        return result
    except Exception,e:
        return e
def back_file(ip,tar_name,tar_md5):#上传文件到备份中心
    remote_dir='/data/sql'
    file_name=os.path.join(remote_dir,os.path.split(tar_name)[1])
    try:
        t=paramiko.Transport((center_hostname,center_port))
        t.connect(username=center_username,password=center_password)
        sftp=paramiko.SFTPClient.from_transport(t)
        sftp.put(tar_name,file_name)
        t.close()
        #print "%s back_file OK" % tar_name
        os.remove(tar_name)
        remot_md5=center_md5(file_name)
        if remot_md5 == tar_md5:
            log2db(ip,tar_name,tar_md5,0)
        else:
            log2db(ip,tar_name,tar_md5,1,'remot_md5!=tar_md5')
    except Exception,e:
        #print "connect error!"
        log2db(ip,tar_name,tar_md5,1,e)
        os.remove(tar_name)
def back_sql():#执行备份
    ip = find_ip()
    tar_name = "/tmp/%s.tar.gz" % ip
    sql_conn = "/usr/bin/find %s -type f  -name '*.sql'|/usr/bin/xargs /bin/tar zcvPf %s" % (GM_path,tar_name)
    sql_tar = os.popen(sql_conn).readlines()
    tar_md5 = md5sum(tar_name)
    if tar_md5 != 0:
        back_file(ip,tar_name,tar_md5)
    else:
        error_log =  "%s not find" % tar_name
        log2db(ip,tar_name,tar_md5,0,error_log)
class PFilePath(ProcessEvent):#文件变化的触发
    def process_IN_CREATE(self, event):
        if os.path.splitext(event.name)[1] == ".sql":
            text = "Create file: %s " % os.path.join(event.path, event.name)
            #print text
            back_sql()
    def process_IN_MODIFY(self, event):
        if os.path.splitext(event.name)[1] == ".sql":
            text = "Modify file: %s " % os.path.join(event.path, event.name)
            #print text
            back_sql()
def FSMonitor():#主监控函数
    back_sql()#运行脚本先备份 sql 文件
    wm = WatchManager()
    mask = IN_CREATE |IN_MODIFY
    notifier = Notifier(wm, PFilePath())
    wdd = wm.add_watch(GM_path, mask, rec=True)
    print 'now starting monitor %s' % (GM_path)
    while True:
        try :
            notifier.process_events()
            if notifier.check_events():
                notifier.read_events()
        except KeyboardInterrupt:
            notifier.stop()
            break
if __name__ == "__main__":
    FSMonitor()
```

此脚本中主要用到 paramiko 和 pyinotify 模块，关于 paramiko 的讲解可以参见：<http://wangwei007.blog.51cto.com/68019/1058726>一文，pyinotify 的用法可以参见官方文档：<https://github.com/seb-m/pyinotify/wiki/Events-types>

本文出自 “王伟” 博客，请务必保留此出处 <http://wangwei007.blog.51cto.com/68019/1284038>