---
title: 如何使用hexo
date: 2018-04-14 18:38:41
tags: utils
---

今天开始使用hexo+github，构建一个简单的blog，做一些技术内容的记录。担心会时不时忘记hexo怎么用，那么就先来记录一下hexo的用法吧。
首先，安装nodejs
```
    sudo apt install nodejs
```
获取了代码库（git@github.com:DatongLi/hexo_blog.git）之后
```
    sudo npm install hexo --save
    sudo npm install
    
    sudo npm install hexo-renderer-pug --save
    sudo npm install hexo-renderer-sass --save
    (
        npm install -g cnpm --registry=https://registry.npm.taobao.org
        sudo cnpm install hexo-renderer-sass --save
    )
```
按理可以正常使用了，后续指令是用hexo的
```
    hexo g //generate ,编译成静态文件
    hexo d //deploy, 部署网站
    hexo s //server, 本地运行
    hexo c //clean, 清空generate生成器的文件
```
新建文件：
```
    hexo new post "newPost"
```
