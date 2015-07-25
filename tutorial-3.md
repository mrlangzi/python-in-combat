# python 写报警程序中的声音实现 winsound

写 windowns 下的报警程序，有一个报警声音的实现，在 python 中有个 winsound 模块可以来实现，方法也很简单：

```
import time
import winsound
def play_music():
    winsound.PlaySound('alert', winsound.SND_ASYNC)
    time.sleep(3)
   >import winsound
   PlaySound(sound, flags)
```

sound 是声音文件名字，该文件为 wav 格式的。flags 为其播放的一些参数，如：

SND_LOOP  
重复地播放声音。SND_ASYNC标识也必须被用来避免堵塞。不能用 SND_MEMORY。

SND_MEMORY  
提供给 PlaySound() 的 sound 参数是一个 WAV 文件的内存映像(memory image)，作为一个字符串。
注意：这个模块不支持从内存映像中异步播放，因此这个标识和 SND_ASYNC 的组合将挂起 RuntimeError。

SND_PURGE  
停止播放所有指定声音的实例。

SND_ASYNC  
立即返回，允许声音异步播放。

SND_NODEFAULT  
不过指定的声音没有找到，不播放系统缺省的声音。

SND_NOSTOP  
不中断当前播放的声音。

SND_NOWAIT  
如果声音驱动忙立即返回。

MB_ICONASTERISK  
播放 SystemDefault 声音。

MB_ICONEXCLAMATION  
播放 SystemExclamation 声音。

MB_ICONHAND  
播放 SystemHand 声音。
  
MB_ICONQUESTION  
播放 SystemQuestion 声音。

MB_OK  
播放 SystemDefault 声音。

python 蜂鸣，通过 python 让电脑发声:

import winsound    
winsound.Beep(37, 2000)

37 是频率(Hz), 2000 是蜂鸣持续多少毫秒(ms).    
第一个参数 frequency 表示分贝数，大小在 37 到 32767 之间。第二个参数是持续时间，以毫秒为单位

本文出自 “王伟” 博客，请务必保留此出处 <http://wangwei007.blog.51cto.com/68019/1231091>