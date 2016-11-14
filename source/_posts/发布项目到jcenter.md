---
title: 发布项目到jcenter
date: 2016-11-14 18:00:27
categories:
- 其他
tags:
- 工具
---

关于怎么发布，大体流程可以参考[泓洋](http://blog.csdn.net/lmj623565791/article/details/51148825)大神的就可以了，我只详细的说明下我遇到的坑

## 坑一 404
报404，提前没有创建组织和库，创建库时候名称起名maven，类型选择Maven

## 坑二 傻傻的分不清，那几个id
傻傻的分不清，那几个id

- 第一个id

```Java
publish {
    userOrg = 'msj'//bintray.com用户名
    groupId = 'com.msj.networkcore'//jcenter上的路径
    artifactId = 'networkcore'//项目名称
    publishVersion = '1.0.0'//版本号
    desc = 'Retrofit2+okhttp3+Rxjava封装的网络框架'//描述，不重要
    website = 'https://github.com/centmeng'//网站，不重要；尽量模拟github上的地址，例如我这样的；当然你有地址最好了
}
```
其中 userOrg中的msj是组织名称，而且创建Repository Name输入maven，类型是Maven


- 第二个id，也是在Terminal执行的命令中
```
 ./gradlew clean build bintrayUpload -PbintrayUser=centmeng -PbintrayKey=******************* -PdryRun=false
```

PbintrayUser 的填写自己用户名


## 坑三 lint
Execution failed for task ':lint'.

在每个build.gradle中添加如下代码,在android括号内


```
android{
 lintOptions {
        abortOnError false
        warning 'InvalidPackage'
    }
}
```

## 坑四 JavaDoc检查

在根build.gradle添加如下代码

```
allprojects {
    repositories {
        jcenter()
    }

    tasks.withType(Javadoc) {
        options{
            encoding "UTF-8"
            charSet 'UTF-8'
            links "http://docs.oracle.com/javase/7/docs/api"
        }
    }
}

tasks.withType(Javadoc) {
    options.addStringOption('Xdoclint:none', '-quiet')
    options.addStringOption('encoding', 'UTF-8')
}

```



