---
title: 从零开始用docker搭建ubuntu
date: 2018-07-23 22:11:17
tags: linux
---

### 先找好源

网易云的镜像源很不错，`https://c.163yun.com/hub#/m/repository/?repoId=3186`，网易做开源真的是业界良心。

### 写好Dockerfile
直接用网易镜像源上面提供的，更新为网易的软件源，还有工具支持ssh，足够用了。
```
FROM hub.c.163.com/public/ubuntu:16.04
MAINTAINER netease
RUN mkdir -p /var/run/sshd
RUN mkdir /root/.ssh/
RUN apt-get update \
    && apt-get update && apt-get install -y openssh-server vim tar wget curl rsync bzip2 iptables tcpdump less telnet net-tools lsof sysstat cron supervisor inetutils-ping ufw \
    && rm -rf /var/lib/apt/lists/*
RUN ufw allow ssh
RUN echo 'root:qwer1234' | chpasswd
RUN ssh-keygen -q -t rsa -b 2048 -f ~/.ssh/id_rsa -P '' -N ''
RUN sed -i s/"PermitRootLogin prohibit-password"/"PermitRootLogin yes"/g /etc/ssh/sshd_config
RUN cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
EXPOSE 22
COPY sshd.conf /etc/supervisor/conf.d/sshd.conf
CMD ["/usr/bin/supervisord"]
```
其中`sshd.conf`文件是：
```
[supervisord]
nodaemon=true
[program:sshd]
command=/usr/sbin/sshd -D
```

### macOS下准备环境
macOS准备环境还挺麻烦
```
brew install docker-machine docker
brew cask install virtualbox
docker-machine create --driver virtualbox default
eval "$(docker-machine env default)"
# 如果出问题：
# docker-machine --debug env default
# docker-machine start
# docker-machine ls

# 如果是linux的话
# service docker restart
# 或者：/etc/init.d/docker restart
# 或者： systemctl restart docker
# 可能要root权限
```


### 执行build拉取镜像
拉取镜像
```
docker pull hub.c.163.com/public/ubuntu:16.04-tools
```

开始build了
```
docker build -t ubuntu:starheap -f `pwd`/Dockerfile ubuntu
```

### run
使用上面拉取的镜像，启动一个容器
```
# 使用ubuntu:starheap镜像以后台模式启动一个名为backend的容器
docker run --name backend -d ubuntu:starheap

# 使用镜像ubuntu:starheap以后台模式启动一个容器,并将容器的80端口映射到主机随机端口
# -P :是容器内部端口随机映射到主机的高端口
docker run -P -d ubuntu:starheap

# 使用镜像ubuntu:starheap以后台模式启动一个容器,将容器的80端口映射到主机的80端口,主机的目录/data映射到容器的/data
# -p : 是容器内部端口绑定到指定的主机端口。
docker run -p 80:80 -v /data:/data -d ubuntu:starheap

# 可以指定ip 地址，默认绑定的是tcp，还可以绑定udp
docker run -p 127.0.0.1:80:80 -v /data:/data -d ubuntu:starheap


# 使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令
docker run -it ubuntu:starheap /bin/bash
```

注意，macOS可能每次使用前都要执行：
```
eval "$(docker-machine env default)"
```

### docker容器的start/stop/restart
```
# 启动已被停止的容器backend
docker start backend

# 停止运行中的容器backend
docker stop backend

# 重启容器backend
docker restart backend

#  docker rm 命令来删除不需要的容器
docker rm backend
```

### 如何ssh上容器
因为用的网易docker镜像，已经安装好了sshd，可以直接在启动容器的时候指定绑定的端口，就可以通过服务器的端口ssh到容器内部的22端口了，具体操作如下：
```
# 将新的镜像启动，并将docker服务器的50001端口映射到容器的22端口上
docker run -d -p 50001:22 ubuntu:starheap /usr/sbin/sshd -D

# 执行ssh就行了
$ ssh root@localhost -p 50001
```


### 使用docker进入容器
ssh连接到docker容器太难配置了，可以用docker直接进入容器。
```
~/Desktop/workspace/docker-ubuntu docker images -a
hub.c.163.com/public/ubuntu   16.04-tools         1196ea15dad6        14 months ago       336MB
hub.c.163.com/public/ubuntu   16.04               70b70c987e8f        2 years ago         224MB
```

```
docker run --name backend2 -itd  -p 50002:22  1196ea15dad6
```

```
~/Desktop/workspace/docker-ubuntu docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                          PORTS                   NAMES
a902de9e2392        1196ea15dad6        "/usr/bin/supervisord"   6 seconds ago        Up 5 seconds                    0.0.0.0:50002->22/tcp   backend2
```


```
docker exec -it a902de9e2392 /bin/bash
```
