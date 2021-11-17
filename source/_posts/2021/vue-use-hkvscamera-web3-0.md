---
title: Vue下使用海康威视WEB无插件开发包V3.2的使用记录
permalink: vue-use-hkvscamera-web3-0
category: Vue
tags:
  - Vue
  - 海康威视
date: 2021-11-17 16:41:05
---
## 简介

最近做的一个项目需要在Web页面上展示视频图像信息.

项目中使用的摄像头是海康威视的.经过一番捣鼓后终于可以正常显示图像了.

于是做个记录,供其他同学爬坑.

## 开发准备

###### WEB无插件开发包 V3.2 官方下载地址:

https://open.hikvision.com/download/5cda567cf47ae80dd41a54b3?type=10&id=4c945d18fa5f49638ce517ec32e24e24

![image-20211116165646268](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211116165646268.png)

下载完成解压后包含以下内容:

![image-20211116165802938](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211116165802938.png)



###### 设备网络搜索软件下载地址:

https://www.hikvision.com/cn/support/Downloads/Desktop-Application/HikvisionTools/?q=%E5%B7%A5%E5%85%B7%E8%BD%AF%E4%BB%B6%EF%BC%88hikvision%20tools%EF%BC%89&position=1

![image-20211116170730350](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211116170730350.png)

这是海康威视官方提供的用于发现相同局域网下所有设备的工具.效果如下图:

![image-20211116170905290](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211116170905290.png)

###### 一个海康威视的摄像头,需要支持WebSocket.

我这边测试用的摄像头是这一款:

![image-20211116170300397](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211116170300397.png)

==测试环境下需要保证摄像头和开发机器在同一个局域网内==

tips:

1. 如果通过 设备网络搜索软件SADP 无法发现你需要是设备,则表明你的开发机器的网络和设备的网络不通.
2. 将摄像头设置为使用 DHCP

![image-20211116171337535](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211116171337535.png)

3. Vue开发环境下无法看到正常的视频图像,需要使用Nginx进行代理

## 快乐的码代码

###### Vue代码可以借鉴这篇文章的


https://blog.csdn.net/Vslong/article/details/118517641

按照上面这篇文章码完Vue代码后如果想正常看到视频图片还需要完成以下操作:

###### 设置Vue代理


在Vue的Config文件夹下的index.js配置文件中设置proxyTable

参考:

```
proxyTable: {
  '/ISAPI': {//配置代理地址，前端请求的所有接口都需要带的前缀
    target: 'http://192.168.10.51:12345',  #我本地监控的Web3.2无插件Nginx代理地址
    changeOrigin: true,//是否进行跨域
    secure: false,
    // logLevel: 'debug',
  },
  '/SDK': {
    target: 'http://192.168.10.51:12345',
    changeOrigin: true,
    secure: false,
    // logLevel: 'debug',
  }
},
```

###### 然后编译发布Vue代码,然后修改Nginx配置.

参考:

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;
    #access_log      off;
    client_max_body_size 50m;
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       12345;
        server_name  192.168.10.51;

        #charset koi8-r;

        access_log  logs/host.access.log  main;
        #websocket相关配置
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header X-real-ip $remote_addr;
      proxy_set_header X-Forwarded-For $remote_addr;
    
        proxy_set_header 'sec-ch-ua' "";
        proxy_set_header 'sec-ch-ua-mobile' "";
        proxy_set_header 'sec-ch-ua-platform:' "";
        proxy_set_header 'Sec-Fetch-Dest' "";
        proxy_set_header 'Sec-Fetch-Mode' "";
        proxy_set_header 'Sec-Fetch-Site' "";

        location / {
            root "../dist";
            index  index.html index.htm;
        }

      location ~ /ISAPI|SDK/ {
          if ($http_cookie ~ "webVideoCtrlProxy=(.+)") {
          proxy_pass http://$cookie_webVideoCtrlProxy;
          break;
          }
      }
      location ^~ /webSocketVideoCtrlProxy {
          #web socket
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
          proxy_set_header Host $host;

          if ($http_cookie ~ "webVideoCtrlProxyWs=(.+)") {
          proxy_pass http://$cookie_webVideoCtrlProxyWs/$cookie_webVideoCtrlProxyWsChannel?$args;
          break;
          }
          if ($http_cookie ~ "webVideoCtrlProxyWss=(.+)") {
          proxy_pass http://$cookie_webVideoCtrlProxyWss/$cookie_webVideoCtrlProxyWsChannel?$args;
          break;
          }
      }
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        # error_page  302  /50x.html;
        # location = /50x.html {
        #     root   html;
        # }
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


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

以上是无插件开发包自带的nginx配置,只简单修改了三处:

```
监控端口:
listen       12345;
server_name  192.168.10.51;
       
静态文件根目录:       
location / {
    root "../dist";
    index  index.html index.htm;
}

添加:
proxy_set_header 'sec-ch-ua' "";
proxy_set_header 'sec-ch-ua-mobile' "";
proxy_set_header 'sec-ch-ua-platform:' "";
proxy_set_header 'Sec-Fetch-Dest' "";
proxy_set_header 'Sec-Fetch-Mode' "";
proxy_set_header 'Sec-Fetch-Site' "";
```

然后重启nginx,访问你的项目web地址应该就能看到视频图像了.

如果还是看不到就继续爬坑吧!!!

## 其它可能会出现的问题

###### 如果出现了 Too-many-header 这个错误

参考: https://www.scaugreen.cn/posts/65442/

## 写在最后

感谢本人在爬坑过程中翻过的所有文章.
