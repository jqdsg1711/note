nignx

背景：Nginx（“engine x”）一个具有高性能的【HTTP】和【反向代理】的【WEB服务器】，同时也是一个【POP3/SMTP/IMAP代理服务器】，是由伊戈尔·赛索耶夫(俄罗斯人)使用C语言编写的，Nginx的第一个版本是2004年10月4号发布的0.1.0版本。另外值得一提的是伊戈尔·赛索耶夫将Nginx的源码进行了开源，这也为Nginx的发展提供了良好的保障。
名词解释

1.WEB服务器：

WEB服务器也叫网页服务器，英文名叫Web Server，主要功能是为用户提供网上信息浏览服务。
2.HTTP

HTTP是超文本传输协议的缩写，是用于从WEB服务器传输超文本到本地浏览器的传输协议，也是互联网上应用最为广泛的一种网络协议。HTTP是一个客户端和服务器端请求和应答的标准，客户端是终端用户，服务端是网站，通过使用Web浏览器、网络爬虫或者其他工具，客户端发起一个到服务器上指定端口的HTTP请求。
3.POP3/SMTP/IMAP：

POP3(Post Offic Protocol 3)邮局协议的第三个版本，
SMTP(Simple Mail Transfer Protocol)简单邮件传输协议，
IMAP(Internet Mail Access Protocol)交互式邮件存取协议，
通过上述名词的解释，我们可以了解到Nginx也可以作为电子邮件代理服务器。
4.反向代理

正向代理![img](https://i-blog.csdnimg.cn/blog_migrate/1ccdbad6ddbe07304b152bba0067173a.png)

 反向代理

![img](https://i-blog.csdnimg.cn/blog_migrate/f40c0a7ec4bed4da624bdedf166df0ec.png)

#### Nginx的优点

(1)速度更快、并发更高

单次请求或者高并发请求的环境下，Nginx都会比其他Web服务器响应的速度更快。一方面在正常情况下，单次请求会得到更快的响应，另一方面，在高峰期(如有数以万计的并发请求)，Nginx比其他Web服务器更快的响应请求。Nginx之所以有这么高的并发处理能力和这么好的性能原因在于Nginx采用了多进程和I/O多路复用(epoll)的底层实现。

(2)配置简单，扩展性强

Nginx的设计极具扩展性，它本身就是由很多模块组成，这些模块的使用可以通过配置文件的配置来添加。这些模块有官方提供的也有第三方提供的模块，如果需要完全可以开发服务自己业务特性的定制模块。

(3)高可靠性

Nginx采用的是多进程模式运行，其中有一个master主进程和N多个worker进程，worker进程的数量我们可以手动设置，每个worker进程之间都是相互独立提供服务，并且master主进程可以在某一个worker进程出错时，快速去"拉起"新的worker进程提供服务。

(4)热部署

现在互联网项目都要求以7*24小时进行服务的提供，针对于这一要求，Nginx也提供了热部署功能，即可以在Nginx不停止的情况下，对Nginx进行文件升级、更新配置和更换日志文件等功能。

(5)成本低、BSD许可证

BSD是一个开源的许可证，世界上的开源许可证有很多，现在比较流行的有六种分别是GPL、BSD、MIT、Mozilla、Apache、LGPL。这六种的区别是什么，我们可以通过下面一张图来解释下：


![img](https://i-blog.csdnimg.cn/blog_migrate/8bda4f11a47375abfb40d43470254939.png)

Nginx本身是开源的，我们不仅可以免费的将Nginx应用在商业领域，而且还可以在项目中直接修改Nginx的源码来定制自己的特殊要求。这些点也都是Nginx为什么能吸引无数开发者继续为Nginx来贡献自己的智慧和青春。

**Nginx的功能特性及常用功能** 
Nginx提供的基本功能服务从大体上归纳为"基本HTTP服务"、“高级HTTP服务”和"邮件服务"等三大类。

**基本HTTP服务**

Nginx可以提供基本HTTP服务，可以作为HTTP代理服务器和反向代理服务器，支持通过缓存加速访问，可以完成简单的负载均衡和容错，支持包过滤功能，支持SSL等。

- 处理静态文件、处理索引文件以及支持自动索引；

- 提供反向代理服务器，并可以使用缓存加上反向代理，同时完成负载均衡和容错；

- 提供对FastCGI、memcached等服务的缓存机制，，同时完成负载均衡和容错；

- 使用Nginx的模块化特性提供过滤器功能。Nginx基本过滤器包括gzip压缩、ranges支持、chunked响应、XSLT、SSI以及图像缩放等。其中针对包含多个SSI的页面，经由FastCGI或反向代理，SSI过滤器可以并行处理。

- 支持HTTP下的安全套接层安全协议SSL.

- 支持基于加权和依赖的优先权的HTTP/2


**高级HTTP服务**

- 支持基于名字和IP的虚拟主机设置

- 支持HTTP/1.0中的KEEP-Alive模式和管线(PipeLined)模型连接

- 自定义访问日志格式、带缓存的日志写操作以及快速日志轮转。

- 提供3xx~5xx错误代码重定向功能

- 支持重写（Rewrite)模块扩展

- 支持重新加载配置以及在线升级时无需中断正在处理的请求

- 支持网络监控

- 支持FLV和MP4流媒体传输


**邮件服务**

Nginx提供邮件代理服务也是其基本开发需求之一，主要包含以下特性：

- 支持IMPA/POP3代理服务功能

- 支持内部SMTP代理服务功能


**Nginx常用的功能模块**  

```
静态资源部署
Rewrite地址重写
    正则表达式
反向代理
负载均衡
    轮询、加权轮询、ip_hash、url_hash、fair
Web缓存
环境部署
    高可用的环境
用户认证模块...
```

**Nginx的核心组成**

```
nginx二进制可执行文件
nginx.conf配置文件
error.log错误的日志记录
access.log访问日志记录 
```

```
docker run \
-p 80:80 \
--name nginx \
-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/html:/usr/share/nginx/html \
-d nginx:latest
```

##### 配置文件的结构和组成

**第一部分:全局块**

从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。 比如上面第一行配置的：

```
worker_processes  1;
```

这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约。

第二部分：events 块

```
events {
    worker_connections  1024;
}
events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 work process 可以同时支持的最大连接数等。上述例子就表示每个 work process 支持的最大连接数为 1024。
```

这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。
**第三部分：http 块**

这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。需要注意的是：http 块也可以包括 http 全局块、server 块。

**http全局块：**
http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

**server块**
这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。

每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。而每个 server 块也分为全局 server 块，以及可以同时包含多个 location 块。

**全局server块**：最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。

**location块**：一个 server 块可以配置多个 location 块。这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

```
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}

```

反向代理:效果访问192.168.88.120->192.168.88.120:8080/hello/hellotomcat.html

```
server {
    listen       80;
    listen  [::]:80;
    server_name  192.168.88.120;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        proxy_pass http://192.168.88.120:8080/hello/hellotomcat.html;
	    index  index.html index.htm;
	
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

反向代理2：

```
server{
     listen  9001;
     server_name  192.168.88.120;
     
     location ~ /edu/ {
     proxy_pass http://192.168.88.120:8080;
     }
     
     location ~ /vod/ {
     proxy_pass http://192.168.88.120:8081;
     }
}
```

负载均衡

1.轮询（默认）

```
upstream myserver{
	server 192.168.88.120:8080;
	server 192.168.88.120:8081;
}

server{
     listen  9001;
     server_name  192.168.88.120;
     
     location / {
     proxy_pass http://mysever;
     root html;
     index index.html index.htm;
     }

}
```

2.权重

```
upstream myserver{
	server 192.168.88.120:8080 weight = 10;
	server 192.168.88.120:8081 weight = 20;
}

server{
     listen  9001;
     server_name  192.168.88.120;
     
     location / {
     proxy_pass http://mysever;
     root html;
     index index.html index.htm;
     }

}
```

3.ip_hash

每个请求按照ip的hash结果进行分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

```
upstream myserver{
	ip_hash;
	server 192.168.88.120:8080;
	server 192.168.88.120:8081;
}

server{
     listen  9001;
     server_name  192.168.88.120;
     
     location / {
     proxy_pass http://mysever;
     root html;
     index index.html index.htm;
     }

}
```

4.fair(第三方)

按照后端服务器的响应时间来分配请求，相应时间短的优先分配

```
upstream myserver{
	server 192.168.88.120:8080;
	server 192.168.88.120:8081;
	fair;
}

server{
     listen  9001;
     server_name  192.168.88.120;
     
     location / {
     proxy_pass http://mysever;
     root html;
     index index.html index.htm;
     }

}
```

## 动静分离

在Web开发中，通常来说，动态资源其实就是指那些后台资源，而静态资源就是指HTML，JavaScript，CSS，img等文件。

动静分离，说白了，就是将网站静态资源（HTML，JavaScript，CSS，img等文件）与后台应用分开部署，提高用户访问静态代码的速度，降低对后台应用服务器的请求。后台应用服务器只负责动态数据请求。



**优势**：分担负载，减轻web服务器的压力，适用于大负载。静态资源放置cdn，同时还可以通过配置缓存到客户浏览器中，这样极大减轻web服务器的压力。



**劣势**：网络环境不佳时，ajax回应很慢，导致页面出现空白，出错处理会不好看。不利于网站SEO（搜索引擎优化），增加了开发复杂度。

#### 实现

动静分离就是根据一定规则静态资源的请求全部请求Nginx服务器，后台数据请求转发到Web应用服务器上。从而达到动静分离的目的。目前比较流行的做法是将静态资源部署在Nginx上，而Web应用服务器只处理动态数据请求。这样减少Web应用服务器的并发压力。具体如下图所示：

![1633743657(1).png](https://ucc.alicdn.com/pic/developer-ecology/dacaac09151c45178d3f334ed469436c.png?x-oss-process=image%2Fresize%2Cw_1400%2Fformat%2Cwebp)

配置Nginx动静分离

1. 修改nginx.conf配置，其中第一个location负责处理后台请求，第二个负责处理静态资源， nginx 的其他配置，请参考前之前的文章。 具体如下所示：

```
 #拦截静态资源
      location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
        root static;
        expires      30d;  
       }          
       
```

上面的示例，主要是配置image、js、css等资源文件的路径和地址。然后设置缓存失效的时间。



完成的Nginx配置如下所示：

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {

   server {
       listen       80;
       server_name  localhost;
      
      #拦截后台请求
      location / {
        proxy_pass http://localhost:81;
        proxy_set_header X-Real-IP $remote_addr;
      }

      #拦截静态资源
      location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
        root static;
        expires      30d;  
       }

    }

}
```

 

2. 在Nginx 下 创建 static 目录，将图片，js, css 等文件 拷贝到该目录下

![1633744420(1).png](https://ucc.alicdn.com/pic/developer-ecology/27260c93eef749738dd87104f3ecc3e6.png?x-oss-process=image%2Fresize%2Cw_1400%2Fformat%2Cwebp)

3. 重启Nginx，使用命令： ./nginx -s reload 重新启动Nginx。

#### 验证测试

![image.png](https://ucc.alicdn.com/pic/developer-ecology/ba9efbdba2bc4b20982a6afde86bc826.png?x-oss-process=image%2Fresize%2Cw_1400%2Fformat%2Cwebp)

通过浏览器的调试工具，通过图中红框内容都可以看出来引用静态资源成功了。动态请求转发到了81端口的web应用服务器，而静态的资源文件，访问的是80端口。说明Nginx的动静分离配置成功。
