---
title: 记录一次VirtualBox安装Centos7的网络配置
category: Linux
tags:
  - Linux
  - CentOS
  - VirtualBox
date: 2016-05-18 13:07:49
---

### 目的:
> 了解虚拟机的网络配置

### 要求:
> 虚拟机和主机能互通
虚拟机要能访问网络

### 限制条件:
> 公司环境下不允许分配独立IP

### 环境:
> win7 or OSX: 10.11.4
VirtualBox: 5.0.20
CentOS: CentOS-7-x86_64-Minimal-1511

### 虚拟机要能访问网络:
> CentOS7 默认使用ip查看IP地址
配置文件位置:  /etc/sysconfig/network-scripts/
需要修改的配置项说明:
BOOTPROTO=dhcp #自动获取ip
ONBOOT=yes #开机启动网卡
重启网卡命令: service network restart
ip addr 查看ip信息

VirtualBox 网络设置为 NAT模式,没有意外的话CentOS可以直接访问网络了

### 虚拟机和主机能互通:
> 配置一个端口映射的规则即可
如下图
![1](http://githubblog-10030337.file.myqcloud.com/linux-virtualbox-centos7-network.png?sign=sKS+edOG1gppcj+qhNL5Rxb5vDRhPTEwMDMwMzM3Jms9QUtJRGNDZFJ2aDZITWsyTE1SNDFEMHo1SERFbzlORndwcWcxJmU9MTQ2NjQzNTI3NCZ0PTE0NjM4NDMyNzQmcj02ODYxNDcyMjAmZj0vbGludXgtdmlydHVhbGJveC1jZW50b3M3LW5ldHdvcmsucG5nJmI9Z2l0aHViYmxvZw==)

### 配置虚拟机的22端口映射
> win下使用putty进行ssh登陆
mac下使用 ssh登陆 ssh -p 12022 root@127.0.0.1

|主机IP|主机端口|虚拟机IP|虚拟机端口|
|-----|-----|-----|-----|
|127.0.0.1|12022|10.0.2.15|22|