---
title: 自定义generator
permalink: rails-tip-custom-generator
category: Ruby On Rails
tags:
  - Ruby On Rails
  - Ruby
date: 2017-03-04 11:57:15
---
在Rails项目中,scaffold提供了生成初始代码的强大功能.但任然不能满足日常开发需求.针对部分重复的代码手动ctrl+c,ctrl+v总显得太low.如果能自定义scaffold那就完美了.
庆幸的是Rails提供了这种功能.下面以我的开源项目opendoc为例,来自定义scaffold.
首先来生成generator.
```ruby
rails generate generator opendoc_scaffold
```
上面的命令会生成下面这些文件
```bash
create  lib/generators/opendoc_scaffold
create  lib/generators/opendoc_scaffold/opendoc_scaffold_generator.rb
create  lib/generators/opendoc_scaffold/USAGE
create  lib/generators/opendoc_scaffold/templates
invoke  test_unit
create    test/lib/generators/opendoc_scaffold_generator_test.rb
```
单从名字上看,我们已经大体上知道这些文件是干啥用的了.
USAGE是定义一些说明.
templates文件夹下放一些模板.
opendoc_scaffold_generator.rb是启动文件.内容如下
```ruby
class OpendocScaffoldGenerator < Rails::Generators::NamedBase
  source_root File.expand_path('../templates', __FILE__) #指定模板根目录
  #创建一个启动方法,以下代码不是自动生成的
  def copy_opendoc_scaffold_file
    #controller
    template 'controller.rb', "app/controllers/backend/#{table_name}_controller.rb"
    #model
    template 'model.rb', "app/models/#{singular_table_name}.rb"
    #views
    copy_file 'erb/new.html.erb', "app/views/backend/#{file_name}/new.html.erb"
    copy_file 'erb/edit.html.erb', "app/views/backend/#{file_name}/edit.html.erb"
    template 'erb/index.html.erb', "app/views/backend/#{file_name}/index.html.erb"
    template 'erb/_form.html.erb', "app/views/backend/#{file_name}/_form.html.erb"
  end
end
```
上诉代码中用到的template方法的意思是复制模板到指定的地方.
比如自动生成model.下面这些代码是我自定义的model模板.
```html
require 'uuidtools'

class <%= singular_table_name.capitalize %> < ApplicationRecord
  enum status: [:archived, :active]

  #validates
  validates :TODO, presence: true, length: { maximum: 50 }
  validates :TODO, numericality: true

  #scope
  default_scope { where("status>?", <%=  singular_table_name.capitalize %>.statuses[:archived]) }
  scope :name_like, ->(name){ where "name like ? ", "%#{sanitize_sql_like(name)}%" }  #防sql注入

  #假删除
  def self.delete(<%= singular_table_name %>)
    <%= singular_table_name %>.status = :archived
    <%= singular_table_name %>.save
  end

  #设置属性值
  def self.set_attribute(<%= singular_table_name %>_params)
    <%= singular_table_name %> = <%= singular_table_name.capitalize %>.new(<%= singular_table_name %>_params)
    <%= singular_table_name %>.sid = UUIDTools::UUID.timestamp_create
    <%= singular_table_name %>.status = :active
    <%= singular_table_name %>
  end
end
```
上面model模板中用到的 singular_table_name 以及 table_name 等这些方法都在 Rails::Generators::NamedBase [这个类下](http://api.rubyonrails.org/classes/Rails/Generators/NamedBase.html).
定义完模板后就可以像使用scaffold一样使用我们的自定义generator了.
```ruby
rails generate opendoc_scaffold groups
rails generate opendoc_scaffold members
```


