---
title: zookeeper cli 常用命令
date: 2018-04-19 23:33:52
tags: utils
---

zk经常使用，但是也经常忘记这些指令，稍微记录一下。

```
// Create Znodes
create /path /data
e.g. create /FirstZnode “Myfirstzookeeper-app”
```

```
// Get Data : It returns the associated data of the znode and metadata of the specified znode
get /path 
e.g. get /FirstZnode
```

```
// Set Data : Set the data of the specified znode
set /path /data
e.g. get /SecondZnode "Data-updated"
```

```
// Watch : Watches show a notification when the specified znode or znode’s children data changes
get /path [watch] 1
e.g. get /FirstZnode 1
```

```
// List Children
ls /path
e.g. ls /MyFirstZnode
```