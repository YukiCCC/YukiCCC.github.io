---
title: Nginx
date: 2023-10-16 09:22:27
tags:
---

## 1. 域名解析



一个域名一定会被解析为一个或多个ip，一般包含两步：

- 本地域名解析

  浏览器会首先在本机的hosts文件中查找域名映射的IP地址，如果找到就返回IP，没找到则进行域名服务器解析，一般本地解析都会失败，因为默认这个文件是空的。

  Windows下的hosts文件地址：C:/windows/system32/drivers/etc/hosts

  Linux下的hosts文件所在路径： /etc/hosts

- 域名服务器解析

  本地解析失败，才会进行域名服务器解析，域名服务器就是网络中的一台计算机，里面记录了所有注册备案的域名和IP映射关系，一般只要域名是正确的，并且备案通过，一定能找到。

## 2. Nginx简介

是一个WEB服务器，是一个高性能的反向代理的服务器、还是一个负载均衡服务器，同时还可以实现动静分离。

特点：在高并发和处理静态资源上相对于tomcat有更多优势.

+ 什么是反向代理

  + 正向代理：客户端先进行代理服务器的设置，客户端发送请求到代理服务器，代理服务器将请求转发至原始服务器。代理服务器所代理的对象是很多个客户端。

  + 反向代理：对于客户而言反向代理就像原始服务器,客户不需要作任何设置，客户端发送请求，直接发送到代理服务器，代理服务器判断向何处转发请求,并将获得的内容返回给客户端。反向代理是对多个服务器进行代理

+ 什么是负载均衡

  可以按照调度规则实现动态、静态页面分离，可以按照轮询、ip哈希、权重等多种方式实现将请求平均分配到后端服务器上

  

## 3. docker安装



### 3.1 拉取镜像

docker pull nginx

### 3.2 启动容器

+ docker run -d --name mynginx1 -p 80:80 nginx bash

+ 配置文件在/etc/nginx/nginx.conf,默认没有vi命令

+ 拷贝nginx目录到宿主机：

  + docker cp mynginx1:/etc/nginx   /opt/nginx

  + docker cp mynginx1:/usr/share/nginx/html  /usr/share/nginx/html

    

+ 删除容器：docker stop mynginx1--》docker rm mynginx1

+ 启动容器并添加数据卷：

+ docker run  -v  /opt/data/nginx:/etc/nginx   -di --name mynginx  -p 80:80 nginx

  ​		  

  

## yum安装

1. 安装yum仓库 yum-utils

   ```shell
   #可以不执行，当出现版本问题时，再进行更新
   sudo yum install yum-utils -y
   ```

2. 创建/etc/yum.repos.d/nginx.repo文件，

   ```
   vi /etc/yum.repos.d/nginx.repo
   ```

   内容如下

   ```shell
   [nginx-stable]
   name=nginx stable repo
   baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
   gpgcheck=1
   enabled=1
   gpgkey=https://nginx.org/keys/nginx_signing.key
   module_hotfixes=true
   
   #[nginx-mainline]
   #name=nginx mainline repo
   #baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
   #gpgcheck=1
   #enabled=0
   #gpgkey=https://nginx.org/keys/nginx_signing.key
   #module_hotfixes=true
   ```

   nginx-stable:稳定版的信息。一般情况下使用此版本

   nginx-mainline：主力版本（或开发版本），如果想要安装此版本，可以先使用如下命令，再安装

   使用stable版本，下面这句话不要执行

   ```shell
   #使用stable版本，下面这句话不要执行
   sudo yum-config-manager --enable nginx-mainline
   ```

3. 安装

   ```shell
   sudo yum install nginx -y
   ```

4. 启动测试

   ```shell
   nginx
   ```

5. 在windows主机上输入 http://192.168.33.10

6. 其它命令

   ```shell
   #停止
   nginx  -s  stop
   #重启
   nginx -s reopen
   #检查是否有语法错误
   nginx -t
   #重新加载
   nginx  -s  reload
   #查看版本
   nginx  -V  或nginx -v
   #查看80端口的程序
   netstat –ano | grep 80
   #卸载Nginx
   yum remove nginx
   ```

关闭防火墙 (需要重启)：chkconfig  iptables off

卸载步骤：

```
卸载时
1：先停止nginx服务 nginx -s stop
2：查找nginx目录文件：find / -name nginx
3: 依次删除找到的目录：rm -rf /usr/sbin/nginx
4：使用yum清除 yum remove nginx
```

其它Linux版本的安装请参考以下文档：

http://nginx.org/en/linux_packages.html#RHEL-CentOS

### 配置文件位置

/etc/nginx  

主配置文件 ngixn.conf

每个服务器配置的conf.d目录下

主启动页：/usr/share/nginx/html

# 四、nginx信号量

## 1. 工作模式

​		nginx是一个多进程/多线程高性能web服务器，在linux系统中，nginx启动后会以后台守护进程（daemon）的方式去运行，后台进程包含一个master进程和多个worker进程（可以通过在nginx.conf配置文件中配置worker_processes参数设置），可以充分利用多核架构。

 		nginx默认的工作模式是以多进程的方式来工作的，nginx也是支持多线程的方式的，只是我们主流的方式还是多进程的方式。nginx在启动之后会有一个master进程和多个worker进程（默认是一个），多个worker子进程将监听同一个端口，并行处理请求。worker 进程数应该设置为等于 CPU 的核数，高流量并发场合也可以考虑将进程数提高至 CPU 核数 * 2。

​		master主进程主要用来管理worker进程，主要作用是：读取并验正配置信息，管理真正提供服务的worker进程，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程。master进程不会对用户请求提供服务，而用户的请求则是worker进程来响应的。

​		nginx是通过信号来控制，比如关闭，重启等去控制nginx进程。nginx信号是属于nginx进程间的通信的一种机制，比如master主进程控制多个worker子进程，也是通过信号控制的，如下图。

![工作模式](/工作模式.png)

## 2. 信号量

信号控制语法

​	***kill -信号选项 nginx的主进程号***

- TERM，INT：快速关闭
- QUIT ：从容关闭（优雅的关闭进程,即等请求结束后再关闭）
- HUP ：平滑重启，重新加载配置文件 （平滑重启，修改配置文件之后不用重启服务器。直接kill -PUT 进程号即可）
- USR1 ：重新读取日志文件，在切割日志时用途较大（停止写入老日志文件，打开新日志文件，之所以这样是因为老日志文件就算修改的文件名，由于inode的原因，nginx还会一直往老的日志文件写入数据）
- USR2 ：平滑升级可执行程序  ，nginx升级时候用
- WINCH ：优雅的关闭旧的进程(配合上USR2来进行升级)

```shell
#查找nginx.pid
find / -name nginx.pid
#查看nginx.pid的进程id ==》结果为4851
cat /var/run/nginx.pid
#关闭nginx进程 （id号为4851，每次都不一样，会变化，需要查找）
#关闭后无法访问，需要重新启动
kill -QUIT 4851
```



# 五、nginx的配置



![配置文件结构](/配置文件结构.png)

## 1. main 全局配置

nginx在运行时与具体业务功能（比如http服务或者email服务代理）无关的一些参数，比如工作进程数，运行的身份等。

```shell
## 指定nginx进程使用什么用户启动，默认是nobody
user       www www;  
#指定启动多少进程来处理请求，一般情况下设置成CPU的核数。默认为1
worker_processes  4;  
#在高并发情况下，通过设置将CPU和具体的进程绑定来降低由于多核CPU切换造成的寄存器等现场重建带来的性能损耗。
#位数和进程数相关。
#两个cpu内核开启两个进程
#worker_processes  2;  
#worker_processes  01 10；分别对应第一个CPU内核，第二个CPU内核  
worker_cpu_affinity 0001 0010 0100 1000;  #分别对应第一个CPU内核……第四个CPU内核 
#error_log是个主模块指令，用来定义全局错误日志文件。
#日志输出级别有debug、info、notice、warn、error、crit可供选择，其中，debug输出日志最为最详细，而crit输出日志最少。
error_log  logs/error.log crit;
#指定进程pid文件的位置。
pid        logs/nginx.pid;
#用于指定一个nginx进程可以打开的最多文件描述符数目，需要使用命令“ulimit -n 8192”来设置。
worker_rlimit_nofile 8192;
```

## 2. events模块

```shell
events {
  #每一个worker进程能并发处理（发起）的最大连接数（包含与客户端或后端被代理服务器间等所有连接数）。
  #默认是1024
  #进程的最大连接数受Linux系统进程的最大打开文件数限制，最大不能超过worker_rlimit_nofile的值
  worker_connections  4096;  
  #use是个事件模块指令，用来指定Nginx的工作模式。
  #Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll。
  #其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，
  #不同的是epoll用在Linux平台上，而kqueue用在BSD系统中。对于Linux系统，epoll工作模式是首选。
  #use [ kqueue | rtsig | epoll | /dev/poll | select | poll ] ;
  use epoll;
}
```

## 3. http服务器

```shell
#include是个主模块指令，实现对配置文件所包含的文件的设定，可以减少主配置文件的复杂度。
#类似于Apache中的include方法。
include    conf/mime.types;
include    /etc/nginx/proxy.conf;
include    /etc/nginx/fastcgi.conf;
#定义路径下默认访问的文件名
index    index.html index.htm index.php;
#default_type属于HTTP核心模块指令，这里设定默认类型为二进制流，也就是当文件类型未定义时使用这种方式。
#例如在没有配置PHP环境时，Nginx是不予解析的，此时，用浏览器访问PHP文件就会出现下载窗口。
default_type application/octet-stream;
```

### 3.1 客户端head缓存

```shell
#服务器名字的hash表大小
server_names_hash_bucket_size 128;
#用来指定来自客户端请求头的header buffer 大小。
client_header_buffer_size 32k; 
#用来指定客户端请求中较大的消息头的缓存最大数量和大小，4为个数，128k为大小，最大缓存为4个128KB。
large_client_header_buffers 4 128k; 
#允许户端请求的最大单个文件字节数。如果有上传较大文件，请设置它的限制值。
client_max_body_size 10m; 
#缓冲区代理缓冲用户端请求的最大字节数。
client_body_buffer_size 128k; 
#高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，
#减少用户空间到内核空间的上下文切换。对于普通应用设为 on，
# 如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。
# 开启 tcp_nopush on; 和tcp_nodelay on; 防止网络阻塞。
sendfile on ; 
tcp_nopush on;
tcp_nodelay on;
#长连接超时时间，单位是秒，65s内没上传完成会导致失败。
keepalive_timeout 65 : 
#用于设置客户端请求主体读取超时时间，默认是60s。
client_body_timeout 60s;
#用于指定响应客户端的超时时间。
send_timeout 60s;
```



### 3.2 FastCGI参数

FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。

```shell
#指定连接到后端FastCGI的超时时间。
fastcgi_connect_timeout 300;  
#指定向FastCGI传送请求的超时时间，此值是已经完成两次握手后向FastCGI传送请求的超时时间。
fastcgi_send_timeout 300;  
#指定接收FastCGI应答的超时时间，此值是已经完成两次握手后接收FastCGI应答的超时时间。
fastcgi_read_timeout 300; 
#用于指定读取FastCGI应答第一部分需要多大的缓冲区。
# 此值表示将使用1个64KB的缓冲区读取应答的第一部分（应答头）
fastcgi_buffer_size 64k; 
# 指定本地需要用多少和多大的缓冲区来缓冲FastCGI的应答请求。
fastcgi_buffers 4 64k;  
#默认值是fastcgi_buffers的两倍。
fastcgi_busy_buffers_size 128k;  
#表示在写入缓存文件时使用多大的数据块，默认值是fastcgi_buffers的两倍。
fastcgi_temp_file_write_size 128k;
#表示开启FastCGI缓存并为其指定一个名称。开启缓存非常有用，可以有效降低CPU的负载，并且防止502错误的发生。
fastcgi_cache TEST;  
#FastCGI缓存指定一个文件路径、目录结构等级、关键字区域存储时间和非活动删除时间。
fastcgi_cache_path /usr/local/nginx/fastcgi_cache levels=1:2 keys_zone=TEST:10m inactive=5m;  
#用来指定应答代码的缓存时间。
#实例中的值表示将200和302应答缓存一个小时。
fastcgi_cache_valid 200 302 1h;  
#将301应答缓存1天，其他应答均缓存1分钟。
fastcgi_cache_valid 301 1d;
#
fastcgi_cache_valid any 1m; 
```



### 3.3 gzip模块设置

```shell
#开启gzip压缩输出
gzip on;
#最小压缩文件大小，页面字节数从header头的Content-Length中获取。默认值为0，不管多大页面都压缩，建议设置成大于1K的字节数，小于1K可能会越压越大。
gzip_min_length 1k;
#压缩缓冲区，表示申请四个8K的内存作为压缩结果流缓存
#默认是申请与原始数据大小相同的内存空间来存储gzip压缩结果。
gzip_buffers    4 8k;
#用于设置识别HTTP协议版本，默认是1.1
#（默认1.1，前端如果是squid2.5请使用1.0）
gzip_http_version 1.1;
#压缩等级，1压缩比最小，处理速度最快，9压缩比最大，传输速度快，但是消耗CPU资源。
#范围1~9
gzip_comp_level 5;
#压缩类型，默认已包含text/html。
gzip_types text/html text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
#和http头有关系，会在响应头加个 Vary: Accept-Encoding ，
# 可以让前端的缓存服务器缓存经过gzip压缩的页面，例如，用Squid缓存经过Nginx压缩的数据。
gzip_vary on;
#Nginx作为反向代理的时候启用，决定开启或者关闭后端服务器返回的结果是否压缩，
#  匹配的前提是后端服务器必须要返回包含”Via”的 header头。
#gzip_proxied any;
```



## 4. nginx 配置虚拟主机

### 4.1 配置虚拟主机流程

1：拷贝一段完整的server段，并放入http标签内

2：更改server_name以及对应网页的root根目录

3：创建server_name 对应网页的根目录，如果没有index首页会出现404错误。

4：对客户端server_name 的主机做host 解析或DNS配置。

5：浏览器访问，或者在Linux客户端做host解析，用wget或curl 访问。

```nginx
server {
         listen 80;
         server_name localhost;
         index index.html index.htm;

         location /emp {
         			#在/opt/project目录下存放各种html文件
                    root /opt/project;
                    #默认的首页
                    index index.html index.htm;
         }
   }

#浏览器中使用localhost/emp来访问对应的资源
```

如果出现502,503错误，尝试临时关闭SELinux后再测试。

```shell
#临时关闭SELinux
setenforce 0
永久关闭：
#输入命令（需要重启服务器）
vim /etc/selinux/config
#设置config文件中的SELINUX=enforcing改为SELINUX=disabled，然后退出保存。
```

如果出现403错误，用以下方式解决

- 看log，查看路径是否正确
- 如果路径正确，则确认配置文件中用户是什么，修改和当前用户匹配（如果当前用户为root，请也将用户改成root）。

### 4.2 location模块语法规则

location的语法规则有两种：前缀字符串（路径名）和正则表达式

- 前缀字符串

```shell
#匹配以/some/path/开头的路径，如：/some/path/domt.html
location /some/path/ { }
```

- 正则表达式

  正则表达式前加上“~”表示区分大小写，加上"~*"表示不区分大小写

  ※当使用插入符时“^~"则表示只匹配前缀字符串，不匹配正则表达式



location [=|~|~*|^~] /uri/ { … }**  

- =     精确匹配，如果找到匹配=号的内容，立即停止搜索，并立即处理请求(优先级最高)
- ~     区分大小写
- ~*  不区分大小写
- ^~  只匹配字符串，不匹配正则表达式
- /    通用匹配，任何请求都会匹配到

```nginx
#匹配跟目录
location =/ {
	
}
#各种图片格式结尾的（正则匹配）
# ~ 区分大小写
# . 匹配除换行符之外的任何字符
# * 匹配0或多次
#\ 转义字符 \. 匹配点好（.)
location ~ .*\.(gif|jpg|jpeg|png|bmp|icon)$ {
}
#将所有请求都交给
location / {

}
```



# 六、nginx动静分离

## 1. 什么是动静分离

在Web开发中，通常来说，动态资源其实就是指那些后台资源，而静态资源就是指HTML、JavaScript、CSS、img等文件。

动静分离就是将动态资源和静态资源分开，将静态资源部署在Nginx上。当一个请求来的时候，如果是静态资源的请求，就直接到nginx配置的静态资源目录下面获取资源，如果是动态资源的请求，nginx利用反向代理的原理，把请求转发给后台应用去处理，从而实现动静分离。

优点：

- 可以很大程度的提升静态资源的访问速度。
- 前后端可以并行开发、有效地提高开发效率。

## 2. nginx结合tomcat实现动静分离

### 项目准备

编写一个带图片的index.html,放置在/opt/project/emp目录中。

图片fruit01.jpg放入/opt/project/static/img目录中

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<form action="">
		用户：<input name="user"><br>
		密码：<input name="password"><br>
		<input type="submit" value="登录">
		<input type="reset" value="取消"> 
	</form> 
	<img src="/static/img/fruit01.jpg" style="width:200px;height:200px"/>
</body>
</html>
```

### nginx配置

思路：动、静态的文件，请求时匹配不同的目录

当访问gif,jpeg时 直接访问/opt/project/static/目录下内容,正则自行配置

```nginx
user  root;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
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

   # include /etc/nginx/conf.d/*.conf;
   server {
        listen 8100;
        server_name localhost;
	    location / {
		    root /opt/project/emp;
		     index index.html index.htm;
	     }
	 	
  	     # 所有静态图片请求都放到static目录下
	     location ~ .*\.(jpg|jpeg|png)$ { 
      		alias   /opt/project/static/;  
  	      }  
  	     error_page   500 502 503 504  /50x.html;  
  		    location = /50x.html {  
     		    root   e:wwwroot;  
  	     }
    }
}
```



### 测试

图片能够正常加载

通过以下语句测试图片的正常加载 

```
curl -I http://localhost/static/img/fruit01.jpg
```

![动静分离图片验证](D:\资料\我的笔记\02.Nginx\图例\动静分离图片验证.png)

# 七、nginx反向代理、负载均衡

## 1. 什么是反向代理

- 代理：通过客户机的配置，实现让一台服务器代理客户机，客户的所有请求都交给代理服务器处理
- 反向代理：用一台服务器，代理真实服务器，用户访问时，不是访问真实的服务器，而是访问代理服务器。

Nginx可以当作反向代理服务来使用时，我们需要提前在Nginx中配置好反向代理的规则，不同的请求，交给不同的真实服务器处理。当请求到达nginx，nginx会根据已经定义的规则进行请求的转发，从而实现路由功能。

**安装在主机上**

![反向代理1](/反向代理1.jpg)

**安装在虚拟机上**

![反向代理2](/反向代理2.jpg)

## 2. Nginx反向代理Tocat 

### 拷贝配置文件

将/etc/nginx/conf.d目录下的default.conf拷贝一份，并命名成my.conf

```shell
#进入配置server的目录
cd /etc/nginx/conf.d
#拷贝文件（保留模板文件，防止被破坏）
cp default.conf  my.conf
#打开my.conf进行编辑
vi  my.conf
```



### my.conf的修改

使用proxy_pass来设置反向代理的服务器

```shell

      1 server {
      2     listen       80;
      3     server_name  localhost;
      4
      5     #access_log  /var/log/nginx/host.access.log  main;
      6
      7     location / {
                #修改内容，注释掉8，9行，增加第10行数据
      8         #root   /usr/share/nginx/html;
      9         #index  index.html index.htm;
     10         proxy_pass  http://192.168.40.251:8080;
     11     }
     12  ....



```



### 修改nginx.conf

/etc/nginx/nginx.conf中的http节点中修改以下内容

```shell
	#修改内容
    #include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/conf.d/my.conf;

```



## 3. 负载均衡  

### upstream模块

nginx 的负载均衡功能依赖于 ngx_http_upstream_module模块，所支持的代理方式有 proxy_pass(一般用于反向代理).fastcgi_pass(一般用于和动态程序交互），memcached_pass,proxy_next_upstream,fastcig_next_pass 以及memcached_next_pass 。

upstream 模块应该放于http{}标签内。

```shell
upstream dynamic {
    zone upstream_dynamic 64k;


    server backend1.example.com      weight=5;
    server backend2.example.com:8080 fail_timeout=5s slow_start=30s;
    server 192.0.2.1                 max_fails=3;
    server backend3.example.com      resolve;

    server backup1.example.com:8080  backup;
    server backup2.example.com:8080  backup;
    #通过该指令配置了每个worker进程与上游服务器可缓存的空闲连接的最大数量。
    #当超出这个数量时，最近最少使用的连接将被关闭。keepalive指令不限制worker进程与上游服务器的总连接。
    keepalive 100;
}

//案例
upstream tc{
		#ip_hash;
		server 192.168.4.91:80 weight=1 backup;
		server 192.168.4.91:8080 weight=3;
	}	
```



### my.conf

```
proxy_pass http://tc;
```



### 负载均衡策略

轮询(rr)：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器故障，故障系统自动清除，使用户访问不受影响。适用于多个服务器的性能相当的情况下，适用此策略。

轮询权值(weight),weight值越大，分配到的访问几率越高，主要用于后端每个服务器性能不均的情况。

ip_hash，每个请求按访问IP的hash结果分配(**IP地址的前三段**)，这样来自同一个IP的固定访问一个后端服务器，主要解决动态网站session共享的问题。 192.168.40.xxx

url_hash，按照访问的URL的hash结果来分配请求，是每个URL定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率，nginx本身不支持，如果想使用需要安装nginx的hash软件包。

### server模块的写法 

server IP 调度状态

server指令指定后端服务器IP地址和端口，同时还可以设定每个后端服务器在负载均衡调度中的状态。

- down  表示当前的server暂时不参与负载均衡。
- backup 预留的备份服务器，当其他所有的非backup服务器出现故障或者忙的时候，才会请求backup机器，因为这台集群的压力最小。
- max_fails 允许请求失败的次数，默认是1，当超过最大次数时，返回proxy_next_upstream模块定义的错误。0表示禁止失败尝试，企业场景：2-3.京东1次，蓝汛10次，根据业务需求去配置。
- fail_timeout，在经历了max_fails次失败后，暂停服务的时间。京东是3s，蓝汛是3s，根据业务需求配置。常规业务2-3秒合理。



## vue项目的发布

### 开发或者测试环境的确认

### 执行打包命令    npm run  build:prod

### 把dist文件夹放入centos 的 /opt/dist目录  

​      放入vagrant共享目录， vagrant reload读取文件，通过cp命令copy到opt目录下

### 修改nginx的配置（server段）

```
#nginx.conf
http {
   .....
   upstream tc {
      server  192.168.66.182:8080;
   }
   
   include /etc/nginx/conf.d/my.conf
}

#my.conf
server {
    ....
    location /{
       root  /opt/dist;
       index index.html index.htm;
    }
    location /core {
       proxy_pass   http://tc;
    }
}
```



# 八、session共享

## 1.ip_hash

缺点：

+ 分配不均匀。
+ 如果后端还作了其他负载均衡，就不能共享session

## 2.tomcat会话复制

会话数据会存储在每个服务器上的堆内存中

### 2.1 实现步骤

+ 在每一个tomcat中添加集群缓存配置

  在tomcat的conf中找到server.xml，在<Engine name="Catalina" defaultHost="localhost">这一行下面添加下面内容：

  ```
  <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster" channelSendOptions="8">
    <Manager className="org.apache.catalina.ha.session.DeltaManager"
               expireSessionsOnShutdown="false"
               notifyListenersOnReplication="true"/>
    <Channel className="org.apache.catalina.tribes.group.GroupChannel">
        <Membership className="org.apache.catalina.tribes.membership.McastService"
              address="228.0.0.4" port="45564" frequency="500" dropTime="3000"/>
        <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
    address="auto" port="4000" autoBind="100" selectorTimeout="5000" maxThreads="6"/>
      <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">       <Transport       className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
      </Sender>
       <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
       <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>
      </Channel>
      <Valve className="org.apache.catalina.ha.tcp.ReplicationValve" filter=""/>
      <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>
      <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
          tempDir="/tmp/war-temp/" deployDir="/tmp/war-deploy/"
          watchDir="/tmp/war-listen/" watchEnabled="false"/>
      
      <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
  </Cluster>
  ```

+ 在每个项目的web.xml中添加下面标签

  ```
  <distributable/>
  ```

### 2.2 缺点

+ 因为每个服务器都存储一份session，所以数据冗余
+ 如果某个服务器内存很小，可能无法存储。

## 3.会话共享

推荐使用Spring Session方案，主流, 会话存储在远程的redis缓存中.

### 3.1 实现原理

+ 客户端第一次发请求时，没有携带sessionID，nginx将请求分发给服务器1 ，然后服务器1 产生session0，spring 对sesion0封装成sesion1，并根据session0计算并更新session1的id,然后 放入redis中，并把session0的原始ID回写到浏览器，这样服务器1 和redis中都会有一个相同的session1


+ 当客户端发送第二次请求的时候，nginx将请求分发给服务器2 （无session），因为请求中携带了一个sessionID，那么服务器2 就根据sessionID得出session1的id，用这个id去redis中获取session。



### 3.2 实现步骤

建两个springboot项目，内容如下，除了端口号不同

+ 两个项目的pom依赖

  ```
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.session</groupId>
      <artifactId>spring-session-data-redis</artifactId>
      <version>2.0.4.RELEASE</version>
    </dependency>
  </dependencies>
  ```

+ templates目录添加login.html

  ```
      <form action="/login" method="post">
          <input name="username"><br>
          <input type="submit">
      </form>
  ```

+ templates目录添加success.html

  ```
      <html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
      <h1>this is session1</h1>
      <span th:text="${session.username}"></span>
  ```

+ yml内容如下,注意：另一项目端口为8080

  ```
  server:
    port: 80
  spring:
    redis:
      host: 127.0.0.1
      port: 6379
      jedis:
        pool:
          max-idle: 8
  ```


+ config包下添加类

  ```
  @Configuration
  public class MyConfig implements WebMvcConfigurer {
      public void addViewControllers(ViewControllerRegistry registry) {
          registry.addViewController("/showSuc").setViewName("success");
          registry.addViewController("/showLogin").setViewName("login");
      }
  }
  ```


+ controller包中添加类

      @Controller
      public class UserCtl {
      @RequestMapping("/login")
      public String hello(HttpSession session, String username){
          session.setAttribute("username",username);
          //注意：这里的session已不是传统session,被重构成新的session，存储在redis中
          System.out.println("name:"+username+",sessionID："+session.getId());
          return "success";
      }
      }

+ **主类添加注解**

      //使用该注解，会重构session,参数为session存活时间
      @EnableRedisHttpSession(maxInactiveIntervalInSeconds = 10000*30)
      @SpringBootApplication
      public class SessionApp {
      public static void main(String[] args) {
          SpringApplication.run(SessionApp.class,args);
      }
      }

  

+ nginx中

  + 修改配置

    ```
    location / {
    		root   /opt/project/emp;
    		index  index.html index.htm;
    		proxy_pass http://tc;
    }
   upstream tc{
        server 192.168.1.144:80;
      server 192.168.1.144:8080;
      }
    ```
  
  + 重启nginx
  
   /usr/local/nginx/sbin/nginx -s reload


+ 启动redis: redis-server redis.conf

+ 启动80和8080两个端口的项目

+ 测试

  + 浏览器输入http://192.168.184.100/login.html,输入用户名

  + http://192.168.184.100/showSuc，反复刷新观看

    