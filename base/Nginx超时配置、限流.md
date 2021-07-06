

Author：Jinwei

EditTimes：2020年11月25日17:31:06



# 一、说明

 Nginx 处理的每个请求均有相应的超时设置。如果做好这些超时时间的限定，判定超时后资源被释放，用来处理其他的请求，以此提升 Nginx 的性能。



# 二、超时配置

1、**keepalive_timeout**

HTTP 是一种无状态协议，客户端向服务器发送一个 TCP 请求，服务端响应完毕后断开连接。

如果客户端向服务器发送多个请求，每个请求都要建立各自独立的连接以传输数据。

HTTP 有一个 KeepAlive 模式，它告诉 webserver 在处理完一个请求后保持这个 TCP 连接的打开状态。若接收到来自客户端的其它请求，服务端会利用这个未被关闭的连接，而不需要再建立一个连接。

KeepAlive 在一段时间内保持打开状态，它们会在这段时间内占用资源。占用过多就会影响性能。

Nginx 使用 keepalive_timeout 来指定 KeepAlive 的超时时间（timeout）。指定每个 TCP 连接最多可以保持多长时间。Nginx 的默认值是 75 秒，有些浏览器最多只保持 60 秒，所以可以设定为 60 秒。若将它设置为 0，就禁止了 keepalive 连接。通常keepalive_timeout应该比client_body_timeout大

```
# 配置段: http, server, location
keepalive_timeout 60s;
```



2、**client_body_timeout**

指定客户端与服务端建立连接后发送 request body 的超时时间。如果客户端在指定时间内没有发送任何内容，Nginx 返回 HTTP 408（Request Timed Out）。

```
# 配置段: http, server, location
client_body_timeout 20s;
```



3、**client_header_timeout**

客户端向服务端发送一个完整的 request header 的超时时间。如果客户端在指定时间内没有发送一个完整的 request header，Nginx 返回 HTTP 408（Request Timed Out）。

```
# 配置段: http, server, location
client_header_timeout 10s;
```

4、**send_timeout**

服务端向客户端传输数据的超时时间。这个设置不会用于整个转发器，而是在两次客户端读取操作之间。如果在这段时间内，客户端没有读取任何数据，Nginx就会关闭连接。

```
# 配置段: http, server, location
send_timeout 30s;
```



5、**proxy转发模块的超时设置**

5.1、proxy_connect_timeout**
		语法 proxy_connect_timeout time 
		默认值 60s
		上下文 http server location
		说明 该指令设置与upstream server的连接超时时间，有必要记住，这个超时不能超过75秒。
		这个不是等待后端返回页面的时间，那是由proxy_read_timeout声明的。如果你的upstream服务器起来了，但是hanging住了（例如，没有足够的线程处理请		求，所以把你的请求放到请求池里稍后处理），那么这个声明是没有用的，因为与upstream服务器的连接已经建立了。

5.2、**proxy_read_timeout**

语法 proxy_read_timeout time 
		默认值 60s
		上下文 http server location
		说明 该指令设置与代理服务器的读超时时间。它决定了Nginx会等待多长时间来获得请求的响应。这个时间不是获得整个response的时间，而是两次reading		操作的时间。（？？什么是两次reading操作的时间）

5.3、**proxy_send_timeout**
		语法 proxy_send_timeout time 
		默认值 60s
		上下文 http server location
		说明 这个指定设置了发送请求给upstream服务器的超时时间。超时设置不是为了整个发送期间，而是在两次write操作期间。如果超时后，upstream没有收到		新的数据，Nginx会关闭连接

5.4、**proxy_upstream_fail_timeout（fail_timeout）**
		语法 server address [fail_timeout=30s]
		默认值 10s
		上下文 upstream
		说明 Upstream模块下 server指令的参数，设置了某一个upstream后端失败了指定次数（max_fails）后，该后端不可操作的时间，默认为10秒



# 三、限流

限流：是用于保护服务器不会因为承受不住同一时刻的大量并发请求而异常。

Nginx限流主要分为两种方式：

-   限制访问频率

-   限制并发连接数

    

### 限制访问频率

限制访问频率其实需要分成两种情况：正常情况下进行访问频率限制以及流量突发情况下进行访问频率限制。

**1、正常流量限制访问频率**
Nginx中使用ngx_http_limit_req_module模块来限制的访问频率，限制的原理实质是基于漏桶算法原理来实现的。在nginx.conf配置文件中可以使用limit_req_zone命令及limit_req命令限制单个IP的请求处理频率。

语法结构：

-   limit_req_zone key zone rate

对于上面语法结构的参数简单做下解释：

-   key: 定义需要限流的对象。

-   zone: 定义共享内存区来存储访问信息。

-   rate: 用于设置最大访问速率。

    

```bash
http {
	limit_req_zone $binary_remote_addr zone=myLimit:10m rate=3r/s;
}
server {
    location / {
        limit_req zone=myLimit;
        rewrite / https://www.baidu.com permanent;
    }
}
```

对配置简单做下解释：
		上面binary_remote_addr就是key，表示基于客户端ip（remote_addr，Nginx基础配置）进行限流，binary_表示压缩内存占用量。定义了一个大小为10M，名称为myLimit的内存区（1M能存储16000 IP地址的访问信息，10M可以存储16W IP地址访问信息），用于存储IP地址访问信息。

rate设置IP访问频率，rate=5r/s表示每秒只能处理每个IP地址的5个请求。Nginx限流是按照毫秒级为单位的，也就是说1秒处理5个请求会变成每200ms只处理一个请求。如果200ms内已经处理完1个请求，但是还是有有新的请求到达，这时候Nginx就会拒绝处理该请求。



**2、突发流量限制访问频率**
上面的配置一定程度可以限制访问频率，但是也存在着一个问题：如果突发流量超出请求被拒绝处理，无法处理活动时候的突发流量，这时候应该如何进一步处理呢？Nginx提供burst参数结合nodelay参数可以解决流量突发的问题，可以设置能处理的超过设置的请求数外能额外处理的请求数。我们可以将之前的例子添加burst参数以及nodelay参数：

语法结构：

-   limit_req zone=ip_limit burst=5 nodelay;

对于上面语法结构的参数简单做下解释：

-   zone=ip_limit对应上面的限流对象地址；
-   bursrt=5表示最大支持5的并发，会有2个请求存在队列中，不会被拒绝；
-   nodelay: 表示并发超出的5个请求，不会被延迟，会被立即执行，如需等待执行的话，配置delay即可。

```bash
http {
	limit_req_zone $binary_remote_addr zone=myLimit:10m rate=3r/s;
}

server {
	location / {
		limit_req zone=myLimit burst=5 nodelay;
		rewrite / http://www.niyueling.cn permanent;
	}
}
```

在原有的location中的limit_req指令中添加了burst=5 nodelay，如果没有添加nodelay参数，则可以理解为预先在内存中占用了5个请求的位置，如果有5个突发请求就会按照200ms去依次处理请求，也就是1s内把5个请求全部处理完毕。如果1s内有新的请求到达也不会立即进行处理，因为紧急程度更低。这样实际上就会将额外的5个突发请求以200ms/个去依次处理，保证了处理速率的稳定，所以在处理突发流量的时候也一样可以正常处理。如果添加了nodelay参数则表示要立即处理这5个突发请求。





### 限制并发连接数
Nginx中的ngx_http_limit_conn_module模块提供了限制并发连接数的功能，可以使用limit_conn_zone指令以及limit_conn执行进行配置。

**说明：**使用nginx的ngx_http_limit_conn_module模块进行限制的，涉及命令limit_conn_zone 和 limit_conn；

limit_conn  perip  10：对应的key是 $binary_remote_addr，表示限制单个IP同时最多能持有10个连接。

limit_conn  perserver  100：对应的key是 $server_name，表示虚拟主机(server) 同时能处理并发连接的总数，即该服务能承受的最大并发数。

注意，只有当 request header 被后端server处理后，这个连接才进行计数。

```bash
http {
   limit_conn_zone $binary_remote_addr zone=perip:10m;
	limit_conn_zone $server_name zone=perserver:10m;	
}

server {
    location / {
        limit_conn perip 10;
   		limit_conn perserver 100;
        rewrite / http://www.niyueling.cn permanent;
    }
}
```

上面配置了单个IP同时并发连接数最多只能10个连接，并且设置了整个虚拟服务器同时最大并发数最多只能100个链接。



# 四、问题记录

1、Nginx超过1分钟后返回404 Nginx页面

```
在Nginx配置中，使用了proxy模块，此时proxy模块的默认超时时间是60s，当超过这个时间就会返回404 Nginx问题。
此时只需要对proxy模块超时时间进行重新设置即可：
keepalive_timeout   60s;
proxy_connect_timeout       300s;
proxy_send_timeout          300s;
proxy_read_timeout          300s;
send_timeout                30s;	
```



# 五、参考文章

>    [Nginx的超时keeplive_timeout配置详解_lexsaints-CSDN博客](https://blog.csdn.net/weixin_42350212/article/details/81123932)
>
>   [nginx中的超时配置 - 开始认识 - 博客园 (cnblogs.com)](https://www.cnblogs.com/kaishirenshi/p/11425918.html)
>
>   [nginx限流配置_会飞的猪-CSDN博客_nginx限流配置](https://blog.csdn.net/zhujuntiankong/article/details/103559476)
>
>   [Nginx限流 - 逆月翎 - 博客园 (cnblogs.com)](https://www.cnblogs.com/niyueling/p/11572003.html)