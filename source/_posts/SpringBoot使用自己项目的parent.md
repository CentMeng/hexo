---
title: SpringBoot使用自己项目的parent
date: 2017-10-29 12:49:57
categories:
- 后端技术
tags:
- 问题解决
- spring-boot
---


我们在使用spring-boot的时候，parent需要继承一个spring的 spring-boot-starter-parent，但是有时候在我们自己的项目中，会定义一个自己的 parent 项目，这种情况下，我们该如何达到即使用了我们的parent，又将spring-boot集成进来呢？

其实，spring-boot也想到了这种情况，所以只需几步就可以完成：

## 自己parent

>dependencyManagement是与dependencies同级。<b>请注意，它的 type 是 pom，scope 是 import，这种类型的 dependency 只能在 dependencyManagement 标签中声明。</b>

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.8.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 自己子项目
-  集成自己的parent

```
<parent>
        <groupId>net.netpay</groupId>
        <artifactId>netpay-parent</artifactId>
        <version>0.0.5-SNAPSHOT</version>
        <relativePath>../netpay-parent/pom.xml</relativePath>
</parent>
```

- 集成spring-boot相关包

> 需要我们注意子项目的 dependencies 中，不需要再次添加对 spring-boot-dependencies 的声明了，否则子项目 将无法编译通过。 这是因为 spring-boot-dependencies 根本就没有对应的jar包，它只是一个 pom 配置，可以去 [maven仓库](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-dependencies/1.5.8.RELEASE/)  看一下。 

> 同时有了parent的 spring-boot-dependencies ，我们在子项目中使用到的相关依赖，就不需要声明version了。

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```





