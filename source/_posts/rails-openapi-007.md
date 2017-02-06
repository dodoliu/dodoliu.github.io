---
title: 用Rails实现一个系统(friendly_id的使用)
category: Ruby On Rails
tags:
  - Ruby On Rails
  - Ruby
date: 2017-02-06 00:11:18
---
##### 简介
如果不对url进行模糊处理,通常的url会像下面这样子:
http://localhost:3000/backend/groups/1/edit
http://localhost:3000/backend/groups/1/interfaces/1/edit
这直接暴露了数据库里有多少条记录.为了降低风险,我们需要借助friendly_id这个gem对url进行模糊处理.
处理的结果应该是下面这样:
http://localhost:3000/backend/groups/0f7094c6-ec6e-11e6-a542-7cd1c3f0bed9/edit
http://localhost:3000/backend/groups/0f7094c6-ec6e-11e6-a542-7cd1c3f0bed9/interfaces/48e86a4e-ec6e-11e6-a542-7cd1c3f0bed9/edit

##### 实现过程
添加[friendly_id](https://github.com/norman/friendly_id)这个gem
```ruby
#Gemfile
gem 'friendly_id', '~> 5.1.0'
```
按照官方教程进行初始化:
```ruby
rails g friendly_id
rails db:migrate
```
###### 先来实现编辑group的url模糊处理
```ruby
#先为groups表添加slug字段
rails g migration add_column_slug_to_groups slug:string:uuiq
rails db:migrate
```
然后修改group model
```ruby
#app/modles/group.rb
#添加如下代码
extend FriendlyId
friendly_id :sid, use: :slugged  #使用groups中的sid字段作为slug
```
然后controller中所有查询都通过friendly
```ruby
#app/controllers/backend/groups_controller.rb
#比如:
@group = Group.friendly.find(params[:id])
```
完成以上操作后即实现了http://localhost:3000/backend/groups/1/edit这个url的模糊处理.
查看http://localhost:3000/backend/groups的html你会发现 编辑 对应的href的值已经模糊处理,但是删除的href还是没处理的状态.这个时候修改groups/index.html.erb
```html
#app/views/backend/groups/index.html.erb
<%= link_to '删除', backend_group_path(group.id), method: :delete, data: { confirm: '确定删除吗?' }, class: 'edit'  %>
为
<%= link_to '删除', backend_group_path(group.sid), method: :delete, data: { confirm: '确定删除吗?' }, class: 'edit'  %>
```
###### 处理interface
如果现在什么都不做,直接打开interface,你会发现interface的首页无法正常显示数据了.因为用于查询的group_id由数字变成了一长串的字符串.
接下来我们修复这个问题.
先按照修改group的方式修改interface使用friendly.因为group和interface有has_many关联,所以需要一些特殊处理!
```ruby
#先为interfaces添加groups的sid外键关联
rails g migration add_column_group_sid_to_interfaces group_sid:string:{50}
rails db:migrate
```
然后修改set_attribute方法传入 params[:group_id]
```ruby
#app/models/intreface.rb
def self.set_attribute(interface_params, group_sid)
    inter = Interface.new(interface_params)
    inter.sid = UUIDTools::UUID.timestamp_create
    inter.group_sid = group_sid
    inter.status = 1
    inter
end
```
这样处理后,新增interface的时候,就会把group的sid插入到interface的group_sid字段中.

###### 其它
也可通过重写model的to_param方法模糊处理url,比如
```ruby
def to_param
    "#{id}_slfkjsdf"
end
```


以上就是friendly_id的简单用法.如有不对的地方欢迎指正!






