---
title: 多远程服务的.ssh配置
date: 2016-06-30 13:24:02
tags:
    - linux
    - mac
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

## 使用例子
第一个host的远程地址对应
```
git clone git@github.com:keeleys/flask_funny.git
```

第二个host的远程地址对应
```
git clone git@sitec.github.com:keeleys/flask_funny.git
```
