---
title: 使用BDD方式开发一个gem(未完成...)
category: Ruby
tags:
  - Ruby
  - Gem
date: 2016-08-07 09:31:28
---
你应该遇到过通过区县查省份,通过区县查邮编的时候.一般我们会打开浏览器google一下.但是,我不想,我想敲一行代码就能迅速的查到,谁让咱是程序员呢😊.这就是我这篇记录的产生原因.
[Code地址](http://bing.com)
本文将记录采用行为驱动开发(BDD)的方式,创建一个gem到发布的真实过程.
安装bundler
```shell
gem install bundler
```
创建gem项目(项目名称为 china_district_code)
```shell
bundler gem china_district_code
```
生成的目录结构应该是这样
```ruby
china_district_code
- bin
- lib
.gitignore
china_district_code.gemspec
Gemfile
Rakefile
README.md
```
引入测试gem.打开gemfile文件,输入以下内容后运行命令: bundle install
```ruby
source 'https://gems.ruby-china.org'

gem 'rspec'
gem 'rspec-core'
gem 'guard-rspec' #自动化测试插件
```
生成配置文件
```ruby
rspec --init #初始化rspec
```
执行完成后,目录结构应该像这样子
```ruby
china_district_code
- bin
- lib
- spec
- - spec_helper.rb
.gitignore
.rspec
china_district_code.gemspec
Gemfile
Rakefile
README.md
```
然后生成我们的测试文件
```ruby
touch spec/china_district_code_spec.rb
```
生成自动化测试配置文件,下面的命令执行完成后,文件夹内会多出一个Guardfile文件(自动化测试Guard的配置文件,默认不用修改).
```ruby
guard init rspec
```
然后可以启动我们的自动化测试,监控文件变化.
```ruby
bundle exec guard
```
关于自动化测试插件 guard的用法可以参考
[Guard的Github地址](https://github.com/guard/guard#readme)
[命令说明地址](https://github.com/guard/guard/wiki/Command-line-options-for-Guard)
以上的基础工作完成,下面开始写我们的行为测试.
打开china_district_code_spec.rb,输入以下内容(请无视我的中国味英语o(╯□╰)o)
```ruby
require 'spec_helper'

describe "China district code" do
  context "district" do
    it "get province,city,area info"
    it "find citys by province"
    it "find areas by citys"
    it "find province by city"
    it "find city,province by area"
    it "find area by district code"
  end

  context "post code" do
    it "find post code  by district code"
  end
end
```
保存后,你会发现Guard已经自动运行了测试,并给出提示,所有的测试都是pending状态.
shell中应该像这样:
```markdown
00:13:39 - INFO - Running: spec/china_district_code_spec.rb
*******

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) China district code district get province,city,area info
     # Not yet implemented
     # ./spec/china_district_code_spec.rb:5

  2) China district code district find citys by province
     # Not yet implemented
     # ./spec/china_district_code_spec.rb:6

  3) China district code district find areas by citys
     # Not yet implemented
     # ./spec/china_district_code_spec.rb:7

  4) China district code district find province by city
     # Not yet implemented
     # ./spec/china_district_code_spec.rb:8

  5) China district code district find city,province by area
     # Not yet implemented
     # ./spec/china_district_code_spec.rb:9

  6) China district code district find area by district code
     # Not yet implemented
     # ./spec/china_district_code_spec.rb:10

  7) China district code post code find post code  by district code
     # Not yet implemented
     # ./spec/china_district_code_spec.rb:14


Finished in 0.00123 seconds (files took 0.14289 seconds to load)
7 examples, 0 failures, 7 pending
```
然后,我们来实现我们的行为.

























