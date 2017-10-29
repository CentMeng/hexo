---
title: Java架构师-Maven安装和发布项目到私服
date: 2017-08-14 16:01:48
categories:
- 后端技术
tags:
- Java架构师
---

>　私服是架设在局域网的一种特殊的远程仓库，目的是代理远程仓库及部署第三方构件。有了私服之后，当 Maven 需要下载构件时，直接请求私服，私服上存在则下载到本地仓库；否则，私服请求外部的远程仓库，将构件下载到私服，再提供给本地仓库下载。我们可以使用专门的 Maven 仓库管理软件来搭建私服，比如：Apache Archiva，Artifactory，Sonatype Nexus。这里我们使用 Sonatype Nexus。

## 安装Maven
> 此处不是重点，大家可以自行百度


## 安装Nexus
> Nexus 专业版是需要付费的，这里我们下载开源版 Nexus OSS。[http://www.sonatype.org/nexus/go](http://www.sonatype.org/nexus/go)

环境准备：Linux服务器，JDK1.8

### 下载最新版本，解压放到/usr/local/nexus3

```
tar -zxvf nexus-3.5.0-02-unix.tar.gz
mv nexus-3.5.0-02 /usr/local/nexus3
```

### 启动Nexus

```
# 进入nexus3/bin目录
./nexus run &
```

> 另外中途遇到一个坑，由于习惯不使用root用户，单独创建个用户，即使添加到root组依然不能执行sudo命令。需要在/etc/sudoers中添加创建的用户，保存使用wq!

### 验证启动

默认网址:http://192.168.80.182:8081/(需换成自己ip)

默认用户名和密码：admin/admin123

## 发布项目到私服

### 创建仓库
> 在创建仓库我们先了解下几个概念：

>- 仓库类型
  - hosted 宿主仓库：主要用于部署无法从公共仓库获取的构件（如 oracle 的 JDBC 驱动）以及自己或第三方的项目构件；
  - proxy 代理仓库：代理中央Maven仓库，当PC访问中央库的时候，先通过Proxy下载到Nexus仓库，然后再从Nexus仓库下载到PC本地。这样的优势只要其中一个人从中央库下来了，以后大家都是从Nexus私服上进行下来，私服一般部署在内网，这样大大节约的宽带。
  - group 仓库组：Nexus 通过仓库组的概念统一管理多个仓库，这样我们在项目中直接请求仓库组即可请求到仓库组管理的多个仓库。


具体创建步骤：

- 创建仓库：

<img src="/img/java/nexus/picture/1.png">

- 创建宿主仓库：

<img src="/img/java/nexus/picture/2.png">

- name输入msj-realease ，版本控制选择realease

<img src="/img/java/nexus/picture/3.png">

- 重复第一二三步，创建msj-snapshots,版本控制选择snapshots，最终如下图

<img src="/img/java/nexus/picture/4.png">


### pom配置

添加如下配置：

```xml
  <!-- 配置发布到私服 -->
    <distributionManagement>
        <repository>
            <id>msj-realease</id>
            <name>Release Repository of msj</name>
            <url>http://192.168.80.182:8081/repository/msj-realease/</url>
        </repository>
        <snapshotRepository>
            <id>msj-snapshots</id>
            <name>Snapshot Repository of msj</name>
            <url>http://192.168.80.182:8081/repository/msj-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
```
> url是点对应仓库的copy按钮，会出现相应的url

<img src="/img/java/nexus/picture/5.png">

### 配置本地maven（非服务器maven）

- settings.xml 中增加如下信息

```xml
  <servers>
    <server>
       <id>msj-realease</id>
       <username>admin</username>
       <password>admin123</password>
   </server>
   <server>
       <id>msj-snapshots</id>
       <username>admin</username>
       <password>admin123</password>
   </server>
  </servers>
```

### 部署仓库

项目根目录，执行mvn deploy命令

<img src="/img/java/nexus/picture/6.png">

### 验证结果

<img src="/img/java/nexus/picture/7.png">
