---
title: Rails Error:libmysqlclient loaderror
category: Ruby On Rails
tags:
  - Ruby On Rails
date: 2016-07-09 17:53:19
---
环境:
osx 10.11.5

ruby '2.2.0'
gem 'rails', '4.2.3'
gem 'mysql2', '~> 0.3.18'

今天在折腾rails时，遇到mysql2与rails版本匹配问题, 虽然mysql2已经指定了 '~> 0.3.18' 版本。报错信息：
```ruby
LiudongyuematoMacBook-Air:r_admin_demo liudongyue$ rails s
/Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/mysql2-0.3.20/lib/mysql2.rb:31:in 'require': dlopen(/Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/mysql2-0.3.20/lib/mysql2/mysql2.bundle, 9): Library not loaded: /usr/local/lib/libmysqlclient.18.dylib (LoadError)
  Referenced from: /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/mysql2-0.3.20/lib/mysql2/mysql2.bundle
  Reason: image not found - /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/mysql2-0.3.20/lib/mysql2/mysql2.bundle
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/mysql2-0.3.20/lib/mysql2.rb:31:in '<top (required)>'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.2/lib/bundler/runtime.rb:76:in 'require'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.2/lib/bundler/runtime.rb:76:in 'block (2 levels) in require'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.2/lib/bundler/runtime.rb:72:in 'each'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.2/lib/bundler/runtime.rb:72:in 'block in require'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.2/lib/bundler/runtime.rb:61:in 'each'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.2/lib/bundler/runtime.rb:61:in 'require'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.2/lib/bundler.rb:134:in 'require'
  from /Users/liudongyue/LDY/Test/r_admin_demo/config/application.rb:16:in '<top (required)>'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/railties-4.2.3/lib/rails/commands/commands_tasks.rb:78:in 'require'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/railties-4.2.3/lib/rails/commands/commands_tasks.rb:78:in 'block in server'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/railties-4.2.3/lib/rails/commands/commands_tasks.rb:75:in 'tap'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/railties-4.2.3/lib/rails/commands/commands_tasks.rb:75:in 'server'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/railties-4.2.3/lib/rails/commands/commands_tasks.rb:39:in 'run_command!'
  from /Users/liudongyue/.rvm/gems/ruby-2.2.0/gems/railties-4.2.3/lib/rails/commands.rb:17:in '<top (required)>'
  from bin/rails:4:in 'require'
  from bin/rails:4:in '<main>'

```
可能的解决方法：
重装mysql,重装mysql2
```bash
brew uninstall mysql
brew install mysql
gem uninstall mysql2
gem install mysql
```
