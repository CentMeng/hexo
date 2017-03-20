---
title: Android抓包
date: 2016-12-30 23:45:45
categories:
- Android
tags:
- 工具
---
#手机上抓包

安卓系统上常用的抓包工具是 tcpdump，具体的操作步骤如下：

- 1) PC 上安装 adb，直接下载或者通过 eclipse 中的安卓开发环境自带的工具集获得。

- 2) 下载 tcpdump: http://www.strazzere.com/Android/tcpdump


  - 网盘地址：链接: https://pan.baidu.com/s/1jHGmImq 密码: b15h


- 3) 检查设备连接情况。

```
adb devices

```


- 4) 把 tcpdump 拷贝至 /data/local 目录，注意，/data/local 目录需要 root 权限才能拷入，所以先使用 adb push 拷贝至手机 /sdcard 目录，再使用 adb shell 进入命令行，使用 su 进入 root 状态， cp 至 /data/local 目录。

```
adb push ./tcpdump /sdcard/tcpdump
adb shell
su
cp /sdcard/tcpdump /data/local/tcpdump
```



- 5) 为 tcpdump 添加可执行权限

```
cd /data/local
chmod 6755 tcpdump

```


- 6) 启动抓包，使用命令

```
 /data/local/tcpdump -p -vv -s 0 -w /sdcard/capture.pcap
```
***“Got”后面的数字表示当前抓到的包的数量。如果在变化，表示有网络流量。***

- 7) 我们刚刚把抓包的结果保存在了 /sdcard 目录下，导出抓包的结果到电脑。

大家看了这么多步骤是不是觉得很复杂，不过不要紧，我们自研的 GT 工具已经把 tcpdump 抓包功能集成进去了，后面介绍 GT 的章节里面会详细介绍抓包方法，在手机上有用户操作界面可以实现一键式抓包，另外 GT 也提供了命令行方式的接口启动抓包，启动命令为：

```
adb shell am broadcast – a com.tencent.wstt.gt.plugin.tcpdump.startTest – es filepath “/sdcard/GT/Tcpdump/Capture/test.pcap”
```
停止抓包命令为：

```
adb shell am broadcast -a com.tencent.wstt.gt.plugin.tcpdump.endTest
```
后面可以看到，命令行方式可以方便的做进自动化测试脚本中。

第三步：根据抓包文件统计流量

这里需要对抓包文件分析，获得抓取的报文总流量，目前 PC 上的抓包软件 wireshark 就提供这样的统计功能。用 wireshark 打开刚刚的抓包文件，点击 Statistics->Summary.

流量的数值为 Bytes 一行的 Displayed 一栏。

<img src="/img/android/zhuabao/1.png" >

<center>图 wireshark 流量统计详细页</center>
