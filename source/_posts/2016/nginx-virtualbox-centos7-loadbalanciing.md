---
title: 虚拟机下CentOS中Nginx的反向代理和负载均衡初探
permalink: nginx-virtualbox-centos7-loadbalanciing
category: Linux
tags:
  - Linux
  - CentOS
  - Nginx
date: 2016-05-28 16:16:56
---
### 目的:
> 属性nginx的反向代理和负载均衡

### 要求:
> 使用nginx配置三个站点
虚拟机的主机能通过负载均衡访问这三个站点

### 环境:
> VirtualBox: 5.0.20
CentOS: CentOS-7-x86_64-Minimal-1511
Nginx: 1.10.0
主机: win7

### 具体设置
#### 三个站点分别为:
##### 站点1:
> 路径: /home/www/blog_1
端口: 801
根目录文件: index.html
根目录文件内容: blog_1

##### 站点2:
> 路径: /home/www/blog_2
端口: 802
根目录文件: index.html
根目录文件内容: blog_2

##### 站点3:
> 路径: /home/www/blog_3
端口: 803
根目录文件: index.html
根目录文件内容: blog_3

##### VirtualBox的网络配置:
> NAT模式下配置虚拟机和主机的端口映射

|主机IP|主机端口|虚拟机IP|虚拟机端口|
|-----|-----|-----|-----|
|127.0.0.1|12080|10.0.2.15|80|
|127.0.0.1|12801|10.0.2.15|801|
|127.0.0.1|12802|10.0.2.15|802|
|127.0.0.1|12803|10.0.2.15|803|

>这样配置的目的是
通过访问 127.0.0.1:12080 访问负载均衡站点
通过 127.0.0.1:12801 ..12802 ..12803 访问各个站点

##### 简单配置后的nginx.conf如下(所有原来注释的内容都被我删掉了):
```bash
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    upstream blog_server {
      server 127.0.0.1:801 weight=1;
      server 127.0.0.1:802 weight=1;
      server 127.0.0.1:803 weight=1;
    }

    server {
      listen 801;
      server_name blog1;
      root /home/www/blog_1;
      index index.html;
    }

    server {
      listen 802;
      server_name blog2;
      root /home/www/blog_2;
      index index.html;
    }
    server {
      listen 803;
      server_name blog3;
      root /home/www/blog_3;
      index index.html;
    }

    server {
        listen       80;
        server_name  localhost;

        location / {

         proxy_pass http://blog_server;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```

> 这样配置后,重启nginx
在主机中访问 127.0.0.1:12080  即可查看到负载均衡带来的效果.

##### 配置项说明:
>三个站点, 分配监听 801,802,803
```bash
    server {
      listen 801;
      server_name blog1;
      root /home/www/blog_1;
      index index.html;
    }

    server {
      listen 802;
      server_name blog2;
      root /home/www/blog_2;
      index index.html;
    }
    server {
      listen 803;
      server_name blog3;
      root /home/www/blog_3;
      index index.html;
    }
```

##### 反向代理和负载均衡的配置
```bash
    upstream blog_server {
      server 127.0.0.1:801 weight=1;
      server 127.0.0.1:802 weight=1;
      server 127.0.0.1:803 weight=1;
    }
```

关于该节点的详细说明([引用别人的](http://www.cnblogs.com/xiaogangqq123/archive/2011/03/04/1971002.html))

##### 定义负载均衡设备的 Ip及设备状态
```bash
upstream myServer {
    server 127.0.0.1:801 down;
    server 127.0.0.1:802 weight=2;
    server 127.0.0.1:803;
    server 127.0.0.1:804 backup;
}
```
##### upstream 每个设备的状态:
> down 表示单前的server暂时不参与负载
weight  默认为1.weight越大，负载的权重就越大。
max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误
fail_timeout:max_fails 次失败后，暂停的时间。
backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

##### 访问入口的配置:
```bash
  server {
        listen       80;
        server_name  localhost;

        location / {

         proxy_pass http://blog_server;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
```
*** 主要是这个配置 ***
*** proxy_pass http://blog_server; #在这个节点下启用负载均衡 ***


