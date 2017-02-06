---
title: 用Rails实现一个系统(CRUD)(待完善)
category: Ruby On Rails
tags:
  - Ruby On Rails
  - Ruby
date: 2017-01-17 00:11:18
---
##### 简介
openapi用户分为两种.一种是管理员,可以直接增删改查接口文档.另一种就是只有查看权限的用户了.
因为接口文档是按照不同的客户(我们称为品牌)分类,同一类接口可能会分给多个用户查看.
所以本次首先实现"品牌"的CRUD.

##### CRUD
在开始CRUD之前,我们先来创建项目.因为是第一篇,这个步骤是少不了的.
```ruby
rails new openapi -d mysql --skip-bundle  #指定数据库为mysql,跳过rails的bundle阶段(原因大家都懂,万恶的墙)
```
不使用Rails的scaffold,因为会生成多个不需要的文件和代码.
```ruby
rails generate controller backend/groups index new edit create update destroy
#创建controller,生成6个基本的action(show我们不需要,所以不用生成).
#backend/groups 的意思是:CRUD的操作只能开放给管理员,
#所以这些功能应该都是后台的,所以我们创建一个backend文件夹用来存放这些文件.
#groups就是我们的"品牌"名称.按照rails的约定,controller应该是复数.
```
执行上面的命令的后会生成的代码我们不能直接用.需要做些修改,如下.
```ruby
class Backend::GroupsController < Backend::ApplicationController
#Backend::ApplicationController 的解释见下面的backend/application_controller.rb
  before_action :set_group, only: [:edit, :update, :destroy]

  include AppHelper

  def index
    q = params[:q]
    if !q.blank?
      @groups = Group.name_like(params[:q])
    else
      @groups = Group.all
    end
  end

  def new
    @group = Group.new
  end

  def edit
  end

  def create
    @group = Group.set_attribute group_params
    respond_to do |format|
      if @group.save
        format.html { redirect_to backend_groups_url, notice: '新增成功!' }
      else
        format.html { render :new }
      end
    end
  end

  def update
    respond_to do |format|
      if @group.update(group_params)
        format.html { redirect_to backend_groups_url, notice: '修改成功!' }
      else
        format.html { render :edit }
      end
    end
  end

  def destroy
    Group.delete(@group)
    respond_to do |format|
      format.html { redirect_to backend_groups_url, notice: '删除成功!' }
    end
  end

  private
    def set_group
      @group = Group.find(params[:id])
    end
    def group_params
      # params.fetch(:group, {}).permit(:app_id, :app_name)
      params.require(:group).permit(:id, :app_id, :name, :position, :brand, :memo)
    end
end

```
backend/application_controller.rb
```ruby
module Backend
  class ApplicationController < ActionController::Base
    protect_from_forgery with: :exception
    layout 'backend'
  end
end
```





