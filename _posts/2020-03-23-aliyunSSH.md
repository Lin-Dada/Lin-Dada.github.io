---
layout:     post
title:      "阿里云服务器ssh经常一段时间就断掉解决办法"
subtitle:   "解决阿里云服务器过一段时间自动断连"
date:       2020-03-23 21:15:00
author:     "Lindada"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - 服务器
    - 阿里云
---

`#vim /etc/ssh/sshd_config`

找到下面两行
```
#ClientAliveInterval 0
#ClientAliveCountMax 3
```
去掉注释，改成
```
ClientAliveInterval 30
ClientAliveCountMax 86400
```
这两行的意思分别是
1. 客户端每隔多少秒向服务发送一个心跳数据
2. 客户端多少秒没有相应，服务器自动断掉连接
重启sshd服务
`#service sshd restart`





