#Nginx配置负载均衡、动静分离、请求日志打印、日志切割、压缩、删除
近期公司有需求，在用户访问我们的API产品时，需要使用nginx的日志打印功能，将详细的信息记录下来，由于本人第一次做这样的操作，研究了一下，将整理出来的内容出来。



### 目录
- <ul><li>- <ul><li>- - - - - - - - - - - 


## 1、编写并且启动nginx（windows环境下，linux配置文件一致）

### 1.1、一些必要的命令

检查nginx是否启动。

```
tasklist /fi "imagename eq nginx.exe"	

```

检查80端口是否被占用

```
netstat -ano | findstr 0.0.0.0:80 或 netstat -ano | findstr "80"    	

```

重新加载配置文件nginx.conf（常用于修改了配置文件）

```
nginx -s reload

```

重启nginx

```
systemctl restart nginx

```

**`注意：每次修改了配置文件，都需要重启nginx或重载一下配置文件`**

### 1.2、下载nginx

下载地址： ps:下载的时候，选择中间带有/windows的 Stable version 稳定版即可，我这里下载的是1.18.0版本。文件为zip压缩包，直接解压（注意，这里不要双击打开无效，规范的解压方式应该是：单击压缩包，右击，选择解压到xxx文件名目录）。

**`注意：文件目录的路径不能存在中文。否则可能存在无法启动的情况！`**

解压后，文件目录如下： <img src="https://img-blog.csdnimg.cn/20200806152224728.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2tfamlud2Vp,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

这里我们需要知道几个地方： 1、config文件夹：这里面主要注意nginx.conf文件，其他的如果只是用nginx做负载均衡，基本不用管。 2、logs文件夹：默认情况下，nginx输出的日志文件access.log、error.log、nginx.pid会随着启动放在这里。 3、nginx.exe：windows版nginx启动文件。

### 1.3、启动nginx

有多种启动nginx的方法： 1、双击nginx.exe，会有一个黑色的类似于cmd窗口的弹窗，一闪而过。 2、win+r打开cmd命令行窗口，切换到nginx解压后的目录下，输入命令 nginx.exe 或 start nginx，回车即可。

### 1.4、检查nginx是否启动

有多种检查方式，以下介绍两种： 1、在浏览器地址栏输入http://localhost:80，回车，会出现nginx默认页面，即启动成功。 <img src="https://img-blog.csdnimg.cn/20200806153425151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2tfamlud2Vp,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> 注意：nginx配置文件config目录下的nginx.config中，默认配置的nginx监听端口为80，如果被占用可以去修改下。 检查80端口是否被占用的命令是： netstat -ano | findstr 0.0.0.0:80 或 netstat -ano | findstr “80” <img src="https://img-blog.csdnimg.cn/20200806154135848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2tfamlud2Vp,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> 当我们修改了nginx的配置文件nginx.conf 时，不需要关闭nginx后重新启动nginx，只需要执行命令 nginx -s reload 即可让改动生效。

2、在cmd命令行窗口输入：

```
tasklist /fi "imagename eq nginx.exe"

```

回车，出现以下界面，则启动成功。 <img src="https://img-blog.csdnimg.cn/2020080615392244.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2tfamlud2Vp,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

### 1.5、关闭nginx

如果使用cmd命令窗口启动nginx，关闭cmd窗口是不能结束nginx进程的。 可使用两种方法关闭nginx： 在nginx目录下： 1、使用taskkill taskkill /f /t /im nginx.exe 2、输入nginx命令 nginx -s quit(完整有序的停止nginx) 或 nginx -s stop(快速停止nginx)

### 1.6、负载均衡配置

nginx用的最多的场景就是负载均衡，用极少的内存占用，有效都提升高并发性能。 配置也很简单，我这边取了一个windows版的tomcat，在webapp下建了一个html文件，随便写几个字表示访问到了这个页面即可。 我们通过修改nginx的配置文件nginx.conf，达到访问nginx代理服务器时，自动跳转到指定服务器（一个或多个）的目的，即通过proxy_pass 配置请求转发地址，即当我们依然输入http://localhost:80 时，请求会跳转到我们配置的服务器。 <img src="https://img-blog.csdnimg.cn/20200806161442679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2tfamlud2Vp,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

当然，upstream里面也可以配置更多的目标机器，以达到负载均衡的目的，同时，当一台服务器出现故障时，nginx能将请求自动转向另一台服务器。

还可以配置fail_timeout超时时间、weight权重等。这里不做具体说明。

### 1.7、配置动静分离策略

通过中间件将动态请求和静态请求分离，可以建少不必要的请求消耗，同时能减少请求的延时。 目前来看，动静分离只有好处：动静分离后，即使动态服务不可用，但静态资源不会受到影响。

配置策略，在固定目录下，新建静态资源文件夹，用于存放静态资源。如图片、固定不变的首页等等。 配置如下： <img src="https://img-blog.csdnimg.cn/20200806162539565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2tfamlud2Vp,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

此时，通过web可以直接访问此静态资源，如：localhost:80/1.png，即可访问到位于D:/static目录下的1.png文件了。

## 2、配置nginx日志

通过配置nginx日志，能更直观的感受、了解到项目的请求情况。

### 2.1、启用nginx日志打印

在nginx的配置文件nginx.conf中，取消#注释即开启了日志打印功能，默认存放于nginx目录下的logs目目录下。 如下： <img src="https://img-blog.csdnimg.cn/20200806163137380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2tfamlud2Vp,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> 这里面有很多的参数，后面有详细的说明，这里只是用于演示怎么做。

### 2.2、配置日志打印格式为json

配置成json格式，能更加方便我们去查看。 在在http模块的log_format中增加 **escape=json** 如下： <img src="https://img-blog.csdnimg.cn/20200806163827478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2tfamlud2Vp,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> 代码片如下：

```
	log_format  main escape=json '{ "@timestamp": "$time_iso8601",'
								'"remote_addr":"$remote_addr",'
								'"remote_user":"$remote_user",'
								'"time_local":"$time_local",'
								'"request":"$request",'
								'"status":"$status",'
								'"body_bytes_sent":"$body_bytes_sent",'
								'"http_referer":"$http_referer",'
								'"http_user_agent":"$http_user_agent",'
								'"http_x_forwarded_for":"$http_x_forwarded_for" }';

```

格式如下： 其实除了开头和结尾以外，都是同样的格式了，注意逗号和分号的位置。

```
	log_format  main escape=json '{	"  " :  "  "	,'
								  '	"  " :  "  "	,'
								  ' "  " :  "  "	}';

```

但是要注意，**`这样修改之后，在默认的日志路径下，是不生效的，必须要修改到指定位置`**，当然，业务中也应该修改日志存放位置到统一的位置下，便于管理。

### 2.3、配置日志文件存放位置

直接在nginx.conf配置文件中，启用即可（此处放的是linux环境下的路径，windows自己替换下）。

```
    access_log  /home/logs/nginx/access.log  main;

```

error_log文件同样，在配置文件的顶部，同样需要配置下。后面的日志切割需要用到。 pid文件同样，需要自己增加下配置。

```
error_log /home/logs/nginx/error.log;
pid        /home/logs/nginx/nginx.pid;

```

启动nginx，尝试进行访问，并查看切换日志文件目录后，产生的日志文件，是否已经打印出相应的内容了。

## 3、配置nginx的自动切割与压缩

日志文件包含了关于系统中发生的事件的有用信息，在排障过程中或者系统性能分析时经常被用到。对于忙碌的服务器，日志文件大小会增长极快，服务器会很快消耗磁盘空间，这成了个问题。除此之外，处理一个单个的庞大日志文件也常常是件十分棘手的事。

logrotate是个十分有用的工具，它可以自动对日志进行截断（或轮循）、压缩以及删除旧的日志文件。例如，你可以设置logrotate，让/var/log/foo日志文件每30天轮循，并删除超过6个月的日志。配置完后，logrotate的运作完全自动化，不必进行任何进一步的人为干预。

### 3.1、配置切割方式

切换到/etc/logrotate.d目录下，打开nginx文件(不存在则新建)，添加内容：

```
/data/aieye/nginxLongs/*.log
{<!-- -->
    daily			#按天切割日志，其它可用值为'daily'，'weekly'或者'yearly'。
    rotate 30		#保留30个文件，时间最久的第31个会被删除
    missingok		#忽略错误
    dateext			#以日期为单位 默认-%Y%m%d
    compress		#压缩文件
    delaycompress	#延迟一天压缩文件
    notifempty		#空文件不切割
    sharedscripts
    postrotate		#在所有其它指令完成后，postrotate和endscript里面指定的命令将被执行。
    				#在这种情况下，rsyslogd 进程将立即再次读取其配置并继续运行。
       if [ -f /usr/local/webserver/nginx/logs/nginx.pid ]; then
        kill -USR1 `cat /usr/local/webserver/nginx/logs/nginx.pid`
       fi
    endscript
}


```

**此时，已经配置完毕。但是此时系统并不会去执行。**

### 3.2、启动分割的方式方法

1、手动执行： 刚开始添加的时候，使用命令手动执行，查看是否正常，日志中需要有一定的内容，根据上面的配置，日志为空不切割。

```
logrotate -vf /etc/logrotate.d/nginx

```

2、定时任务

使用crontab -e 命令编辑，并添加一行新的任务

```
crontab -e

```

#以下为定时任务cron的执行时间格式，可以自己根据需求调整。 #每天0时1分

```
01 00 * * * /usr/local/nginx/logjob.sh

```

至此，nginx的日志自定义内容打印、日志的切割、自动压缩、自动删除等已经完成。

## 4、nginx配置文件详细说明

```
...              #全局块

events {<!-- -->         #events块
   ...
}

http      #http块
{<!-- -->
    ...   #http全局块
    server        #server块
    {<!-- --> 
        ...       #server全局块
        location [PATTERN]   #location块
        {<!-- -->
            ...
        }
        location [PATTERN] 
        {<!-- -->
            ...
        }
    }
    server
    {<!-- -->
      ...
    }
    ...     #http全局块
}

```
1. 全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。1. events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。1. http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。1. server块：配置虚拟主机的相关参数，一个http中可以有多个server。1. location块：配置请求的路由，以及各种页面的处理情况。
以下为基础配置文件详细说明：

```
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {<!-- -->
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {<!-- -->
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {<!-- -->   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {<!-- -->
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {<!-- -->       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}

```

## 5、nginx配置常见变量参数及获取请求头中参数方式方法

```
$args                    #请求中的参数值
$query_string            #同 $args
$arg_NAME                #GET请求中NAME的值
$is_args                 #如果请求中有参数，值为"?"，否则为空字符串
$uri                     #请求中的当前URI(不带请求参数，参数位于$args)，可以不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改，$uri不包含主机名，如"/foo/bar.html"。
$document_uri            #同 $uri
$document_root           #当前请求的文档根目录或别名
$host                    #优先级：HTTP请求行的主机名&gt;"HOST"请求头字段&gt;符合请求的服务器名.请求中的主机头字段，如果请求中的主机头不可用，则为服务器处理请求的服务器名称
$hostname                #主机名
$https                   #如果开启了SSL安全模式，值为"on"，否则为空字符串。
$binary_remote_addr      #客户端地址的二进制形式，固定长度为4个字节
$body_bytes_sent         #传输给客户端的字节数，响应头不计算在内；这个变量和Apache的mod_log_config模块中的"%B"参数保持兼容
$bytes_sent              #传输给客户端的字节数
$connection              #TCP连接的序列号
$connection_requests     #TCP连接当前的请求数量
$content_length          #"Content-Length" 请求头字段
$content_type            #"Content-Type" 请求头字段
$cookie_name             #cookie名称
$limit_rate              #用于设置响应的速度限制
$msec                    #当前的Unix时间戳
$nginx_version           #nginx版本
$pid                     #工作进程的PID
$pipe                    #如果请求来自管道通信，值为"p"，否则为"."
$proxy_protocol_addr     #获取代理访问服务器的客户端地址，如果是直接访问，该值为空字符串
$realpath_root           #当前请求的文档根目录或别名的真实路径，会将所有符号连接转换为真实路径
$remote_addr             #客户端地址
$remote_port             #客户端端口
$remote_user             #用于HTTP基础认证服务的用户名
$request                 #代表客户端的请求地址
$request_body            #客户端的请求主体：此变量可在location中使用，将请求主体通过proxy_pass，fastcgi_pass，uwsgi_pass和scgi_pass传递给下一级的代理服务器
$request_body_file       #将客户端请求主体保存在临时文件中。文件处理结束后，此文件需删除。如果需要之一开启此功能，需要设置client_body_in_file_only。如果将次文件传 递给后端的代理服务器，需要禁用request body，即设置proxy_pass_request_body off，fastcgi_pass_request_body off，uwsgi_pass_request_body off，or scgi_pass_request_body off
$request_completion      #如果请求成功，值为"OK"，如果请求未完成或者请求不是一个范围请求的最后一部分，则为空
$request_filename        #当前连接请求的文件路径，由root或alias指令与URI请求生成
$request_length          #请求的长度 (包括请求的地址，http请求头和请求主体)
$request_method          #HTTP请求方法，通常为"GET"或"POST"
$request_time            #处理客户端请求使用的时间,单位为秒，精度毫秒； 从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。
$request_uri             #这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI，不包含主机名，例如："/cnphp/test.php?arg=freemouse"
$scheme                  #请求使用的Web协议，"http" 或 "https"
$server_addr             #服务器端地址，需要注意的是：为了避免访问linux系统内核，应将ip地址提前设置在配置文件中
$server_name             #服务器名
$server_port             #服务器端口
$server_protocol         #服务器的HTTP版本，通常为 "HTTP/1.0" 或 "HTTP/1.1"
$status                  #HTTP响应代码
$time_iso8601            #服务器时间的ISO 8610格式
$time_local              #服务器时间（LOG Format 格式）
$cookie_NAME             #客户端请求Header头中的cookie变量，前缀"$cookie_"加上cookie名称的变量，该变量的值即为cookie名称的值
$http_NAME               #匹配任意请求头字段；变量名中的后半部分NAME可以替换成任意请求头字段，如在配置文件中需要获取http请求头："Accept-Language"，$http_accept_language即可
$http_cookie
$http_host               #请求地址，即浏览器中你输入的地址（IP或域名）
$http_referer            #url跳转来源,用来记录从那个页面链接访问过来的
$http_user_agent         #用户终端浏览器等信息
$http_x_forwarded_for
$sent_http_NAME          #可以设置任意http响应头字段；变量名中的后半部分NAME可以替换成任意响应头字段，如需要设置响应头Content-length，$sent_http_content_length即可
$sent_http_cache_control
$sent_http_connection
$sent_http_content_type
$sent_http_keep_alive
$sent_http_last_modified
$sent_http_location
$sent_http_transfer_encoding

```

获取请求头中的一些参数，只需要通过$http_参数名即可获取，可以直接放在日志中打印。

```
$http_参数名

```

## 6、nginx日志切割常见配置

```
# 配置		    # 说明
daily			# 指定转储周期为每天
weekly			# 指定转储周期为每周
monthly			# 指定转储周期为每月
rotate count	# 指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
compress		# 通过gzip 压缩转储以后的日志
nocompress		# 不做gzip压缩处理
create mode owner group	# 轮转时指定创建新文件的属性，如create 0777 nobody nobody
nocreate		# 不建立新的日志文件
delaycompress	# 和compress 一起使用时，转储的日志文件到下一次转储时才压缩
nodelaycompress	# 覆盖 delaycompress 选项，转储同时压缩
missingok		# 如果日志丢失，不报错继续滚动下一个日志
ifempty			# 即使日志文件为空文件也做轮转，这个是logrotate的缺省选项
notifempty		# 当日志文件为空时，不进行轮转
mail address	# 把转储的日志文件发送到指定的E-mail 地址
olddir directory # 转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
noolddir		# 转储后的日志文件和当前日志文件放在同一个目录下
sharedscripts	# 运行postrotate脚本，作用是在所有日志都轮转后统一执行一次脚本。如果没有配置这个，那么每个日志轮转后都会执行一次脚本
prerotate		# 在logrotate转储之前需要执行的指令，例如修改文件的属性等动作；必须独立成行
postrotate		# 在logrotate转储之后需要执行的指令，例如重新启动 (kill -HUP) 某个服务！必须独立成行
dateext			# 使用当期日期作为命名格式
dateformat .%s	# 配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，必须配合dateext使用，只支持 %Y %m %d %s 这四个参数
size(minsize) log-size	# 当日志文件到达指定的大小时才转储，log-size能指定bytes(缺省)及KB (sizek)或MB(sizem)，例如 size 100M


```

## 7、参考文章

>  
 https://www.cnblogs.com/clsn/p/8428257.html https://www.cnblogs.com/you-men/p/12827117.html https://www.cnblogs.com/houchaoying/p/8806787.html https://www.runoob.com/w3cnote/nginx-setup-intro.html 


**`感谢大佬无私奉献！`**
