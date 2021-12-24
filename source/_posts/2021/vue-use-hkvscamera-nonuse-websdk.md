---
title: 不使用海康威视的WEBSDK如何在浏览器中展示实时视频流
permalink: vue-use-hkvscamera-nonuse-websdk
category: Vue
tags:
  - Vue
  - 海康威视
date: 2021-12-24 10:52:14
---
## 前言

海康威视官方提供了Web3.0和Web3.2的SDK,

Web3.0需要浏览器支持NPAPI,但是高版本的浏览器都已经禁用了这种功能.

Web3.2需要设备支持WebSocket.

这两个Web SDK都无法适用于高版本浏览器显示海康威视老设备的视频流.

无奈只能求助官方技术支持.

官方技术支持响应还是很迅速的.描述完我的需求后,技术小哥哥(小姐姐)立马给我了一个方案(估计是被咨询了很多次):

```
您可以用RTSP协议取流，然后用第三方比如FFmpeg对数据处理成FLV或者RTMP，在谷歌网页上播放
```

又经过一通查询发现高版本浏览器也不支持RTMP流的显示了.只能使用flv,然后使用bilibili开源的flv.js进行展示.

方案确认了,那还愁啥,搞起来!嗨起来!

## 准备

因为我的项目部署环境是windows,所以所有软件都是windows版本.

ffmpeg下载地址

```
http://ffmpeg.org/download.html#build-windows
```

通过查询,rtsp要处理成flv需要借助nginx的nginx-http-flv-module模块.windows下编译带该模块的nginx超级麻烦,不过有人在CSDN上提供了编译好后的可用包.

下载地址:

```
https://download.csdn.net/download/KayChanGEEK/12270210
```

为了方便测试,建议再下载一个vlc:

```
https://www.videolan.org/
```

## Nginx配置

配置文件参考

```
#user  nobody;
# multiple workers works !
worker_processes  2;
 
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
 
#pid        logs/nginx.pid;
 
events {
    worker_connections  8192;
    # max value 32768, nginx recycling connections+registry optimization = 
    #   this.value * 20 = max concurrent connections currently tested with one worker
    #   C1000K should be possible depending there is enough ram/cpu power
    # multi_accept on;
}
 
rtmp {
    server {
        listen 1935;
        chunk_size 4000;
        application live {
             live on;
        }
    }
}
 
http {
    #include      /nginx/conf/naxsi_core.rules;
    include       mime.types;
    default_type  application/octet-stream;
 
    #log_format  main  '$remote_addr:$remote_port - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
 
    #access_log  logs/access.log  main;
 
#     # loadbalancing PHP
#     upstream myLoadBalancer {
#         server 127.0.0.1:9001 weight=1 fail_timeout=5;
#         server 127.0.0.1:9002 weight=1 fail_timeout=5;
#         server 127.0.0.1:9003 weight=1 fail_timeout=5;
#         server 127.0.0.1:9004 weight=1 fail_timeout=5;
#         server 127.0.0.1:9005 weight=1 fail_timeout=5;
#         server 127.0.0.1:9006 weight=1 fail_timeout=5;
#         server 127.0.0.1:9007 weight=1 fail_timeout=5;
#         server 127.0.0.1:9008 weight=1 fail_timeout=5;
#         server 127.0.0.1:9009 weight=1 fail_timeout=5;
#         server 127.0.0.1:9010 weight=1 fail_timeout=5;
#         least_conn;
#     }
 
    sendfile        off;
    #tcp_nopush     on;
 
    server_names_hash_bucket_size 128;
 
## Start: Timeouts ##
    client_body_timeout   10;
    client_header_timeout 10;
    keepalive_timeout     30;
    send_timeout          10;
    keepalive_requests    10;
## End: Timeouts ##
 
    #gzip  on;
 
    server {
        listen       80;
        server_name  localhost;
 
 
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }
        location /stat.xsl {
            root nginx-rtmp-module/;
        }
        location /control {
            rtmp_control all;
        }
 
    # 重要 http-flv  转流配置 
        location /live {
            flv_live on; #打开 HTTP 播放 FLV 直播流功能
            chunked_transfer_encoding on; #支持 'Transfer-Encoding: chunked' 方式回复
 
            add_header 'Access-Control-Allow-Origin' '*'; #添加额外的 HTTP 头
            add_header 'Access-Control-Allow-Credentials' 'true'; #添加额外的 HTTP 头
            add_header 'Cache-Control' 'no-cache';
        }
 
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
 
        ## Caching Static Files, put before first location
        #location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        #    expires 14d;
        #    add_header Vary Accept-Encoding;
        #}
 
# For Naxsi remove the single # line for learn mode, or the ## lines for full WAF mode
        location / {
      add_header Cache-Control no-cache;
      add_header 'Access-Control-Allow-Origin' '*' always;
      add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
      add_header 'Access-Control-Allow-Headers' 'Range';
    
            #include    /nginx/conf/mysite.rules; # see also http block naxsi include line
            ##SecRulesEnabled;
         ##DeniedUrl "/RequestDenied";
         ##CheckRule "$SQL >= 8" BLOCK;
         ##CheckRule "$RFI >= 8" BLOCK;
         ##CheckRule "$TRAVERSAL >= 4" BLOCK;
         ##CheckRule "$XSS >= 8" BLOCK;
            root   html;
            index  index.html index.htm;
        }
 
# For Naxsi remove the ## lines for full WAF mode, redirect location block used by naxsi
        ##location /RequestDenied {
        ##    return 412;
        ##}
 
## Lua examples !
#         location /robots.txt {
#           rewrite_by_lua '
#             if ngx.var.http_host ~= "localhost" then
#               return ngx.exec("/robots_disallow.txt");
#             end
#           ';
#         }
 
        #error_page  404              /404.html;
 
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
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
        #    fastcgi_pass   127.0.0.1:9000; # single backend process
        #    fastcgi_pass   myLoadBalancer; # or multiple, see example above
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
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
    #    listen       443 ssl spdy;
    #    server_name  localhost;
 
    #    ssl                  on;
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
    #    ssl_session_timeout  5m;
    #    ssl_prefer_server_ciphers On;
    #    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    #    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:ECDH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!eNULL:!MD5:!DSS:!EXP:!ADH:!LOW:!MEDIUM;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
}

```

有两个地方需要特殊说明:

第一处:这里是配置rtmp, 1935端口是默认端口.

```
rtmp {
    server {
        listen 1935;
        chunk_size 4000;
        application live {
             live on;
        }
    }
}
```

第二处: flv拉流的配置

```
location /live {
            flv_live on; #打开 HTTP 播放 FLV 直播流功能
            chunked_transfer_encoding on; #支持 'Transfer-Encoding: chunked' 方式回复
 
            add_header 'Access-Control-Allow-Origin' '*'; #添加额外的 HTTP 头
            add_header 'Access-Control-Allow-Credentials' 'true'; #添加额外的 HTTP 头
            add_header 'Cache-Control' 'no-cache';
        }
```

## 海康威视摄像头RTSP流获取

网上一堆介绍,可自行搜索学习.

```
rtsp://admin:123456@192.168.10.54:554/Streaming/Channels/101?transportmode=unicast

rtsp:协议
admin:摄像头登录用户名
123456:摄像头登录密码
192.168.10.54:摄像头ip
554:拉rtsp流的端口
101:通道表示(主码流通道)
transportmode=unicast:传输模式
```

## 使用ffmpeg进行流转换

在命令行中输入一下命令

```
ffmpeg  -rtsp_transport tcp -i rtsp://admin:123456@192.168.10.54:554/Streaming/Channels/101?transportmode=unicast -vcodec copy -an -acodec copy -f flv rtmp://127.0.0.1:1935/live/mystream

rtmp://127.0.0.1:1935/live/mystream: 这个地址是nginx中配置的rtmp节点的地址, live是配置的app名称,mystream可以随意指定
```

出现下图所示效果且进程不终止,表明转换流成功了.

![image-20211213174237900](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211213174237900.png)

有了上面的rtmp地址后,我们可以知道flv取流的地址为

```
http://127.0.0.1/live?port=1935&app=live&stream=mystream

这是通过nginx中http节点配置的.请注意app的值和stream的值
对应的是
rtmp://127.0.0.1:1935/live/mystream
中的 live和mystream
```

有了flv拉流的地址后,我们就可以在vlc中进行校验

## 在vlc中进行测试

点击vlc菜单中的 "媒体",然后选择 "流"

然后选择"网络"

![image-20211213175059382](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211213175059382.png)

然后将右下方的 "串流" 选择为 "播放",在等待几秒后就可以看到正常的视频图像了.

![image-20211213175204858](https://gitee.com/dodoliu/dodoliuimg/raw/master/2021/image-20211213175204858.png)

如果没有显示,请继续爬坑吧!!!

## 在vue中显示

我的前端项目是个vue项目.

先安装flv.js

```
npm install --save flv.js
```

html主要代码

```
  <video
    id="vPull"
    autoplay
    style="height: 100%; width: 100%; object-fit: fill"
    muted
  ></video>
```

js主要代码.

参考: https://blog.csdn.net/HJFQC/article/details/109626836

感谢原作者

```
mounted() {
  this.play("http://127.0.0.1/live?port=1935&app=live&stream=mystream");
}
```

我需要想说明一下如何调用,在mounted方法中直接调用play方法,然后传入准备好的flv流地址.

没有意外的话,vue项目启动后就可以正常查看视频了.

## .net core后台动态切换摄像头地址

经过上面所有操作后,我们已经可以在前端进行视频展示了.但是如果想切换摄像头该怎么搞?

可以参考以下思路:

通过后端代码进行ffmpeg的进程创建和关闭.

我的后端是.net core项目,参考代码如下:

```
/// <summary>
/// 启动ffmpeg进程
/// </summary>
/// <returns></returns>
[HttpGet("TestCmd54")]
public async Task<int> TestCmd54()
{    
    var psi = new System.Diagnostics.ProcessStartInfo("D:\\App\\ffmpeg\\bin\\ffmpeg.exe", "-rtsp_transport tcp -i rtsp://admin:123456@192.168.10.54:554/Streaming/Channels/101?transportmode=unicast -vcodec copy -an -acodec copy -f flv rtmp://127.0.0.1:1935/live/mystream");
    var ss = System.Diagnostics.Process.Start(psi);
    return await Task.FromResult(ss.Id);
}
/// <summary>
/// 关闭ffmpeg进程
/// </summary>
/// <returns></returns>
[HttpGet("TestCmd54Stop")]
public async Task<int> TestCmd54Stop(int processId)
{ 
    var stopProcess = System.Diagnostics.Process.GetProcessById(processId);
    if (!stopProcess.HasExited)
    {
        stopProcess.Kill();
    }

    return await Task.FromResult(1);
}
```
