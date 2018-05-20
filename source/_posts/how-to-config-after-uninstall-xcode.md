---
title: macOS卸载掉Xcode后如何设置
date: 2018-05-20 15:16:52
tags: linux
---
Xcode大的时候有10G，占用很多空间，如果不用开发MacOS或者iOS，完全可以卸载掉。然后输入指令：
```
sudo xcode-select --install
```
可以完成安装常用环境。而往往运行命令的时候会出现报错

```
~ g++ --version
xcrun: error: active developer path ("/Applications/Xcode.app/Contents/Developer") does not exist
Use `sudo xcode-select --switch path/to/Xcode.app` to specify the Xcode that you wish to use for command line developer tools, or use `xcode-select --install` to install the standalone command line developer tools.
See `man xcode-select` for more details.
```
是因为环境变量没设置好，可以用`xcode-select -p`查看，修改路径为：

```
sudo xcode-select --switch /Library/Developer/CommandLineTools
```
即可。