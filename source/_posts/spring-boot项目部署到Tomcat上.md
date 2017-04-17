---
title: spring-boot项目部署到Tomcat上
date: 2017-01-07 14:45:46
categories:
- 后端技术
tags:
- spring-boot
---

spring-boot默认提供内嵌的tomcat，所以打包直接生成jar包，用Java -jar命令就可以启动。但是，有时候我们更希望一个tomcat来管理多个项目，这种情况下就需要项目是war格式的包而不是jar格式的包。spring-boot同样提供了解决方案，只需要简单的几步更改就可以了，这里提供maven项目的解决方法：

## 1.将项目的启动类Application.java继承SpringBootServletInitializer并重写configure方法


```
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}

```

## 2.在pom.xml文件中，project下面增加package标签

```
<packaging>war</packaging>
```

## 3.在pom.xml文件中，dependencies下面添加


```
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
</dependency>

```

## 4.打包

```
mvn package
```