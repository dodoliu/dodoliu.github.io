---
title: 基于一个已经存在的MySql库创建Rails的Model
permalink: ruby-existing-mysql-generate-model
category: Ruby On Rails
tags:
  - Ruby On Rails
date: 2016-06-18 15:16:27
---
我有一个很奇葩的需求!
我需要在不改变现有MySql表结构的情况下使用Rails访问MySql.目前只需要用到ActiveRecord的查询功能.
为何会有这种奇葩需求呢?!
现有系统的数据架构是运行很久且稳定的,且现有系统不允许添加一些方便平时使用的小功能.那只能另辟蹊径了.又因为我想尝试下在Windows下搞Rails开发.于是乎,这个奇葩需求就产生了.
既然有想法了,那么就动手做吧!
windows下安装Rails环境就不说了,网上一堆.
访问MySql使用mysql2.如果在确认数据库配置没有任何问题的前提下,还是无法访问MySql.请尝试下载安装MySql的动态链接库文件libmysql.dll.这是[下载地址](http://www.mysql.com/downloads/connector/c/)
如果没有意外,安装完动态链接库后一切就正常了.如何还不正常...请问Google!
这个需求的唯一关键点就是设置Model的默认访问table名称.因为按照Rails的约定,Model是单数,数据库中的表必须是复数.比如有Account这个Model,如果数据库中的表是account,那么需要按照下面代码中的设置来创建Model.
```ruby
class Account < ActiveRecord::Base
    self.table_name = 'account'
end
```
这样设置之后,就可以愉快的使用ActiveRecord的强大功能了.

