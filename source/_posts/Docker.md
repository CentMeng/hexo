---
title: Docker
date: 2018-08-05 11:45:17
categories:
- 后端技术
tags:
- 运维相关
---

> 新接触python的flask和react，由于装环境太麻烦，所以决定用Docker搞一把。首先先打一个python3.6和centos的镜像

## 1. python3.6和centos7

### 1.1 下载基础镜像

```sh
#搜索centos7
docker search centos7ååå
#下载centos7镜像
docker pull centos

#如果没有登录需要使用docker login命令登录

#查看镜像
docker image ls

#启动镜像
docker run -ti image_id 

#查看镜像，可以查看容器id
docker ps -a

```

### 1.2 在基础镜像安装python

- 进入/usr/local目录

- 下载python

```sh
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz
```

> 如果没有wget使用yum安装 yum install wget
> 
> 由于xz结尾的文件，故可以使用如下命令来解压：

```sh
  xz -d Python-3.6.5.tar.xz
  #再执行tar
  tar xvf Python-3.6.5.tar
```

继续执行如下命令：

```sh
#更改目录
mv python-3.6.5 python3

#进入python解压后目录
cd  python3 

./configure --prefix=/usr/local/python3 --enable-optimizations

make

#替换python
# 进入/usr/bin目录
cd /usr/bin

# 之前2.7版本备份
mv python python.bak
其中有python, python2.7, python2三个文件，其实都是指向python2.7的，这里将python备份

#做超链接
ln -s /usr/local/python3/bin/python /usr/bin/python

#检验
python -V
```

中途如果出现失败情况，则通过yum安装相应包

```sh
#如果出现 错误尝试安装
yum install gcc
yum install gcc-c++
#如果sudo命令缺失
yum install sudo

make
#如果make命令缺失
yum install make
```

### 1.3 将容器打包成镜像

> commit时候不要退出docker镜像,commit后可以exit退出

```
docker commit -a "mengshaojie.com" -m "python3.6 in centOS" 容器名称或id 打包的镜像名称:标签  
OPTIONS说明：  
-a :提交的镜像作者；  
-c :使用Dockerfile指令来创建镜像；  
-m :提交时的说明文字；  
-p :在commit时，将容器暂停。

#查看容器id
docker ps -a 

docker commit -a "mengshaojie.com" -m "python3.6.5 in CentOS" 容器id python36-centos7

#标记镜像
docker tag python36-centos7（原镜像） mengshaojie（用户名）/python36-centos7（名称）:1.0（标记即版本号）

```

### 1.4 推送到[远程仓库](https://hub.docker.com)

```
docker push mengshaojie/python36-centos7:1.0
```

## 2. 上一步骤镜像基础上加入mongodb和redis

### 2.1 进入mengshaojie/python36-centos7:1.0镜像

```
docker run -ti mengshaojie/python36-centos7:1.0

```

### 2.2 安装mongodb

#### 2.2.1 下载安装
下载地址：[https://www.mongodb.com/download-center#production](https://www.mongodb.com/download-center#production)

```
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-amazon-3.6.3.tgz
```

安装路径：/usr/local/mongodb

#### 2.2.2 配置环境变量

```
 vi /etc/profile
 
```
最后面增加如下代码：
export PATH=$PATH:/usr/local/mongodb/bin

```
source /etc/profile
source /root/.bashrc
```

#### 2.2.3 设置mongodb的数据库路径

```
mongod --dbpath=/opt/mongodb/data
```

### 2.3 安装redis

#### 2.3.1 下载安装
下载地址：[https://redis.io/](https://redis.io/)

```
wget http://download.redis.io/releases/redis-4.0.9.tar.gz

```

安装路径：/usr/local/redis

### 2.3.2 安装

```
#redis目录
make
```



### 2.4 重复1.3和1.4（记住修改名称）


```

docker ps -a

docker commit -a "mengshaojie.com" -m "python3.6.5 in CentOS" e14e59c9e5fe python36-mongodb363-redis409-centos7

docker tag python36-mongodb363-redis409-centos7 mengshaojie/python36-mongodb363-redis409-centos7:1.0

docker push mengshaojie/python36-mongodb363-redis409-centos7:1.0

```


### 3 Docker其他命令


```
#删除所有容器
sudo docker rm $(docker ps -a -q)
#删除tag和镜像
docker rmi -f mengshaojie/common:1.0
#本地目录拷贝到容器中
docker cp 本地目录 容器id:容器目录
#容器和镜像区别，镜像是相当于安装盘，容器相当于安装盘安装的多少个系统
exit 退出docker后重新进入容器需要执行以下不走
## 查询容器
docker ps -a
## 启动容器
docker start 容器id
## 进入容器，这样和docker run -ti 镜像id不同的是会进入同一个容器，不会新创建个容器
docker attach 容器id
# 导出容器
docker export 容器id > /localpath/***.tar
# 导入容器
docker import 容器id
# 把正在运行的容器保存成镜像
docker commit 
# 保存镜像和加载镜像
docker save 镜像id > /home/myimage-save-0411.tar 

docker load < /home/myimage-save-0411.tar

# 查询所有镜像的ip
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)

```


#导出 export 与 保存 save 的区别

 - (1).export导出的镜像文件大小  小于 save保存的镜像

 - (2).export 导出（import导入）是根据容器拿到的镜像，再导入时会丢失镜像所有的历史，所以无法进行回滚操作（docker tag <LAYER ID> <IMAGE NAME>）；而save保存（load加载）的镜像，没有丢失镜像的历史，可以回滚到之前的层（layer）。（查看方式：docker images --tree）
 - [https://blog.csdn.net/clj198606061111/article/details/50450793](https://blog.csdn.net/clj198606061111/article/details/50450793)

 
