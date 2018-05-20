---
title: 配置shadowsocks
date: 2018-05-20 16:56:01
tags: linux
---

#### 1. 安装shadowsocks
```
pip install shadowsocks
```

#### 2. server端设置

```
ssserver -c config.json -d start
```

```
{
    "server":"0.0.0.0",
    "server_port":5333,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"xxxxxx",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": true
}

```

#### 3. client端启动
本地执行：
```
/usr/local/opt/shadowsocks-libev/bin/ss-local -c shadowsocks.json &
```

```
{
  "server":"xx.xx.xxx.xx",
  "server_port":5333,
  "local_port":1080,
  "password":"xxxxxx",
  "timeout":300,
  "method":"aes-256-cfb",
  "local_address":"127.0.0.1",
  "fast_open":false,
  "tunnel_remote":"8.8.8.8",
  "dns_server":["8.8.8.8", "8.8.4.4"],
  "tunnel_remote_port":53,
  "tunnel_port":53
}
```