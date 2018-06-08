---
title: ubuntu16.04配置samba
date: 2018-06-08 21:09:24
tags: linux
---
#### 1. samba安装
`sudo apt-get install samba samba-common smbclient`

#### 2. 查看samba服务
samba安装成功后，会默认启动samba服务。
`ps -ef | grep smb`

#### 3. 修改samba配置文件/etc/samba/smb.conf
```
#======================= Share Definitions =======================
 
# Un-comment the following and create the netlogon directory for Domain Logons
# Un-comment the following (and tweak the other settings below to suit)
# to enable the default home directory shares. This will share each
# user's home directory as \\server\username
[homes]
  comment = Home Directories
  browseable = no
 
# By default, the home directories are exported read-only. Change the
# next parameter to 'no' if you want to be able to write to them.
  read only = no
 
# File creation mask is set to 0700 for security reasons. If you want to
# create files with group=rw permissions, set next parameter to 0775.
  create mask = 0775
 
# Directory creation mask is set to 0700 for security reasons. If you want to
# create dirs. with group=rw permissions, set next parameter to 0775.
  directory mask = 0775
```
#### 4. 重启samba服务
`/etc/init.d/samba restart`
#### 5. 调试samba功能
##### 5.1 创建linux登录账号（例如test_user/test_password）
`sudo adduser test_user`
##### 5.2 给上述test_user用户创建samba密码
`sudo smbpasswd -a test_user`

如果因权限不足问题，不能创建、修改该目录下的文件，请检查`3.修改samba配置文件/etc/samba/smb.conf`是否配置正确。samba详细配置见/etc/samba/smb.conf 文件中的注释或用命令man smb.conf查看。