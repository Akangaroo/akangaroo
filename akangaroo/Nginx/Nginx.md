## Nginx简介

Nginx ("engine x") 是一个高性能的 HTTP 和反向代理服务器,特点是占有内存少，并发能力强。

Nginx 可以作为静态页面的 web 服务器，同时还支持 CGI 协议的动态语言，比如 perl、php等。但是不支持 java。java 程序只能通过与 tomcat 配合完成。



### 正向代理概念

一般来说，访问google等外网的时候是不能直接连接的，此时需要在**客户端配置代理服务器**，通过代理服务器来访问google。简单来说就是搭建梯子。

**正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端**

![image-20210127191919819](http://img.akangaroo.cn/nginx/nginx01.png)

### 反向代理概念

一般来说，tomcat是8080端口，而我们http协议的端口为80端口。如何通过80端口访问8080和8081端口？此时需要是在**服务端配置代理服务器**，该服务器监听80端口，然后将请求转发到tomcat，即**代理将来自外网客户端的请求转发到内网服务器**。

**反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端**

![image-20210127192425454](http://img.akangaroo.cn/nginx/nginx02.png)

## 安装和配置文件

### 安装

安装采用的是docker安装

```shell
# 拉取nginx的镜像
docker pull nginx

# 运行镜像
docker run --name nginx-test -p 80:80 -d nginx

# 在宿主机新建目录
mkdir -p /opt/nginx/html /opt/nginx/logs /opt/nginx/conf

# 将下面两个复制到/opt/nginx/conf文件夹下
cp 容器ID:/etc/nginx/nginx.conf /opt/nginx/conf
cp 容器ID:/etc/nginx/conf.d /opt/nginx/conf

# 将更改运行的nginx-test停止
docker stop nginx-test

# 运行nginx
docker run -d -p 80:80 --name nginx -v /opt/nginx/conf/conf.d/:/etc/nginx/conf.d -v /opt/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /opt/nginx/logs/:/var/log/nginx -v /opt/nginx/html:/usr/share/nginx/html nginx

-d 后台运行
-p 指定端口
--name 指定名称
-v 挂载目录，可以看配置中的介绍
```

在`/opt/nginx/html`目录下新建`index.html`文件

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Docker搭建Nginx</title>
</head>
<body>
    <h1>人生得意须尽欢</h1>
</body>
</html>
```

将服务器的防火墙关闭或者开启80端口，输入`ip地址`即可成功访问。



### 配置文件

之前安装的时候，我们挂载了几个目录。目录挂载的意思就是，将docker中Nginx的目录与宿主机的目录对应，修改宿主机文件的同时也会修改docker中Nginx文件的信息。

```shell
# 该目录与Nginx的配置有关
-v /opt/nginx/conf/conf.d/:/etc/nginx/conf.d
# 该文件与Nginx的配置有关
-v /opt/nginx/conf/nginx.conf:/etc/nginx/nginx.conf 
# 该文件与Nginx的日志有关
-v /opt/nginx/logs/:/var/log/nginx 
```

先看`nginx.conf`文件

![image-20210127194901055](http://img.akangaroo.cn/nginx/nginx03.png)

主要包括三块的内容：

**全局块**：从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

```
worker_processes：处理并发数
```

**events**：影响 Nginx 服务器与用户的网络连接

```
worker_connections  1024：最大连接数
```

**http**：这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。

```
include /etc/nginx/conf.d/*.conf：Nginx的主要配置在`conf.d`目录下。在`*.conf`中存在server块，server块中存在着我们需要配置的信息
```



通过`cd conf.d`进入conf.d目录，发现有一个default.conf文件，进入后，发现server块。

server块中可以配置多个location块。

红色箭头是默认存在的，绿色箭头是我自己的配置。

![image-20210127200421044](http://img.akangaroo.cn/nginx/nginx04.png)



## 反向代理配置

上述介绍了Nginx中的配置文件，如果要进行反向代理的配置，在`conf.d`目录下创建以`.conf`结尾的文件即可。



**server_name**配置域名或者ip地址，因为我是有域名的，所以直接配置的域名。

**location /** 表示处理所有请求

**proxy_pass **表示把请求都交给`http://118.178.141.179:8080`来处理

> 每行结尾加;
>
> proxy_pass中要配置http://

![image-20210127201035528](http://img.akangaroo.cn/nginx/nginx05.png)

配置成功后，退出，`docker restart nginx`重启Nginx使配置生效

之后输入域名或者ip即可成功访问，因为监听的是80端口，在浏览器中不需要输入端口。



location指定说明：

- 该指令用于匹配 URL，语法格式如下

  ```
  location [= | ~ | ~* | ^~] uri {
  }
  ```

- =：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。

- ~：用于表示 uri 包含正则表达式，并且区分大小写。

- ~*：用于表示 uri 包含正则表达式，并且不区分大小写。

- ^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location块中的正则 uri 和请求字符串做匹配。

> 注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。



## 负载均衡配置

负载均衡：通过Nginx将负载发送到不同的服务器。



在server块外面配置upstream

```
 upstream server_pool{
     server	118.178.141.179:8080 ;
     server	118.178.141.179:8081 ;
 }
```

接着在location块中配置如下即可

```
location / {
      proxy_pass http://server_pool;
}
```



nginx分类服务器的策略：

1. 轮询（默认）

   每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器挂掉则自动剔除 。

2. 权重（weight）

   weight 代表权重默认为 1, 权重越高被分配的客户端越多

   ```
    upstream server_pool{
        server	118.178.141.179:8080 weight=1;
        server	118.178.141.179:8081 weight=2;
    }
   ```

3. ip_hash

   每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。

   ```
    upstream server_pool{
    	 ip_hash;
        server	118.178.141.179:8080;
        server	118.178.141.179:8081;
    }
   ```

4. fair（第三方）

   按后端服务器的响应时间来分配请求，响应时间短的优先分配。

   ```
    upstream server_pool{
    	 fair;
        server	118.178.141.179:8080;
        server	118.178.141.179:8081;
    }
   ```

   



## 动静分离介绍

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

例如由Nginx来解析静态资源，tomcat解析动态资源。

![image-20210127211440690](http://img.akangaroo.cn/nginx/nginx06.png)

