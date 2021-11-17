---
title: Rails下sidekiq无法异步发送邮件的问题
permalink: rails-tip-sidekiq-async-sendmail
category: Rails
tags:
  - Rails
  - sidekiq
date: 2017-04-02 21:48:18
---
我在研究sidekiq异步发送邮件的过程中,遇到了sidekiq无法执行队列中的任务的情况.
明明sidekiq的"已进入队列"已经包含了发送邮件的任务,并且我非常确定发送邮件的程序是没有问题的.
但是无论怎么尝试,邮件都发送不出去.
rails console中看到的入队信息是这样的
```ruby
2.3.0 :015 > OrderMailer.confirm_email(1).deliver_later
Enqueued ActionMailer::DeliveryJob (Job ID: a7cda76b-1f01-4f4a-a695-3be1c6bb2640) to Sidekiq(development_mailers) with arguments: "OrderMailer", "confirm_email", "deliver_now", 1
```
这里很明确的指出队列的名称是 development_mailers
在我没有查明原因之前我的sidekiq.yml配置项是这样的
```ruby
:concurrency: 5
:pidfile: tmp/pids/sidekiq.pid

:queues:
    - default
    - [myqueue, 2]

development:
  :concurrency: 5
staging:
  :concurrency: 10
production:
  :concurrency: 20
```
因为对sidekiq研究不透,当时并没有注意 :queues 这块的含义.
在反复google后终于在[这篇教程](https://gist.github.com/maxivak/690e6c353f65a86a4af9)里找到了问题的原因所在.
其中有这样一句话,让我恍然大悟:
"!!! important. You may need to include new queue names in sidekiq.yml file:"
我了个大艹!
原来是我没有把 development_mailers 队列加到可执行队列中去.
添加后的sidekiq.yml配置项是这样的
```ruby
:concurrency: 5
:pidfile: tmp/pids/sidekiq.pid

:queues:
    - default
    - [myqueue, 2]
    - development_mailers

development:
  :concurrency: 5
staging:
  :concurrency: 10
production:
  :concurrency: 20
```
重启sidekiq后,世界美好了!
ps:
```ruby
:queues:
    - default
    - [myqueue, 2]
    - development_mailers
```
这块的含义是sidekiq可以执行的队列集合,其中 [myqueue, 2] 这里的2 表示被分配到的权重是2.
可参考[这里](http://wdxtub.com/2016/07/06/sidekiq-guide/)



