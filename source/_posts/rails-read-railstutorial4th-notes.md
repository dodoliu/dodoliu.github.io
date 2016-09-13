---
title: 看《Ruby on Rails 教程（第四版）》杂记 (持续更新...)
category: Ruby On Rails
tags:
  - Ruby On Rails
  - Ruby
date: 2016-08-29 22:07:13
---
以下内容均为《Ruby on Rails 教程（第四版）》学习摘要及总结!

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
一些用法:
| 代码        | 匹配HTML         |
| ------------- |:-------------:|
| assert_select 'div'      | < div>foobar< /div> |
| assert_select 'div','foobar'     | < div>foorbar< /div> |
| assert_select 'div.nav'     | < div class='nav'>foorbar< /div> |
| assert_select 'div#profile'     | < div id='profile'>foorbar< /div> |
| assert_select 'div[name=yo]'     | < div name='yo'>hey< /div> |
| assert_select "a[href=?]",'/',count:1    | < div href='/'>foo< /div> |
| assert_select "a[href=?]",'/',text: "foo"    | < div href='/'>foo< /div> |


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
##### hash
```ruby
#几种写法 及 多层嵌套后的hash取值方式
2.3.1 :001 > a = { "a1" => 1, "a2" => 2  }
 => {"a1"=>1, "a2"=>2}
2.3.1 :002 > b = { :b1 => 1, :b2 => 2 }
 => {:b1=>1, :b2=>2}
2.3.1 :003 > c = { c1: 1, c2: 2 }
 => {:c1=>1, :c2=>2}

 #多层嵌套后的hash取值
 2.3.1 :004 > d = { d1: { dd1: { ddd1: 6, ddd2: 7}, dd2: 4 }, d2: 3 }
 => {:d1=>{:dd1=>{:ddd1=>6, :ddd2=>7}, :dd2=>4}, :d2=>3}

2.3.1 :011 > d[:d1]
 => {:dd1=>{:ddd1=>6, :ddd2=>7}, :dd2=>4}
2.3.1 :012 > d[:d1][:dd1]
 => {:ddd1=>6, :ddd2=>7}
2.3.1 :013 > d[:d1][:dd1][:ddd2]
 => 7

#hash的merge方法
2.3.1 :018 > a
 => {"a1"=>1, "a2"=>2}
2.3.1 :019 > b
 => {:b1=>1, :b2=>2}
2.3.1 :020 > a.merge b
 => {"a1"=>1, "a2"=>2, :b1=>1, :b2=>2}

#函数调用时,如果哈希是最后一个参数,可以省略花括号.比如:
stylesheet_link_tag 'application', { media: 'all', 'data-turbolinks-track': 'reload'}
stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload'
```
##### 字符串、数组、hash、Range的声明
```ruby
a = ''
a = String.new
a = String.new('dd')
b = []
b = Array.new
c = {}
c = Hash.new
d = Range.new(0,2)  #声明一个范围
```
##### 预处理引擎的执行顺序
从右向左执行
foobar.js.erb.coffee #先通过coffee处理,再通过ruby处理
##### 路由的 _path 和 _url
root_path -> '/'
root_url -> 'http://www.example.com/'
约定 只有重定向使用_url形式,其余都使用_path形式.(因为HTTP标准严格要求重定向的URL必须完整.不过在大多数浏览器中,两种形式都可以正常使用)
##### 测试方法 assert_template
检查路由是否使用正确的视图渲染
get root_path
assert_template 'static_pages/home'
\#请求root路由,看root路由是否使用了static_pages下的home视图渲染.
##### 沙盒模式
rails console --sandbox
在沙盒模式下做的所有修改都不会影响实际数据.
在控制台下可以使用 reload方法重新加载环境
##### ActiveRecord的一些方法
user.new #在内存中创建对象
user.save #保存到数据库,save方法会返回true和false
以上两步和
user.create #create方法会返回对象的实例
等效.
user.valid? #只检测对象是否有效
user.destroy #create方法的逆操作,也会返回对象的实例.(销毁的对象还在内存中)
user.update_attributes(name: 'The Dude', email: 'dude@abides.org') #更新name和email属性,返回ture和false.该方法无法跳过验证
user.update_attribute(:name, 'El Duderino') #更新单个属性,该方法可以跳过验证
##### Model的验证
validate :email, format: { with: /<regular expression>/ } #validate可以有format参数,提供正则验证
```ruby
VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
validates :email, presence: true, length: {maximum: 255 }, format: {with: VALID_EMAIL_REGEX }
```
##### 正则表达式 /i,/g,/m,/gi,/ig 区别
/i (忽略大小写)
/g (全文查找出现的所有匹配字符)
/m (多行查找)
/gi(全文查找、忽略大小写)
/ig(全文查找、忽略大小写)
##### 产生7位随机的字符
('a'..'z').to_a.shuffle[0..7].join
##### dup方法复制一个对象
duplicate_user = @user.dup
##### model的 before_save 方法
```ruby
class User < ApplicationRecord
    before_save { self.email = email.downcase } #在执行save方法前,先把email转换为小写
    validates :name, presence: true, length: { maximum: 50 }
end
```
##### has_secure_password 方法
```ruby
calss User < ApplicationRecord
    has_secure_password
end
```
在模型中调用了该方法后,会自动添加以下功能:
在数据库中的password_digest列存储安全的密码哈希值;
获得一对虚拟属性,password和password_confirmation,而且创建用户对象时会执行存在性验证和匹配验证;
获得authenticate方法,如果密码正确,返回对应的用户对象,否则返回false;
这个方法需要表有 password_digest字段




























