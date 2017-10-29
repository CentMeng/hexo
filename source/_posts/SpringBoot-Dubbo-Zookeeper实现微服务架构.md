---
title: SpringBoot+Dubbo+Zookeeper实现微服务架构
date: 2017-10-29 12:45:06
categories:
- 后端技术
tags:
- Java架构师
- spring-boot
---

> 以自己的netpay项目为例，向大家介绍。项目背景，之前项目比较大，要进行拆分，而且放弃之前的架构模式，重新搭架构。基于之前整个系统都使用dubbo进行RPC，所以决定继续使用Dubbo，不适用springcloud。如有问题，还请多多指教。

## 整体框架图
<img src="/img/java/springboot/picture/微服务框架图.png">

- netpay-parent netpay项目的parent项目，只有pom文件
- netpay-api 存放netpay项目所有服务都会用到的公共Entity
- netpay-common 存放netpay项目所有服务都会用到的公共工具类和组件
- netpay-merchant-facade 存放netpay-merchant对外暴露的服务和entity
- netpay-merchant-service netpay-merchant服务实现

## 相关技术流程
- maven私有库搭建
- zookeeper集群部署
- 按框架图创建项目
- spring-boot集成dubbo
- spring-boot集成mybatis
- spring-boot多profile配置

### maven私有库搭建

我们需要把创建的api和common及项目等要以maven方式引用，所以需要搭建maven私有库。具体方法参照[Maven安装和发布项目到私服](https://centmeng.github.io/2017/08/14/Java%E6%9E%B6%E6%9E%84%E5%B8%88-Maven%E5%AE%89%E8%A3%85%E5%92%8C%E5%8F%91%E5%B8%83%E9%A1%B9%E7%9B%AE%E5%88%B0%E7%A7%81%E6%9C%8D/)

### zookeeper集群部署

众所周知,zookeeper是注册中心同时也可以作为分布式锁来使用，对于它的部署和使用可参照[Zookeeper简介说明](https://centmeng.github.io/2017/05/11/Java%E6%9E%B6%E6%9E%84%E5%B8%88-ZooKeeper/)

### 创建项目

- groupId统一为net.netpay

- netpay-parent、netpay-api、netpay-common、netpay-merchant-facade都以创建maven项目方式，netpay-merchant-service以创建spring-boot项目方式。

> 其中netpay-api、netpay-common、netpay-merchant-facade、netpay-merchant-service的parent都继承netpay-parent。

```xml
 <parent>
        <groupId>net.netpay</groupId>
        <artifactId>netpay-parent</artifactId>
        <version>0.0.5-SNAPSHOT</version>
        <relativePath>../netpay-parent/pom.xml</relativePath>
    </parent>
```
> netpay-merchant-service为spring-boot项目，关于不使用默认spring-boot项目方法请参考[Spring Boot 不使用默认的 parent，改用自己的项目的 parent](https://centmeng.github.io/2017/10/29/SpringBoot使用自己项目的parent/)

### spring-boot集成dubbo
- 部署dubbo-admin，下载地址链接: https://pan.baidu.com/s/1geHVnZt 密码: yai8
- 创建生产者
  - pom引用包

  ```xml
 			<dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
		  <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.3</version>
            <exclusions>
                <exclusion>
                    <artifactId>spring</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                 <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
  ```

  - properties文件
  
  ```
  ## Dubbo配置
###应用名称
dubbo.application.name=netpay-merchant
###注册中心类型
dubbo.registry.protocol=zookeeper
###注册中心地址
dubbo.registry.address=127.0.0.1:2181
###暴露服务方式
dubbo.protocol.name=dubbo
###暴露服务端口
dubbo.protocol.port=20880
###Zookeeper地址
zookeeper.address=192.168.60.203:2180
  ```
  
  - 创建Service暴露的服务接口
 
 ```Java
 package net.netpay.merchant.facade;

 /**
 * @author Vincent.M
 * @mail mengshaojie@188.com
 * @date 17/9/20
 * @copyright ©2017 孟少杰 All Rights Reserved
 * @desc
 */
public interface TestService {

    public String getInfo(String id);
}
  ```
  
  - 创建暴露服务的具体实现类
  
  ```Java
  import net.netpay.merchant.entity.query.WzPayRecord;
import net.netpay.merchant.facade.TestService;
import net.netpay.merchant.facade.query.WzPayRecordService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
/**
 * @author Vincent.M
 * @mail mengshaojie@188.com
 * @date 17/9/20
 * @copyright ©2017 孟少杰 All Rights Reserved
 * @desc
 */
@Component
public class TestServiceImpl implements TestService {

    @Override
    public String getInfo(String id) {
        System.out.println("****"+id);
        return "aaa";
    }
}
  ```
  
  - 创建resources/dubbo/dubbo-netpay-merchant.xml
  
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://code.alibabatech.com/schema/dubbo
    http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="${dubbo.application.name}" />

    <!-- 注册中心暴露服务地址 -->
     <!--<dubbo:registry address="multicast://224.5.6.7:1234" />-->
     <!--<dubbo:registry protocol="zookeeper" address="10.170.219.98:2181,10.173.55.173:2181" /> -->
    <dubbo:registry protocol="${dubbo.registry.protocol}" address="${dubbo.registry.address}" />

    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="${dubbo.protocol.name}" port="${dubbo.protocol.port}" />

    <!-- 用户服务接口 -->
<dubbo:service interface="net.netpay.merchant.facade.TestService" ref="testService" />
    <bean id="testService" class="net.netpay.merchant.service.TestServiceImpl"/>
    </beans>
  ```
  
  - 修改启动类Application
  
  ```Java
  @SpringBootApplication
@EnableScheduling
@EnableAsync
@MapperScan("net.netpay.merchant.dao")//MyBatis配置
@ComponentScan({"net.netpay"})//组件扫描
@ImportResource(value = {"classpath:dubbo/dubbo-netpay-merchant.xml","classpath:configs/application-memcached.xml"})//引入dubbo配置和memcache的配置，后续引入其他配置都可以添加
//@PropertySource("classpath:dubbo/dubbo.properties") //引入properties的配置举例，此处屏蔽没有用到
@EnableConfigurationProperties({BaseConfig.class})//自定义的一些配置
public class NetpayMerchantApplication {
	public static void main(String[] args) {
		SpringApplication.run(NetpayMerchantApplication.class, args);
	}
}
  ```
  
  - 启动spring-boot，查看dubbo-admin
  
  <img src="/img/java/springboot/picture/dubbo-admin查看提供者.png">
  
- 创建消费者

  > pom引入的包和properties的配置文件一样，只不过应用名不同,这里只介绍下controller中的引用和dubbo配置文件引用消费者
  
  - 1.dubbo配置文件引用消费者
  
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://code.alibabatech.com/schema/dubbo
    http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="${dubbo.application.name}" />

    <!-- 注册中心暴露服务地址 -->
     <!--<dubbo:registry address="multicast://224.5.6.7:1234" />-->
     <!--<dubbo:registry protocol="zookeeper" address="10.170.219.98:2181,10.173.55.173:2181" /> -->
    <dubbo:registry protocol="${dubbo.registry.protocol}" address="${dubbo.registry.address}" />

    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="${dubbo.protocol.name}" port="${dubbo.protocol.port}" />

   <dubbo:reference id="testService" interface="net.netpay.merchant.facade.TestService" check="false" timeout="60000"/>
  ```
  
  - 2.具体引用Controller
  
  > <b>注意：老版本dubbo引用需使用@Autowired</b>
 
  ```Java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import javax.annotation.Resource;
@Controller
public class TestController {

    @Resource//老版本的dubbo需要使用@Autowired
    private TestService testService;
    @RequestMapping(value = "/" ,produces = "application/json;charset=utf-8")
    @ResponseBody
    public String test(){
        return testService.getInfo("msj");
    }
}
  ```

  - 3.dubbo-admin查看

  <img src = "/img/java/springboot/picture/dubbo-admin查看消费者.png">

### spring-boot集成mybatis

之前用的一直都是JPA，但由于对项目改造，老项目用的iBatis为了尽量减少工作量采用MyBatis进行数据持久化。MyBatis的配置比较简单，可能根据需求不同，也有所不同，这里就大概介绍下，多数据源的问题，大家可以查看下其他相关资料。

- pom引入

```xml
  <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>
```

- properties配置

```
## Mybatis 配置
#此处可以不配置
#mybatis.typeAliasesPackage=net.netpay.merchant.entity
## 两个*表示扫描子文件夹classpath目录在resource下
mybatis.mapper-locations=classpath:mapper/**/*.xml
## 此处可以配置一些别名，如果不需要也可以不配置
#mybatis.config-location=classpath:configs/mybatis-config.xml
```

- Application启动类

```Java
@SpringBootApplication
@EnableScheduling
@EnableAsync
@MapperScan("net.netpay.merchant.dao")//MyBatis配置mapper扫描包
@ComponentScan({"net.netpay"})//组件扫描
@ImportResource(value = {"classpath:dubbo/dubbo-netpay-merchant.xml","classpath:configs/application-memcached.xml"})//引入dubbo配置和memcache的配置
@EnableConfigurationProperties({BaseConfig.class})//自定义的一些配置
public class NetpayMerchantApplication {
public static void main(String[] args) {
    SpringApplication.run(NetpayMerchantApplication.class, args);
}
}
```

### spring-boot多profile配置
在本项目中，引入了三个环境，生产（prod）、测试（test）、开发（dev），只需要创建三个application.properties即可，如下图
<img src="/img/java/springboot/picture/spring-boot多profile.png">

- application.properties指定

```
spring.profiles.active=test
```

Ok,以上就是整个项目的集成过程。如有错误点，希望大家多多指教。
