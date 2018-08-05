---
title: SpringBoot starter制作
date: 2018-08-05 11:10:46
categories:
- 后端技术
tags:
- spring-boot
---


> 大家都知道，在spring-boot有众多starter。在传统Maven项目中通常将一些层、组件拆分为模块来管理，以便相互依赖复用，在spring-boot项目中我们则可以创建自定义spring-boot starter来达成该目的。
å

这里我们做一个KafKa的starter来进行讲解，首先，我们先引入依赖，pom.xml如下

- pom.xml引入

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zongjie</groupId>
    <artifactId>spring-boot-kafka-starter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.20</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.10</artifactId>
            <version>0.8.2.0</version>
            <scope>compile</scope>
            <exclusions>
                <exclusion>
                    <artifactId>log4j</artifactId>
                    <groupId>log4j</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.15</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>1.5.8.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

> [Spring官方](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#boot-features-custom-starter-naming) Starter通常命名为spring-boot-starter-{name}如 spring-boot-starter-web， Spring官方建议非官方Starter命名应遵循{name}-spring-boot-starter的格式。


- 写配置和service

这里我们写kafka需要从application.properties读取的配置，在引用的项目中，配置即可

以生产者为例的配置文件

```Java
/**
 * @author Vincent.M
 * @mail mengshaojie@188.com
 * @date 2018/5/21
 * @copyright ©2018 孟少杰 All Rights Reserved
 * @desc
 */
@Data
@ConfigurationProperties(ProducerProperties.PREFIX)
public class ProducerProperties {

    public static final String PREFIX = "com.mq.kafka.producer";

    private String serializerClass;
    private String producerType;
    private int maxQueueBuffer;
    private String brokerList;
    private int requiredAck;

    public Properties getProperties() {
        Properties properties = new Properties();
        if (serializerClass != null) {
            properties.setProperty("serializer.class", serializerClass);
        }
        if (producerType != null) {
            properties.setProperty("producer.type", producerType);
        }
        if (maxQueueBuffer > 0) {
            properties.setProperty("queue.buffering.max.ms", String.valueOf(maxQueueBuffer));
        }
        if (brokerList != null) {
            properties.setProperty("metadata.broker.list", brokerList);
        }
        if (requiredAck > 0) {
            properties.setProperty("request.required.acks", String.valueOf(requiredAck));
        }

        return properties;
    }
}
```

service需要被引用的类

```
public class KafkaProducer implements InitializingBean {

    private static ConcurrentHashMap<String, ProducerWrapper> producerWrapperMap = new ConcurrentHashMap<String, ProducerWrapper>();

    ProducerProperties producerProperties;

    public KafkaProducer(ProducerProperties producerProperties){
        this.producerProperties = producerProperties;
    }

    /**
     * 同步发送生产者数据
     *
     * @param topic    主题
     * @param routeKey 消息分组的字段,为null会选择随机分组
     * @param msg      消息体
     */
    public void send(KafkaTopics topic, String routeKey, String msg) {
        getProducer(topic.getTopic()).send(topic, routeKey, msg);
    }

    /**
     * 异步发送生产者数据，会先放到内存队列中再发送
     *
     * @param topic
     * @param routeKey
     * @param msg
     */
    public void sendAsync(KafkaTopics topic, String routeKey, String msg) {
        getProducer(topic.getTopic()).sendAsync(topic, routeKey, msg);
    }

    /**
     * 获取信息发送器
     *
     * @param topic
     * @return
     */
    private  ProducerWrapper getProducer(String topic) {
        if (!producerWrapperMap.containsKey(topic)) {
            synchronized (producerWrapperMap) {
                if (!producerWrapperMap.containsKey(topic)) {
                    if (producerProperties != null) {
                        producerWrapperMap.put(topic, new ProducerWrapper(topic, producerProperties.getProperties()));
                    }
                }
            }
        }
        return producerWrapperMap.get(topic);
    }

    public void afterPropertiesSet() throws Exception {
        Assert.notNull(producerProperties, "producerProperties 'producer' is required");
    }

}
```



