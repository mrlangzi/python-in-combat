# Python 处理 cassandra 升级后的回滚脚本

前几天开发把分布式存储服务器 cassandra 升级了，担心升级不成功，所以写了一个升级回滚失败的脚本

环境说明：

       升级后的目录结构为：
    Cassandra 数据文件放在 /opt/cassandra/data/ 下
    data 目录下有很多 keyspace 的目录：如：system 目录，Keyspcace 目录下有 coumlfailmly 目录，

如:/opt/cassandra/data/system/peers/snapshots/1370569934254  此下面是所有的数据文件

       如：system-peers-ib-10-Summary.db
           system-peers-fsdfsfsfd-10-Summary.db

现要把所有 Keyspace 目录下的所有 db 文件挪到 /opt/cassandra/data/system下，（-ib- 文件除外）

如： /opt/cassandra/data/system/peers/snapshots/1370569934254/system-peers-fsdfsfsfd-10-Summary.db=======》 /opt/cassandra/data/system/peers-fsdfsfsfd-10-Summary.db   (注意还得重命令，把文件名的 keyspace 部份去掉)

上脚本：

![pic](images/1.jpg) 

脚本支持： /cassandra/data 和 /opt/cassasnra/data 这个路径下的目录。

本文出自 “决胜千里之外” 博客，请务必保留此出处 <http://lubing.blog.51cto.com/5293532/1221146>