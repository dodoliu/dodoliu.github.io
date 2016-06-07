---
title: Hexo搭建静态Blog后的源码管理
category: Hexo
tags:
  - Hexo
  - Git
date: 2016-06-05 14:59:18
---
当我们使用Hexo方便快捷的搭建完静态Blog,并愉快的写完第一篇Blog发布后.可能会遇到这样的问题:
我的Blog源码怎么管理啊?!
官方的命令咋没有把源码一并提交到GitHub啊?!
我要是有多台电脑,还要再安装一遍不成?!
同志,不要怕,接下来请跟我一步一步做.

#### 下面拿我的 dodoliu blog为例,以下所有的命令操作都假设你处在blog文件夹的根目录下,且你的系统是OSX
* 先删除 dodoliu blog文件夹下的 .git 文件夹,该文件夹在windows下默认是隐藏的
```bash
cd dodoliu
rm -rf .git
```
* 再删除 themes 主题文件夹下的 .git 文件夹,如果发现有 .github 文件夹也一并删掉.
```bash
cd themes/next
rm -rf .git .github
```
* 然后回到dodoliu blog的根目录,创建git仓库
```bash
cd dodoliu
git init
```
* 然后添加文件到git仓库,在添加前确认你blog根目录下的.gitignore文件内容像我这样
```bash
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```
* 接下来把本地代码提交到本地git仓库
```bash
git add . && git commit -m 'init'
```
* 然后copy 线上 github的仓库地址,添加到本地仓库(https://github.com/dodoliu/dodoliu.github.io.git是我的github仓库地址,请替换成你自己的)
```bash
git remote add origin https://github.com/dodoliu/dodoliu.github.io.git
```
* 然后创建一个本地分支,并切换到该分支下
```bash
git branch blog
git checkout blog
```
* 接下来,提交代码到github线上的blog分支中
```bash
git push origin blog
```
* 提交到github时,如果不想每次都打这么多命令,可以进行这样的设置
```bash
git push --set-upstream origin blog
#之后就用下面这个命令推送吧
git push
```
然后到github上看看的master和blog分支的区别吧!
以后再次提交请在blog分支下进行操作.
到这里,恭喜你!
大功告成,开始你的下一篇blog吧!