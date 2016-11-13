---
title: Retrofit2介绍
date: 2016-10-17 13:19:39
categories:
- Android技术
tags:
- 框架
---
# Retrofit2介绍

## RESTful 
在开始了解 Retrofit 的使用之前，我们需要理解RESTful概念因为 Retrofit 的初衷就是根据 RESTful风格的API 来进行封装的。关于RESTful 我们可以参考
[《RESTful API 设计指南》](http://www.androidchina.net/3749.html)

## Retrofit
Retrofit与okhttp共同出自于Square公司，retrofit就是对okhttp做了一层封装。把网络请求都交给给了Okhttp，我们只需要通过简单的配置就能使用retrofit来进行网络请求了。官方介绍：[http://square.github.io/retrofit/](http://square.github.io/retrofit/)

### 引用

- Gradle：
```
compile 'com.squareup.retrofit2:retrofit:2.1.0'
```

### 使用

学习一门新的技术的时候，我们首先要参考的就是官方文档，我们看看官方文档是怎么写的：[http://square.github.io/retrofit/](http://square.github.io/retrofit/)

大家看到官方文档，很是简单粗暴，一言不合就上代码

```Java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}

```
从上面代码我们可知，Retrofit将HTTP API转换为interface接口的方式（Retrofit turns your HTTP API into a Java interface.）

```Java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);

```

Retrofit 根据之前定义的接口生成一个具体的实现。通俗的讲就是我们可以将Retrofit类看作是一个“工厂类”的角色，我们在接口中提供了此次的“产品”的生产规格信息，而Retrofit则通过信息负责为我们生产。

```Java
Call<List<Repo>> repos = service.listRepos("octocat");

```
"Call"：通过之前封装的请求接口对象创建的任一的Call都可以发起一个同步（或异步）的HTTP请求到远程服务器。

以上信息就是简单的Retrofit的操作，通过这个实例感受最多的就是解耦明确；使用简单。通过注解的方式描述request让人眼前一亮。

### 其他注解
- @Path

```Java
public interface IDemoBiz
{
 @GET("modify/{userId}")
 Call<User> modify(@Path("userId") String userId);
}
```

- @Query
对于@GET来说，参数信息是可以直接放在url中上传的。那么你马上就反应过来了，这一样也存在严重的耦合！于是，就有了 @Query

```Java
public interface IDemoBiz
{
 @GET("modify/{userId}")
 Call<User> modify(@Path("userId") String userId,@Query("name") String name);
}
```

- @QueryMap
假设我要在参数中上传10个参数呢？

```Java
public interface IDemoBiz
{
 @GET("modify/{userId}")
 Call<User> modify(@Path("userId") String userId,@QueryMap Map<String,String> params);
}
```


- @Post @Body Post请求上传body

```Java
public interface IDemoBiz
{
 @POST("add")
 Call<List<User>> get(@Body User user);
}
```

- @Post @FormUrlEncoded 表单上传

***服务器通过request.getParameter的方式直接读取参数信息***

```Java
public interface IDemoBiz
{
  @POST("modify/{userId}")
 Call<User> modify(@Path("userId") String userId,@Field("name") String name,@Field("nickname") String nickname);
}
```

- @Headers与@Header

可以设置HTTP的header，拿使用编码来举例

```Java
public interface IDemoBiz
{
  @Headers("Content-type:application/x-www-form-urlencoded;charset=UTF-8")
  @FormUrlEncoded
  @POST("modify/{userId}")
 Call<User> modify(@Path("userId") String userId,@Field("name") String name,@Field("nickname") String nickname);
}

```

上面代码说的使用@Headers的用法，那么@Header怎么使用呢？@Header与@Headers不同在于是动态的添加请求头信息。

```Java
 @FormUrlEncoded
  @POST("modify/{userId}")
 Call<User> modify(@Header("Content-type" String contentType),@Path("userId") String userId,@Field("name") String name,@Field("nickname") String nickname);
```
HTTP的Header也可以在okhttp的拦截器中添加。

- 其他

当然我们还可以进行文件上传和下载，这里就不过多介绍了，网上有很多示例。



### 注意事项
***特别注意如果需要继承rxjava和gson转换，则还需要引入***

```
   compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
```

```
   compile 'com.squareup.retrofit2:converter-gson:2.1.0'
```
***其他格式转换引入***

```
Gson: com.squareup.retrofit2:converter-gson
Jackson: com.squareup.retrofit2:converter-jackson
Moshi: com.squareup.retrofit2:converter-moshi
Protobuf: com.squareup.retrofit2:converter-protobuf
Wire: com.squareup.retrofit2:converter-wire
Simple XML: com.squareup.retrofit2:converter-simplexml
Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars

```
***如果不添加转换器会报IllegalArgumentException异常,这也会是新手开发碰到的第一个坑***

