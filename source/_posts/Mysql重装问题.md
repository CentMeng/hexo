---
title: Mysql重装问题
date: 2017-01-07 14:32:30
tags:
- Mysql
categories:
- 数据库
---



这个问题，还是我大学学习Mysql遇到这个问题，当时做了总结记录下来。没想到朋友今天遇到相同问题，我还能找到自己总结的word文档。所以，打算放到自己博客上，帮助其他人能够看到解决，二是方便自己归档记录

***当时用的还是Mysql5.1,所以这次总结也已此为主，其他版本无用，可不要怪我哦。***

## 1、控制面板里的增加删除程序内进行删除

## 2、删除MySQL文件夹下的my.ini文件，如果备份好，可以直接将文件夹全部删除

## 3、注册表信息删除

- HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\MySQL 目录删除

- HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\MySQL 目录删除

- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\MySQL 目录删除

## 4、这一条是很关键的

C:\Documents and Settings\All Users\Application Data\MySQL这里还有MySQL的文件，必须要删除***注意：Application Data这个文件夹是隐藏的，需要打开个文件夹选择菜单栏 工具→文件夹选项→查看→隐藏文件和文件夹 一项选上 显示所有文件和文件夹 确定***

OK！以上4步完成，再次安装吧

当时自己还用的Windows所以这篇也是这对Winows系统。


