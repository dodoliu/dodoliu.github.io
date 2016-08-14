---
title: 使用BDD方式开发一个gem
category: Ruby
tags:
  - Ruby
  - Gem
date: 2016-08-07 09:31:28
---
你应该遇到过通过区县查省份,通过区县查邮编的时候.一般我们会打开浏览器google一下.但是,我不想,我想敲一行代码就能迅速的查到,谁让咱是程序员呢😊.这就是我这篇记录的产生原因.
[源码地址](https://github.com/dodoliu/china_district_code)
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
    it "find areas by city"
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

  3) China district code district find areas by city
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
然后,我们来实现我们的行为,忽略折腾的过程....直接上完成的code.
```ruby
china_district_code_spec.rb

require 'spec_helper'
require 'china_district_code'
require 'csv'

describe "China district code" do
  context "district" do
    it "get province,city,area info" do
      ChinaDistrictCode::load_china_district_code
      csv_folder_path = File.join(File.expand_path('../..',__FILE__),'lib','china_district_code','csv')
      provinces = CSV.read(File.join(csv_folder_path,'province.csv'))
      expect(provinces[0][0]).to eq('11')
    end
    it "find citys by province" do
      expect(ChinaDistrictCode::find_citys_by_province('河北省')).to include(["130100", "石家庄市"])
    end
    it "find areas by city" do
      expect(ChinaDistrictCode::find_areas_by_city('石家庄市')).to include(["130103", "桥东区"])
    end
    it "find province by city" do
      expect(ChinaDistrictCode::find_province_by_city('太原')).to eql(["140000", "山西省"])
    end
    it "find city,province by area" do
      expect(ChinaDistrictCode::find_province_city_by_area('桥东区')).to eql([["130000", "河北省"], ["130100", "石家庄市"]])
    end
    # it "find area by district code"
  end

  context "post code" do
    it "find post code  by district code"
  end
end
```
然后,一个个实现我们的需求,享受由红变绿的过程(囧)
```ruby
#encoding: utf-8
require 'logger'

require "china_district_code/version"
require 'china_district_code/china_district_code/loader'

LOGGER = Logger.new(File.join(File.dirname(__FILE__),'/china_district_code/log/debug.log'))

module ChinaDistrictCode

  load_file 'models'
  load_file 'helpers'

  include CsvHelper
  include DistrictHelper

  #reload china district code from 'http://www.stats.gov.cn/tjsj/tjbz/xzqhdm/201401/t20140116_501070.html'
  def self.load_china_district_code
    DistrictHelper::get_china_district_code
  end

  #find citys by province  
  def self.find_citys_by_province(name)
    result = DistrictHelper::find_citys_by_province(name)
    puts result.to_s
    result
  end

  #find areas by city
  def self.find_areas_by_city(name)
    result = DistrictHelper::find_areas_by_city(name)
    puts result.to_s
    result
  end

  #find province by city
  def self.find_province_by_city(name)
    result = DistrictHelper::find_province_by_city(name)
    puts result.to_s
    result
  end

  #find province,city by area
  def self.find_province_city_by_area(name)
    result = DistrictHelper::find_province_city_by_area(name)
    puts result.to_s
    result
  end
end
```
完成了gem的开发,接下来需要编译并发布.我们先修改一下gem的配置项.
打开 'china_district_code.gemspec',修改如下
```ruby
# coding: utf-8
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'china_district_code/version'

Gem::Specification.new do |spec|
  spec.name          = "china_district_code"
  spec.version       = ChinaDistrictCode::VERSION
  spec.authors       = ["dodoliu"]
  spec.email         = ["donliu29@gmail.com"]

  spec.summary       = %q{ China District Code Query plugin }
  spec.description   = %q{ China District Code Query plugin }
  spec.homepage      = "https://github.com/dodoliu/china_district_code"

  spec.files         = `git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
  spec.bindir        = "exe"
  spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]

  spec.add_development_dependency "bundler", "~> 1.12"
  spec.add_development_dependency "rake", "~> 10.0"
end

```
在发布之前,我们先安装到本地试用一下,(可以用 rake -T 查看rake任务)
```bash
修改版本号为 VERSION = "0.0.1"
rake build #编译gem
rake install #安装到本地

 -> ⮀ china_district_code ⮀ ⭠ master± ⮀ rake build
china_district_code 0.0.1 built to pkg/china_district_code-0.0.1.gem.
 -> ⮀ china_district_code ⮀ ⭠ master± ⮀ rake install
china_district_code 0.0.1 built to pkg/china_district_code-0.0.1.gem.
china_district_code (0.0.1) installed.
#然后irb调用一下
2.3.1 :003 > require 'china_district_code'
 => true
2.3.1 :004 > ChinaDistrictCode.find_citys_by_province '湖南'
[["430100", "长沙市"], ["430200", "株洲市"], ["430300", "湘潭市"], ["430400", "衡阳市"], ["430500", "邵阳市"], ["430600", "岳阳市"], ["430700", "常德市"], ["430800", "张家界市"], ["430900", "益阳市"], ["431000", "郴州市"], ["431100", "永州市"], ["431200", "怀化市"], ["431300", "娄底市"], ["433100", "湘西土家族苗族自治州"]]
 => [["430100", "长沙市"], ["430200", "株洲市"], ["430300", "湘潭市"], ["430400", "衡阳市"], ["430500", "邵阳市"], ["430600", "岳阳市"], ["430700", "常德市"], ["430800", "张家界市"], ["430900", "益阳市"], ["431000", "郴州市"], ["431100", "永州市"], ["431200", "怀化市"], ["431300", "娄底市"], ["433100", "湘西土家族苗族自治州"]]
2.3.1 :005 >
```
完成以上步骤后,可以确认我们的gem是没有问题的.下面打包发布到https://rubygems.org/
```ruby
rake release
```
你有可能会遇到这样的错误提示!(可以通过将代码提交到本地git仓库解决.(git add . && git commit -m 'first commit'))
```marddown
 -> ⮀ china_district_code ⮀ ⭠ master± ⮀ rake release
china_district_code 0.0.1 built to pkg/china_district_code-0.0.1.gem.
rake aborted!
There are files that need to be committed first.
/Users/dodoliu/.rvm/gems/ruby-2.3.1/gems/bundler-1.12.5/lib/bundler/gem_helper.rb:131:in `guard_clean'
/Users/dodoliu/.rvm/gems/ruby-2.3.1/gems/bundler-1.12.5/lib/bundler/gem_helper.rb:60:in `block in install'
Tasks: TOP => release => release:guard_clean
(See full trace by running task with --trace)
```
然后再次运行 rake release时,可能还会遇到下面这个错误(这是要求你的代码需要提交到远程仓库.我选择提交到github)
```markdown
 ✘ ⮀ -> ⮀ china_district_code ⮀ ⭠ master± ⮀ rake release
china_district_code 0.0.1 built to pkg/china_district_code-0.0.1.gem.
Tagged v0.0.1.
Untagging v0.0.1 due to error.
rake aborted!
Couldn't git push. `git push ' failed with the following output:

fatal: No configured push destination.
Either specify the URL from the command-line or configure a remote repository using

    git remote add <name> <url>

and then push using the remote name

    git push <name>


/Users/dodoliu/.rvm/gems/ruby-2.3.1/gems/bundler-1.12.5/lib/bundler/gem_helper.rb:121:in `perform_git_push'
/Users/dodoliu/.rvm/gems/ruby-2.3.1/gems/bundler-1.12.5/lib/bundler/gem_helper.rb:113:in `git_push'
/Users/dodoliu/.rvm/gems/ruby-2.3.1/gems/bundler-1.12.5/lib/bundler/gem_helper.rb:64:in `block (2 levels) in install'
/Users/dodoliu/.rvm/gems/ruby-2.3.1/gems/bundler-1.12.5/lib/bundler/gem_helper.rb:145:in `tag_version'
/Users/dodoliu/.rvm/gems/ruby-2.3.1/gems/bundler-1.12.5/lib/bundler/gem_helper.rb:64:in `block in install'
Tasks: TOP => release => release:source_control_push
(See full trace by running task with --trace)
```
提交完成后,再次执行 rake release,看到以下内容则表示成功了.啦啦啦
```base
 -> ⮀ china_district_code ⮀ ⭠ master ⮀ rake release
china_district_code 0.0.1 built to pkg/china_district_code-0.0.1.gem.
Tagged v0.0.1.
Pushed git commits and tags.
Pushed china_district_code 0.0.1 to rubygems.org.
```
然后自己装一下,让下载次数加1...😄


















