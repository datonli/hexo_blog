---
title: 增加swap分区
date: 2018-05-20 20:17:05
tags: linux
---

在执行free -m的是时候提示Cannot allocate memory（swap文件可以放在自己喜欢的位置如/var/swap）:
```
mkdir /opt/images/
rm -rf /opt/images/swap 
dd if=/dev/zero of=/opt/images/swap bs=1024 count=128000 
mkswap /opt/images/swap
swapon /opt/images/swap
free -m
```

使用完毕后可以关掉swap:
```
swapoff swap  
rm -f /opt/images/swap 
```