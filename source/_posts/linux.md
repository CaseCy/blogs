---
title: "Linux"
tags:
  - linux
url: 101.html
id: 101
comments: false
categories:
  - linux
date: 2018-10-19 09:31:48
---
引用：[Linux知识整理](http://www.cnblogs.com/hafiz/p/6686187.html)

---

## 各文件夹的作用
```
       1)./             所有其他目录的最顶级根目录
　　　　2)./home    系统用户的家目录，每个用户在该文件夹下有一个与登录名同名的目录作为该用户的家目录，但是root用户的家目录就在根目录下。
　　　　3)./root　　root用户的家目录
　　　　4)./boot     系统内核和开机必须文件所在目录
　　　　5)./etc       系统配置文件所在目录
　　　　6)./dev      系统所有设备文件所在的目录
　　　　7)./usr(unix system resource) 保存程序的相关文件
　　　　8)./tmp      临时文件所在的目录
　　　　9)./var　　 主要放置系统执行过程中经常变化的文件，例如缓存（cache）或者是随时更改的登录文件（log file）
　　　　10)./opt　  用于存储第三方软件的目录，不过我们还是习惯放在/usr/local下
　　　　11)./bin、/usr/bin        常用的可执行指令文件目录
　　　　12)./sbin    root用户才有权限执行的指令
　　　　13)./lib、/usr/lib、/usr/local/lib    系统可复用类库目录
　　　　14)./mnt、/media        外部设备的mountpoint,当检测到设备接入时会自动产生挂载点
　　　　15)./lost+found        每个分区都会创建一个该目录，用户系统异常时恢复丢失的东西
　　　　16)./proc        系统进程以及网络状态信息目录，在内存中
```