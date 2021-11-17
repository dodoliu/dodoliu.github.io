---
title: 如何单步调试ruby代码
permalink: ruby-debug
category: Ruby
tags:
  - Ruby
  - Ruby On Rails
  - MySql
date: 2016-07-23 14:31:48
---
C#写习惯了,当使用Sublime写Ruby时没有IDE进行debug,实在是不习惯.
怎么办?!
不能debug,创造条件也要debug!
经过一番搜索加实践,记录如下.

需要的gem
```ruby
gem install pry
gem install pry-byebug #支持ruby2.0.0,如果需要支持低版本的ruby可以使用 pry-debugger这个gem
```
安装了上述gem后,在需要debug的代码中添加 binding.pry
```ruby
require 'pry'

binding.pry
quick_sort! [66,13,51,76,81,26,57,69,23], 0, 8
```
当代码运行起来后就会停在binding.pry下一行代码的位置.
pry是不支持单步调试的,为啥呢?!
如果需要单步调试就需要 pry-byebug这个gem.
[pry文档](https://github.com/pry/pry)
[pry-byebug文档](https://github.com/deivid-rodriguez/pry-byebug)

pry-byebug的一些用法

使用简化的命令
```base
#在用户根目录下创建 .pryrc配置文件
vim ~/.pryrc
#输入内容
if defined?(PryByebug)
  Pry.commands.alias_command 'c', 'continue'
  Pry.commands.alias_command 's', 'step'
  Pry.commands.alias_command 'n', 'next'
  Pry.commands.alias_command 'f', 'finish'
end
```
如果需要通过按enter键达到执行上一次输入的命令的效果,则在配置文件中输入以下内容.
```base
# Hit Enter to repeat last command
Pry::Commands.command /^$/, "repeat last command" do
  _pry_.run_command Pry.history.to_a.last
end
```
断点用法:
```ruby
break SomeClass#run            #断点打在 SomeClass这个类下的run方法上
break Foo#bar if baz?          #当baz?返回true时,断点打在 Foo类下的bar方法上
break app/models/user.rb:15    #断点打在 user.rb的第15行
break 14                       #断点打在第14行

break --condition 4 x > 2      #当x>2时,命中第四个断点
break --condition 3            #删除命中第三个断点的条件

break --delete 5               #删除第5个断点
break --disable-all            #禁用所有断点

break                          #显示出所有断点的位置
break --show 2                 #显示第二个断点附近的代码

break --delete-all             #删除所有断点
break --enable 2               #启用第二个断点...为毛没有一次启用所有断点的功能?!
```
退出pry
```ruby
exit!
```
