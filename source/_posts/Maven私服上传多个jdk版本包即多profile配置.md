---
title: Maven私服上传多个jdk版本包即多profile配置
date: 2017-11-10 23:01:13
categories:
- 后端技术
tags:
- Java架构师
- maven
---

> 在项目中，我们打的jar包，由于使用者jdk版本不同，有可能导致Unsupported major.minor version 52.0 类似这样的错误，解决方法就是上传私服多个jdk版本。

-----

## 项目整体框架图
<img src="/img/maven/picture/微服务框架图.png">

- netpay-parent netpay项目的parent项目，只有pom文件
- netpay-api 存放netpay项目所有服务都会用到的公共Entity
- netpay-common 存放netpay项目所有服务都会用到的公共工具类和组件
- netpay-merchant-facade 存放netpay-merchant对外暴露的服务和entity
- netpay-merchant-service netpay-merchant服务实现

-----

## 配置

- 1.netpay-parent中pom.xml

所有项目都继承parent，所以给parent设置了属性，则其他项目也适用。在netpay-parent中pom.xml中添加如下代码

```xml
   <build>
        <finalName>netpay-parent</finalName>
        <plugins>

            <!-- 解解决maven update project 后版本降低为1.5的bug -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
           
        </plugins>
    </build>


    <profiles>
        <profile>
            <id>default</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <jar.source>${java.version}</jar.source>
                <jar.target>${java.version}</jar.target>
            </properties>
        </profile>
        <profile>
            <id>jdk16</id>
            <build>
                <plugins>
                    <plugin>
                        <artifactId>maven-jar-plugin</artifactId>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                                <configuration>
                                    <classifier>jdk16</classifier>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
            <properties>
                <jar.source>1.6</jar.source>
                <jar.target>1.6</jar.target>
            </properties>
        </profile>
    </profiles>
```

打包不同版本的jdk，只需要修改build中plugin的jar.source，jar.target改成响应jdk版本例如1.8或者1.6。

<b>profiles是重点,也是多profile配置的核心，在这里定义了两个profile，一个是default默认的，一个是jdk16（如需改成其他的jdk则对应修改jdk16和1.6）</b>

- 2.修改运行环境

 > 此处用idea作为开发工具，eclipse和myeclipse同理，在idea中每打包一个jdk版本修改如下几个地方

  - settings（Preferences）
  
  <img src="/img/maven/picture/maven的importing.jpg">
  
  <img src="/img/maven/picture/maven的runner.jpg">

  - Module Settings
  
  > 各个子项目都需要修改
  
  <img src="/img/maven/picture/modulesetting1.jpg">
  
  <img src="/img/maven/picture/modulesetting2.jpg">
  
- 3.Rebuild 项目

- 4.上传到私服

执行命令（在netpay-parent项目下）：

  默认方式：
  
  ```
  mvn deploy
  ```
  
  使用jdk16配置方式：
  
  ```
  mvn deploy -P jdk16
  ```

-----

## 使用

- 默认方式

```xml
<dependency>
			<groupId>net.netpay</groupId>
			<artifactId>netpay-common</artifactId>
			<version>0.0.1-SNAPSHOT</version>
			<classifier>jdk16</classifier>
</dependency>
```

- 使用jdk16版本方式

```xml
<dependency>
			<groupId>net.netpay</groupId>
			<artifactId>netpay-common</artifactId>
			<version>0.0.1-SNAPSHOT</version>
			<classifier>jdk16</classifier>
		</dependency>
```

Ok,以上就是maven配置多个profile的方式，望大家多多指教。