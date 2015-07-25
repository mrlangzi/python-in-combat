# 如何将 Mac OS X10.9 下的 Python2.7 升级到最新的 Python3.3

Mac OS X10.9 默认带了 Python2.7，不过现在 Python3.3.3 出来了，如果想使用最新版本，赶紧升级下吧。基本步骤如下。

第 1 步：下载 Python3.3 
 
下载地址如下：  
[Python3.3](http://python.org/download/releases/3.3.3/)  
这里面有 windows 和 mac os x 下的安装程序，下载那个 64 位的安装程序（估计现在没有用 32 位的 mac os x 的吧）

第 2 步： 
 
安装下载的 img 文件，安装完后的目录如下：  

```
/Library/Frameworks/Python.framework/Versions/3.3
```

第 3 步：移动 python 的安装目录

原来的安装目录见第 2 步，不过所有的 python 都在

```
/System/Library/Frameworks/Python.framework/Versions
```

目录中，所以最好使用下面的命令移动一下，当然不移动也可以。但后面步骤中的某些路径需要修改下。

```
sudo mv /Library/Frameworks/Python.framework/Versions/3.3 /System/Library/Frameworks/Python.framework/Versions
```

第 4 步：改变 Python 安装目录的用户组为 wheel  

```
sudo chown -R root:wheel /System/Library/Frameworks/Python.framework/Versions/3.3
```

python2.7 的用户组就是 wheel，3.3 也照葫芦画瓢吧！

第 5 步：修改 Python 当前安装目录的符号链接
  
在 /System/Library/Frameworks/Python.framework/Versions/ 目录下有一个 Current，这是一个目录符号链接，指向当前的 Python 版本。原来指向 2.7 的，现在指向 3.3。所以应先删除 Current。然后重新建立 Current 符号链接，命令如下：

```
sudo rm /System/Library/Frameworks/Python.framework/Versions/Current
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.3 /System/Library/Frameworks/Python.framework/Versions/Current
```

第 6 步：删除旧的命令符号链接

在 /usr/bin 目录下有 4 个 python 命令的符号链接，使用下面的命令先删除

````
sudo rm /usr/bin/pydoc
sudo rm /usr/bin/python
sudo rm /usr/bin/pythonw
sudo rm /usr/bin/python-config
```

第 7 步：重新建立新的命令符号链接
  
将第 6 步删除的符号链接重新使用下面命令建立，它们都指向 Python3.3 了。

```
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.3/bin/pydoc3.3 /usr/bin/pydoc
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.3/bin/python3.3 /usr/bin/python
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.3/bin/pythonw3.3 /usr/bin/pythonw
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.3/bin/python3.3m-config /usr/bin/python-config
```

第 8 步：更新 /root/.bash_profile 文件中的路径

```
cd ~
vim .bash_profile 
```

在.bash_profile 插入下面的内容即可

```
# Setting PATH for Python 3.3
# The orginal version is saved in .bash_profile.pysave
PATH="/System/Library/Frameworks/Python.framework/Versions/3.3/bin:${PATH}"
export PATH
```

ok，现在重新启动一下 Console，然后执行 python --version，得到的就是 Python 3.3.3。如果在程序中，需要使用下面代码获取 python 版本

```
import platform
print(platform.python_version())
```

如果还是用了如 PyDev 等 IDE，仍然需要更新一下相关的路径。

现在可以使用最新的 Python3.3.3 了。

本文出自 “李宁的极客世界” 博客，请务必保留此出处 <http://androidguy.blog.51cto.com/974126/1336145>