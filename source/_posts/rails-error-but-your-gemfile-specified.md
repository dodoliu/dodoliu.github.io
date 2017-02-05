---
title: Your Ruby version is *, but your Gemfile specified *
category: Ruby On Rails
tags:
  - Ruby On Rails
date: 2017-02-05 12:08:59
---

错误描述:
```bash
Your Ruby version is 2.0.0, but your Gemfile specified 2.3.0
```
排错过程:

查看ruby版本
```bash
✘ ⮀ -> ⮀ openapi ⮀ ruby-2.3.0 ⮀ ⭠ master± ⮀ ruby -v
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-darwin15]
```
查看当前使用的ruby版本
```bash
 -> ⮀ openapi ⮀ ruby-2.3.0 ⮀ ⭠ master± ⮀ rvm list

rvm rubies

   ruby-2.0.0-p247 [ x86_64 ]
   ruby-2.2.0 [ x86_64 ]
   ruby-2.2.1 [ x86_64 ]
=> ruby-2.3.0 [ x86_64 ]
 * ruby-2.3.1 [ x86_64 ]

# => - current
# =* - current && default
#  * - default
```
错误解决:
重新设置当前使用的ruby版本,问题解决!
```bash
rvm use 2.3.0
```
问题原因:
不明白!为何rvm已经设置了当前使用的ruby版本是2.3.0,但是还要重新设置一下!
