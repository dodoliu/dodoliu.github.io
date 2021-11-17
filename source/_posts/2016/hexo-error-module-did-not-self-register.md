---
title: Hexo Error:Module did not self-register
permalink: hexo-error-module-did-not-self-register
category: Hexo
tags:
  - Hexo
date: 2016-08-11 08:28:05
---
在启动hexo服务时遇到该问题.
```markdown
hexo s
[Error: Module did not self-register.]
{ [Error: Cannot find module './build/default/DTraceProviderBindings'] code:'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/Debug/DTraceProviderBindings'] code:'MODULE_NOT_FOUND' }
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
分析原因,应该是昨天安装了iTerm后使用了zsh,然后好多命令都找不到了...然后各种幺蛾子问题都出来了!!!
这是因为mac默认使用的是bash,现在设置成zsh后,好多命令都不存在了...最简单的方式是就重装一次!!!!
我再尝试 升级 nvm
```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.4/install.sh | bash
```
然后重装 node和hexo后解决了该问题
```bash
nvm install node
npm install hexo --no-optional
```
