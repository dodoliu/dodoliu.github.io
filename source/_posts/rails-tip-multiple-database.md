---
title: Rials Tip:多库操作
category: Ruby On Rails
tags:
  - Ruby On Rails
date: 2016-07-22 17:53:19
---
我有个需求需要Rails连接mysql和sqlserver,这就需要rails的多数据库操作支持.
rails对mysql原生支持是很好的,所以不需要改动什么.但是操作sqlserver就需要手动配置一下了.
查了一堆资料后记录如下:
需要用到的gem:
[github地址](https://github.com/rails-sqlserver/activerecord-sqlserver-adapter)
```ruby
gem 'tiny_tds'
gem 'activerecord-sqlserver-adapter', '~> 4.2.0'
```
database.yml针对sqlserver的配置
```ruby
sqlserver_db:
  adapter: sqlserver
  encoding: utf8
  host: localhost
  port: 6381
  database: sqlserver_db
  username: sa
  password: 123456
```
model的设置:
[具体原因参考](http://technology.customink.com/blog/2015/06/22/rails-multi-database-best-practices-roundup/)
在models文件夹下创建一个sqlserver_base.rb的文件,代码为:
```ruby
class SqlserverBase < ActiveRecord::Base
  establish_connection configurations['sqlserver_db']  #切换到sqlserver
  self.abstract_class = true  #声明该类为抽象类
end
```
sqlserver表对应的model这样声明:
```ruby
class Friend < SqlserverBase
  self.table_name = 'Friend'
end
```