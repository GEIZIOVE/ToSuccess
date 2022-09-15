# Nginx

## Nginx简介

### 概述

Nginx (“engine x”) 是一个高性能的 HTTP 和反向代理服务器,特点是占有内存少，并发能力强，事实上 nginx 的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用 nginx 网站用户有：百度、京东、新浪、网易、腾讯、淘宝等

### Nginx 作为 web 服务器

Nginx 可以作为静态页面的 web 服务器，同时还支持 CGI 协议的动态语言，比如 perl、php等。但是不支持 java。Java 程序只能通过与 tomcat 配合完成。Nginx 专为性能优化而开发，性能是其最重要的考量,实现上非常注重效率 ，能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数。

https://lnmp.org/nginx.html

### 正向代理

需要在客户端配置代理服务器进行指定网站访问

[![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70.png)](https://img-blog.csdnimg.cn/20200316112500861.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200316112500861.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 反向代理

暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。

[![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16629711716361.png)](https://img-blog.csdnimg.cn/2020031611252820.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[在这里插入图片描述](https://img-blog.csdnimg.cn/2020031611252820.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 负载均衡

增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡

[![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16629711716362.png)](https://img-blog.csdnimg.cn/20200316112557925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200316112557925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

[![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16629711716363.png)](https://img-blog.csdnimg.cn/20200316112647985.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200316112647985.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 高可用(失败迁移)

高可用（High Availability）是系统架构设计中必须考虑的因素之一，它通常是指，通过设计减少系统不能提供服务的时间。

[![在这里插入图片描述](E:\Development\Typora\images\20200318190542115.png)](https://img-blog.csdnimg.cn/20200318190542115.png)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200318190542115.png)



## Nginx的原理

[![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16629711716364.png)](https://img-blog.csdnimg.cn/20200318213130726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200318213130726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



[![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16629711716375.png)](https://img-blog.csdnimg.cn/20200318213002428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200318213002428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



> `一个master多个worker的好处`:
>
> - 可以使用nginx -s reload 热部署
>
>   在docker中使用docker exec -it<path_of_nginx> reload
>
> - 每个 woker 是独立的进程，如果有其中的一个 woker 出现问题，其他 woker 独立的，继续进行争抢，实现请求过程，不会造成服务中断
>
> ```
> 设置多少个worker合适
> ```
>
> ​    worker数和cpu的数量相等最合适(eg:8核就设置8个worker)，每个worker线程可以可
>
> `连接数worker_connection`:
>
> 1. 发送请求，占用了worker的几个连接数？
>
>    2个或者4个，请求占了两个(响应可能也占两个):client-worker-服务(比如tomcat)
>
> 2. nginx每个worker支持最大的连接数据1024，支持的并发数是多少?
>
>    eg:假设有4个worker 4*1024/2 或者 4*1024/4
>
> 普通的静态访问最大并发数是： worker_connections * worker_processes /2，
>
> 而如果是 HTTP 作 为反向代理来说，最大并发数量应该是 worker_connections * >worker_processes/4。

## 配置文件

包含三部分内容

1. 全局块：配置服务器整体运行的配置指令

   比如 worker_processes 1;工作进程数,越大,可以支持的并发数量就越多

2. events 块：影响 Nginx 服务器与用户的网络连接

   比如 worker_connections 1024; 支持的最大连接数为 1024

3. http 块

   还包含两部分：

   http 全局块

   server 块

```
user  nginx;
worker_processes  1;  #工作进程数，越大，可以支持的并发数量就越多

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;  #支持的最大连接数
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
root的处理结果是：root路径＋location路径
alias的处理结果是：使用alias路径替换location路径
```

**root实例**：

```
location ^~ /t/ {
     root /www/root/html/;
}
```

如果一个请求的URI是/t/a.html时，web服务器将会返回服务器上的/www/root/html/t/a.html的文件。

**alias实例**：

```
location ^~ /t/ {
 alias /www/root/html/new_t/;
}
```

如果一个请求的URI是/t/a.html时，web服务器将会返回服务器上的/www/root/html/new_t/a.html的文件。注意这里是new_t，因为alias会把location后面配置的路径丢弃掉，把当前匹配到的目录指向到指定的目录。

`使用alias时，目录名后面一定要加"/"`。

```
location /img/ {
    alias /var/www/image/;
}
location /img/ {
    root /var/www/image;
}
server {
    listen       80;
    server_name  39.97.180.158;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;


    location / {
        #root   html;
        proxy_pass http://39.97.180.158:8080;
        index  index.html index.htm index.jsp;
    }


    location ~ /test/b.html {
        proxy_pass http://39.97.180.158:8081;
    }

   #负载均衡
    location ~ /test01/a.html{
        proxy_pass http://myserver;
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
}
```

## 两台tomcat的反向代理

```
location / {
       #root   html;
       proxy_pass http://39.97.180.158:8080;
       index  index.html index.htm index.jsp;
   }


   location ~ /test/b.html {
       proxy_pass http://39.97.180.158:8081;
   }
```

## 实现负载均衡

**nginx 分配服务器策略**

> 第一种 轮询（默认）
>
> 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。
>
> 第二种 weight
>
> weight 代表权重默认为 1,权重越高被分配的客户端越多
>
> 第三种 ip_hash
>
> 每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器
>
> 第四种 least_conn（最少连接数）
>
> 在一些要求需要更长的时间才能完成的应用情况下， 最少连接可以更公平地控制应用程序实例的负载。使用最少连接负载均衡，nginx不会向负载繁忙的服务器上分发请求，而是将请求分发到负载低的服务器上。
>
> 第五种 fair（第三方）
>
> 按后端服务器的响应时间来分配请求，响应时间短的优先分配。
>
> 第六种 url_hash（第三方）
>
> 按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

```
第三方的负载均衡策略的实现需要安装第三方插件。
```

| 名称               | 模式            |
| ------------------ | --------------- |
| 轮询               | 默认方式        |
| weight             | 权重方式        |
| ip_hash            | 依据ip分配方式  |
| least_conn         | 最少连接方式    |
| fair（第三方）     | 响应时间方式    |
| url_hash（第三方） | 依据URL分配方式 |

nginx.conf(注意在http块中)

```
http {
......

#负载均衡
upstream myserver{
        #ip_hash;  #每个请求安访问的ip结果分配，这样每个访客固定访问这一个服务器，可以解决session的问题 
        #least_conn
        server 39.97.180.158:8080; #weight=1 ,设置权重
        server 39.97.180.158:8081;  #weight=1
        #fair 根据响应时间来分配
}

}
```

default.conf

```
server {
    listen       80;
    server_name  39.97.180.158;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;



    location / {
        #root   html;
        proxy_pass http://39.97.180.158:8080;
        index  index.html index.htm index.jsp;
    }


    location ~ /test/b.html {
        proxy_pass http://39.97.180.158:8081;
    }

   #负载均衡
    location ~ /test01/a.html{
        proxy_pass http://myserver;
    }
}
```

## 实现动静分离

> 两种方法:
>
> 一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；
>
> 另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。

[![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16629711716376.png)](https://img-blog.csdnimg.cn/20200318191014773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200318191014773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



```
#实现动静分离，此静态文件在nginx服务器中
location ~ /www/{
     root /data/;
     index index.html index.jsp index.htm;
}

#实现动静分离，此静态文件在nginx服务器中
location ~ /image/{
     alias /data/image/;
     autoindex on; #即打开了目录浏览功能
 }
 
```

[![在这里插入图片描述](E:\Development\Typora\images\20200318185415944.png)](https://img-blog.csdnimg.cn/20200318185415944.png)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200318185415944.png)


[![在这里插入图片描述](E:\Development\Typora\images\2020031818535043.png)在这里插入图片描述](https://img-blog.csdnimg.cn/2020031818535043.png)



## 高可用

[![在这里插入图片描述](E:\Development\Typora\images\20200318190542115.png)](https://img-blog.csdnimg.cn/20200318190542115.png)

[在这里插入图片描述](https://img-blog.csdnimg.cn/20200318190542115.png)



需求:

（1）需要两台 nginx 服务器

（2）需要 keepalived

```
yum install -y keepalived
```

(3) 修改配置文件 /etc/keepalived/keepalived.conf

```
       主机和备机的配置文件在于：state 和 priority 两个地方，修改下这里就行，主机初始状态为 MASTER，优先级为100 备机初始状态为BACKUP，优先级为80

! Configuration File for keepalived

global_defs { # 全局配置
   notification_email {
     123@qq.com # 接受告警的邮箱
   }
   notification_email_from sample@163.com # 指定发件人
   smtp_server 192.168.200.1 # 指定的 smtp服务器地址
   smtp_connect_timeout 30   # 指定 smtp 连接超时时间
   router_id LVS_DEVEL       # 负载均衡标识，在局域网应该是唯一的
  
}

vrrp_script chk_nginx { 
    script "/etc/keepalived/nginx_check.sh" #指定要执行的脚本的路径 
    interval 5 # 指定脚本执行的间隔。单位是秒。默认为1s。
    weight -20 #权重，修改挂掉的服务器的权重
}

vrrp_instance VI_1 {
    state MASTER    # 指定该keepalived节点的初始状态,备份服务器写BACKUP 这里这个并不意味着他就是主机！！是根据下面比较 priority 值来判断主机的,
    interface ens33 # vrrp实例绑定的接口，用于发送VRRP包 这里需要写成 ifconfig 显示的网卡名
    virtual_router_id 51 # 指定VRRP实例ID，范围是0-255
    priority 100 # 指定优先级，优先级高的将成为MASTER
    advert_int 1 # 指定发送VRRP通告的间隔。单位是秒
    authentication {
        auth_type PASS # 指定认证方式。PASS简单密码认证(推荐),AH:IPSEC认证(不推荐)
        auth_pass 1111 # 指定认证所使用的密码。最多8位
    }
    virtual_ipaddress {
        192.168.65.135 # 指定VIP地址
    }
    track_script {
        chk_nginx
    }
}
```

(4) 创建定时检测 nginx 是否正常运行的脚本 (执行 chmod +x nginx_check.sh 使得脚本可以执行)

```
#!/bin/bash
A=`ps -C nginx --no-header |wc -l`
 if [ $A -eq 0 ];then
      # /usr/bin/docker rstart docker-nginx  >> ./tmp/echo.log 2>> ./tmp/echo2.log  #正常情况下这里需要重启 nginx，这里已疯,环境变量导致着无法识别命名
      sleep 3
            if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
                        ps -ef|grep keepalived|grep -v grep|awk '{print $2}'|xargs kill -9
            fi
fi

```

