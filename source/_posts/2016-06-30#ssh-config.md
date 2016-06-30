---
title: 多远程服务的.ssh配置
date: 2016-06-30 13:24:02
tags:
    -linux
    -mac
---

## ~/.ssh/config
```
Host 192.168.1.198
  Port 10022

Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa

Host sitec.github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/sitec
```
