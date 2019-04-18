---
title: nginx重试机制的深坑
date: 2019-04-18 17:45:43
categories:
- 后端技术
tags:
- 深坑解决
---

> 写了一个批量发短信的方法，为了方便，通过接口请求的方式进行调用，在调用过程中由于发送数量多，造成请求处理超时，导致了nginx重试机制，最终导致短信多次重发。血淋淋的教训啊~

## nginx

公司的服务和大多数公司服务一样，都采用多节点的方式部署，并使用nginx进行负载均衡。nginx是常用的一种HTTP和反向代理服务器，支持容错和负载均衡，其中nginx的重试机制就是容错的一种。

在nginx的配置中，proxy_next_upstream 和proxy_next_upstream_tries是重试配置，其中proxy_next_upstream给出了在什么情况下进行重试，官方文档给出的说明是：


```
Syntax: proxy_next_upstream error | timeout | invalid_header | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | off ...;  
Default:    proxy_next_upstream error timeout;  
Context:    http, server, location 
```

根据文档所知，当请求服务器发生错误或超时时，会尝试到下一台服务器。proxy_next_upstream_tries官方给出的文档是：

```
Syntax: proxy_next_upstream_tries number;  
Default:    proxy_next_upstream_tries 0;  
Context:    http, server, location  
This directive appeared in version 1.7.5.  
```

proxy_next_upstream_tries配置决定最多重试多少次，0表示不限制。我们的nginx没有找到该配置项，默认尝试了3次。

好了，下面看下破案现场吧

<img src="/img/java/ken/nginxretry.jpeg" />

可以看到超时时间是60秒，超时之后，都会向之前没有重试过的服务器再次请求。


## 总结

场景：请求一个接口，处理时间较长（如1分钟），且为同步操作（即处理完成后才返回结果）。若nginx配置的响应等待时间（proxy_read_timeout）为60秒，就会触发超时重试，将请求又打到另一台。如果处理中没有考虑到重复数据的场景，就会发生数据多次重复调用！所以下次如有这种情况可以使用IP:Port的方式进行操作，请注意你的姿势。
