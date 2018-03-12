---
layout:     post
title:      "linux笔记: 解决ubuntu源更新的问题"
subtitle:   ""
date:       2018-03-12 01:00:00
author:     "Yuna"
header-img: "img/home-bg.jpg"
tags:
    - linux
    - 安装笔记
---


### 问题描述

在ubuntu进行apt下载及更新操作时，总是提示获取资源失败，如：

```
W: 无法下载 http://cn.archive.ubuntu.com/ubuntu/dists/raring-updates/multiverse/binary-i386/Packages 404 Not Found [IP: 202.118.1.64 80]

E: Some index files failed to download. They have been ignored, or old ones used instead.
```


### 问题分析

使用的ubuntu系统版本已经停止更新，因此已经从源中移除，故找不到文件


### 解决方案

+ 升级系统或换用LTS版本
+ 修改apt的源： `$ sudo vim /etc/apt/sources.list`

	将其中的`http://cn.archive.ubuntu.com/ubuntu/`全部改为`http://old-releases.ubuntu.com/ubuntu/`