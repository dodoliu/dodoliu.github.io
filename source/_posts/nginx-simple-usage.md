---
title: 虚拟机下CentOS7安装Nginx及简单用法
category: Linux
tags:
  - Linux
  - CentOS
  - Nginx
date: 2016-05-20 22:36:41
---
### 目的:
> 熟悉nginx的安装与使用

### 环境:
> VirtualBox: 5.0.20
CentOS: CentOS-7-x86_64-Minimal-1511
Nginx: 1.10.0

### 安装步骤:
> Nginx下载地址: http://nginx.org/download/nginx-1.10.0.tar.gz
下载完成后解压: tar zxvf nginx-1.10.0.tar.gz
然后进入解压目录: cd nginx-1.10.0
编译安装:
./configure
make && make install

默认安装目录在: /usr/local/nginx

### 启动:
> cd到 nginx下的 sbin目录
执行命令 ./nginx

### 停止:
> 使用ps命令查看 nginx的pid
ps -A | grep nginx
从容停止: kill -QUIT 查询到的nginx的pid
快速停止: kill -TERM 查询到的nginx的pid
强制停止: kill -9 查询到的nginx的pid

### 重启:
> cd到nginx的sbin目录
./nginx -s reload (这个命令在我测试的时候发现不能完全重启...不知为何...之后我都选择先停止,再启动)

### 修改conf:
> 默认配置文件在nginx的根目录的conf中
进入该目录 vim nginx.conf 进行编辑.
编辑完成后一般需要先检测配置文件是否正常再重启系统nginx,以免配置文件错误.
检测命令(假如在nginx的sbin目录下): ./nginx -t