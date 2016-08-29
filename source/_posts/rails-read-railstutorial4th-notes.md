---
title: 看《Ruby on Rails 教程（第四版）》杂记 (持续更新...)
category: Ruby On Rails
tags:
  - Ruby On Rails
  - Ruby
date: 2016-08-29 22:07:13
---
以下内容均为《Ruby on Rails 教程（第四版）》学习摘要!

* ***编写测试的指导方针***
与应用代码相比,如果测试代码特别简短,倾向于先编写测试;
如果对想实现的功能不是特别清楚,倾向于先编写应用代码,然后再编写测试,并改进实现方式;
安全是头等大事,保险起见,要为安全相关的功能线编写测试;
只要发现一个问题,就编写一个测试重现这种问题,避免回归,然后再编写应用代码修正问题;
尽量不为以后可能修改的代码(例如HTML结构的细节)编写测试;
重构之前要编写测试,集中测试容易出错的代码.
在实际的开发中,根据上述方针,我们一般先编写控制器和模型测试,然后再编写集成测试(测试模型、试视图和控制器在一起时的表现)。如果应用代码很容易出错，或者经常变动（视图就是这样），我们就完全不测试。
* ***assert_select***
test中的辅助函数,用于检测html中是否有指定的html标签,这个方法有时也叫"选择符"
assert_select 'title','Home I Ruby on Rails Tutorial Sample App' #含义: 检测有没有< title \>标签,以及其中的内容是不是 "Home I Ruby on Rails Tutorial Sample App"




