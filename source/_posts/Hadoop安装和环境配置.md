---
title: Hadoop安装和环境配置
date: 2016-12-02 20:43:30
categories:
- 大数据
tags:
- 工具
---
主要分为三步(前提是已安装HomeBrew和Java)，我们这里的安装通过HomeBrew进行安装
1. 配置ssh localhost 免密码登陆
2. 安装Hadoop已经进行配置文件设置 （伪分布式）
3. 运行Hadoop

## 配置SSH


为了确保在远程管理Hadoop以及Hadoop节点用户共享时的安全性, Hadoop需要配置使用SSH协议
首先在系统偏好设置->共享->打开远程登录服务->右侧选择允许所有用户访问

先验证下是否可以免登陆 ssh localhost,这时候如果出现如下错误
```
ssh: connect to host localhost port 22: Connection refused
```

我们只需执行如下两行命令

```
sh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

```

解释：

第一行：ssh -keygen 代表生成密钥，-t代表指定生成的密钥类型，dsa代表dsa密钥认证的意思（密钥类型）；-P用于提供密语，-f 指定生成的密钥文件

第二行：将公钥加入到用于认证的公钥文件中

这时候，我们再试ssh localhost命令，就不会被refused


## 安装Hadoop

### HomeBrew安装
其实很简单，只需一行命令

```
brew install hadoop

```

至于查看安装目录，可以使用以下命令

```
brew list hadoop
```

可以看到我这里安装的是2.7.1版本，并且目录是/usr/local/Cellar/hadoop/2.7.1


### 配置Hadoop

我们需要对4个文件进行修改和配置，具体如下：

#### hadoop-env.sh
文件路径：/usr/local/Cellar/hadoop/2.7.1/libexec/etc/hadoop/hadoop-env.sh

将代码：
```
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true"
```

替换为：

```
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true -Djava.security.krb5.realm= -Djava.security.krb5.kdc="

```

#### core-site.xml

文件路径：/usr/local/Cellar/hadoop/2.7.1/libexec/etc/hadoop/core-site.xml

将下面代码添加到文件中：

```xml
<configuration>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/usr/local/Cellar/hadoop/hdfs/tmp</value>
		<description>A base for other temporary directories.</description>
	</property>
	<property>
		<name>fs.default.name</name>
		<value>hdfs://localhost:9000</value>
	</property>
</configuration>
```

***注： fs.default.name 保存了NameNode的位置，HDFS和MapReduce组件都需要用到它，这就是它出现在core-site.xml 文件中而不是 hdfs-site.xml文件中的原因。在该处配置HDFS的地址和端口号。***

#### mapred-site.xml 

文件路径：/usr/local/Cellar/hadoop/2.7.1/libexec/etc/hadoop/mapred-site.xml或者mapred-site.xml.templete

将下面代码添加到文件中：

```xml
<configuration>
       <property>
         <name>mapred.job.tracker</name>
         <value>localhost:9010</value>
       </property>
 </configuration>
```

变量mapred.job.tracker 保存了JobTracker的位置，因为只有MapReduce组件需要知道这个位置，所以它出现在mapred-site.xml文件中。

#### hdfs-site.xml

文件路径：/usr/local/Cellar/hadoop/2.7.1/libexec/etc/hadoop/hdfs-site.xml

```xml
<configuration>
    <property>
      <name>dfs.replication</name>
      <value>1</value>
     </property>
 </configuration>
```
变量dfs.replication指定了每个HDFS默认备份方式通常为3, 由于我们只有一台主机和一个伪分布式模式的DataNode，将此值修改为1。

## 运行Hadoop

-  启动hadoop之前需要格式化hadoop系统的HDFS文件系统
跳转到目录/usr/local/Cellar/hadoop/2.7.1/bin下
执行如下命令：

```
hadoop namenode -format
```

如果出现command not found :hadoop则需要配置环境变量，具体怎么配置，这是基本技能，就不在这里说明了。

- 然后进入/usr/local/Cellar/hadoop/2.7.1/sbin   

执行如下命令：

```
./start-all.sh
```

或者分开启动：

```
./start-dfs.sh
./start-yarn.sh
```

- 验证hadoop是否启动

Resource Manager: http://localhost:50070
JobTracker: http://localhost:8088
Specific Node Information: http://localhost:8042