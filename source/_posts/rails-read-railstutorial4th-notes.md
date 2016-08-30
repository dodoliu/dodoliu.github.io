---
title: 看《Ruby on Rails 教程（第四版）》杂记 (持续更新...)
category: Ruby On Rails
tags:
  - Ruby On Rails
  - Ruby
date: 2016-08-29 22:07:13
---
以下内容均为《Ruby on Rails 教程（第四版）》学习摘要!

##### 编写测试的指导方针
与应用代码相比,如果测试代码特别简短,倾向于先编写测试;
如果对想实现的功能不是特别清楚,倾向于先编写应用代码,然后再编写测试,并改进实现方式;
安全是头等大事,保险起见,要为安全相关的功能线编写测试;
只要发现一个问题,就编写一个测试重现这种问题,避免回归,然后再编写应用代码修正问题;
尽量不为以后可能修改的代码(例如HTML结构的细节)编写测试;
重构之前要编写测试,集中测试容易出错的代码.
在实际的开发中,根据上述方针,我们一般先编写控制器和模型测试,然后再编写集成测试(测试模型、试视图和控制器在一起时的表现)。如果应用代码很容易出错，或者经常变动（视图就是这样），我们就完全不测试。
##### assert_select
test中的辅助函数,用于检测html中是否有指定的html标签,这个方法有时也叫"选择符"
assert_select 'title','Home I Ruby on Rails Tutorial Sample App' #含义: 检测有没有< title \>标签,以及其中的内容是不是 "Home I Ruby on Rails Tutorial Sample App"
##### provide 帮助方法
```ruby
<% provide(:title, "Home") %>

<html>
<head>
<title><%= yield(:title) %></title>
</head>
</html>
```
##### 使用git开发的步骤
有新的需求时,创建一个分支,再分支上进行开发,开发完成后合并到主分支.
```git
git checkout master #切换到master分支
git merge static-pages #把static-pages分支merge到master分支
```
##### ps 命令
```bash
ps aux  #查看系统进程
ps aux | grep spring #查找spring
```
##### Guard的配置
```ruby
 # Defines the matching rules for Guard.
guard :minitest, spring: true, all_on_start: false do
    watch(%r{^test/(.*)/?(.*)_test\.rb$})
    watch('test/test_helper.rb') { 'test' }
    watch('config/routes.rb') { integration_tests }
    watch(%r{^app/models/(.*?)\.rb$}) do |matches|
        "test/models/#{matches[1]}_test.rb"
    end
    watch(%r{^app/controllers/(.*?)_controller\.rb$}) do |matches|
        resource_tests(matches[1])
    end
    watch(%r{^app/views/([^/]*?)/.*\.html\.erb$}) do |matches|
            ["test/controllers/#{matches[1]}_controller_test.rb"] +
        integration_tests(matches[1])
    end
    watch(%r{^app/helpers/(.*?)_helper\.rb$}) do |matches|
        integration_tests(matches[1])
    end
    watch('app/views/layouts/application.html.erb') do
            'test/integration/site_layout_test.rb'
    end
    watch('app/helpers/sessions_helper.rb') do
        integration_tests << 'test/helpers/sessions_helper_test.rb'
    end
    watch('app/controllers/sessions_controller.rb') do ['test/controllers/sessions_controller_test.rb','test/integration/users_login_test.rb']
    end
    watch('app/controllers/account_activations_controller.rb') do 'test/integration/users_signup_test.rb'
    end
    watch(%r{app/views/users/*}) do
            resource_tests('users') +
    ['test/integration/microposts_interface_test.rb'] 
    end
end

# Returns the integration tests corresponding to the given resource.
def integration_tests(resource = :all) if resource == :all
    Dir["test/integration/*"] else
    Dir["test/integration/#{resource}_*.rb"] end
end

# Returns the controller tests corresponding to the given resource.
def controller_test(resource) "test/controllers/#{resource}_controller_test.rb"
end
    # Returns all tests for the given resource.
def resource_tests(resource)
    integration_tests(resource) << controller_test(resource)
end
#下面这行会让 Guard 使用 Rails 提供的 Spring 服务器减少加载时间,而且启动时不运行整个测试组件。 
guard :minitest, spring: true, all_on_start: false do
#使用 Guard 时,为了避免 Spring 和 Git 发生冲突,应该把 spring/ 文件夹加到 .gitignore 文件中,让 Git 忽 略这个文件夹。
```
##### bang方法约定
方法名后加一个感叹号的方法称为"炸弹"(bang)方法.bang方法的含义是 方法处理的结果会改变对象的值.比如:
```bash
2.3.1 :001 > a = %w(2 3 5)
 => ["2", "3", "5"]
2.3.1 :002 > a.sort
 => ["2", "3", "5"]
2.3.1 :003 > a = %w(5 9 2)
 => ["5", "9", "2"]
2.3.1 :004 > a.sort
 => ["2", "5", "9"]
2.3.1 :005 > a
 => ["5", "9", "2"]
2.3.1 :006 > a.sort!
 => ["2", "5", "9"]
2.3.1 :007 > a
 => ["2", "5", "9"]
```
##### 数组的 push | <<
为数组添加元素可以使用 push方法,也可以使用 << 运算符.
```bash
2.3.1 :008 > a << 'bbb' << 'dddd'
 => ["2", "5", "9", "bbb", "dddd"]
```
##### map块变量调用方法简写形式 &
```ruby
2.3.1 :011 > a.map {|item| item.upcase }
 => ["2", "5", "9", "BBB", "DDDD"]
2.3.1 :012 > a.map(&:downcase)
 => ["2", "5", "9", "bbb", "dddd"]
```


