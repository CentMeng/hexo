---
title: AOP总结（自定义注解和AOP及反射结合）
date: 2018-08-05 11:32:22
categories:
- 后端技术
tags:
- Spring
---

> 面向切面编程即AOP（Aspect Oriented Program），是在运行时，动态地将代码切入到类的指定方法，指定位置上的编程思想就是面向切面的编程。

在工作中，遇到了想实时监听订单状态变更的需求，于是决定用kafka消息队列来传递订单变更的消息，但是再每个调用订单变更的地方太麻烦了，所以决定写个公共方法，然后通过切面编程来解决此需求。废话不多说了，上代码

## demo代码

### 自定义注解

```Java
/**
 * @author Vincent.M
 * @mail mengshaojie@188.com
 * @date 2018/5/18
 * @copyright ©2018 孟少杰 All Rights Reserved
 * @desc 订单状态改变，kafka集群生产消息注解
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface KafKaProductor {
    KafkaTopics topic();
    String routeKey() default "";
    String message() default "";
    //type =1 增删改查状态改变
    int type() default 0;
}
```

### 方法

```Java
 @KafKaProductor(topic=KafkaTopics.ORDER,type = 1)
    public int insertSelective(Userorder userorder){
        return orderMapper.insertSelective(userorder);
    }
```

### AOP切面代码

```Java
import com.alibaba.fastjson.JSONObject;
import com.zongjie.mq.kafka.enums.KafkaTopics;
import com.zongjie.mq.kafka.KafkaProducer;
import com.demo.order.server.annotations.KafKaProductor;
import com.zongjie.order.server.model.UserOrderMQ;
import com.zongjie.order.server.model.Userorder;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

/**Java
 * @author Vincent.M
 * @mail mengshaojie@188.com
 * @date 2018/5/18
 * @copyright ©2018 孟少杰 All Rights Reserved
 * @desc KafKa发送消息切面
 */
@Aspect
@Component
@Slf4j
public class KafKaProductAspect {


    @Autowired
    KafkaProducer kafkaProducer;

    @Around("execution(* com.demo.order.server.service.impl..*.insertSelective(..)) || execution(* com.demo.order.server.service.impl..*.updateByPrimaryKeySelective(..)) ")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        log.info("[KafKa生产],订单，执行类{},执行方法{}", signature.getDeclaringTypeName(), signature.getName());
        Method targetMethod = signature.getMethod();
        Object obj = null;
        if (targetMethod.isAnnotationPresent(KafKaProductor.class)) {
            KafKaProductor orderStatusUpdate = (KafKaProductor) targetMethod.getAnnotation(KafKaProductor.class);
            KafkaTopics topic = orderStatusUpdate.topic();
            String routeKey = orderStatusUpdate.routeKey();
            String message = orderStatusUpdate.message();
            int type = orderStatusUpdate.type();
            log.info("[KafKa生产],订单，执行类{},执行方法{},topic:{},routeKey:{},message:{}", signature.getDeclaringTypeName(), signature.getName(), topic, routeKey, message);
            obj = pjp.proceed();
            switch (type) {
                case 1:
                    if ((Integer) obj > 0) {
                        Userorder userorder = (Userorder) pjp.getArgs()[0];
                        log.info("[KafKa生产],订单type=1，执行类{},执行方法{},topic:{},原message:{}", signature.getDeclaringTypeName(), signature.getName(), topic, JSONObject.toJSONString(userorder));
                        UserOrderMQ userOrderMQ = new UserOrderMQ();
                        BeanUtils.copyProperties(userorder,userOrderMQ);
                        log.info("[KafKa生产],订单type=1，执行类{},执行方法{},topic:{},转换后message:{}", signature.getDeclaringTypeName(), signature.getName(), topic, JSONObject.toJSONString(userOrderMQ));
                        kafkaProducer.send(com.zongjie.mq.kafka.enums.KafkaTopics.ORDER, routeKey, JSONObject.toJSONString(userOrderMQ));
                    }
                    break;
            }
        }else{
            obj = pjp.proceed();
        }
        return obj;
    }
}

```

上面的例子，是针对订单变更，发送kafka生产者消息的例子。使用了自定义注解,来传递消息类型，消息内容等信息，用AOP的配置execution的表达式设置方法切面，然后通过反射获取注解内容，和方法内容。


下面来总结下，首先说下execution表达式吧

## execution表达式

> 在多个表达式之间使用  || , or 表示  或 ，使用  && , and 表示  与 ， ！ 表示非，例如上面demo

- execution(* com.demo.service.impl..*.*(..))


<table>
<tr>
<td> 符号 </td>
<td> 含义 </td>
</tr>
<tr>
<td> execution（） </td>
<td> 表达式的主体 </td>
</tr>
<tr>
<td> 第一个”*“符号 </td>
<td> 表示返回值的类型任意 </td>
</tr>
<tr>
<td> com.demo.service.impl </td>
<td> AOP所切的服务的包名 </td>
</tr>
<tr>
<td> 包名后面的”..“ </td>
<td> 表示当前包及子包 </td>
</tr>
<tr>
<td> 第二个”*“ </td>
<td> 表示类名，*即所有类。此处可以自定义 </td>
</tr>
<tr>
<td> .*(..) </td>
<td> 表示任何方法名，括号表示参数，两个点表示任何参数类型,set*标识set开头的方法*Dao标识Dao结尾的方法 </td>
</tr>
</table>

举例说明:

```
com.demo.order.server.service.impl..*.insert*(..)) 

com.demo.order.server.service.impl包下所有insert开头的方法
```

## advice

> 大家都知道spring中有@Before、@Around和@After等advice，这里我们介绍下advice的执行场景和注意点。

- @Before是在所拦截方法执行之前执行一段逻辑。@After 是在所拦截方法执行之后执行一段逻辑。@Around是可以同时在所拦截方法的前后执行一段逻辑。

- 如果在同一个 aspect 类中，针对同一个 pointcut，定义了两个相同的 advice(比如，定义了两个 @Before)，那么这两个 advice 的执行顺序是无法确定的，哪怕你给这两个 advice 添加了 @Order 这个注解，也不行。

- 对于@Around这个advice，不管它有没有返回值，但是必须要方法内部，调用一下 pjp.proceed();否则，Controller 中的接口将没有机会被执行，从而也导致了 @Before这个advice不会被触发。


## 进过的坑
- 不能在方法methodA中调用某个方法methodB，然后对这个调用的某个方法methodB做切面，要对某个类要调用的方法做切面才有效,例如methodA
ååå