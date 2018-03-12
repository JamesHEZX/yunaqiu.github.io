---
layout:     post
title:      "linux笔记: 解决ubuntu安装libssl-dev失败"
subtitle:   "解决Ubuntu系统libssl-dev依赖不匹配问题"
date:       2018-03-12 02:00:00
author:     "Yuna"
header-img: "img/home-bg.jpg"
tags:
    - linux
    - 安装笔记
---


### 问题描述

在ubuntu进行apt操作时，提示libssl依赖不匹配，如：

```
The following packages have unmet dependencies:
 libssl-dev : Depends: libssl1.0.0 (= 1.0.1-4ubuntu5.7) but 1.0.1c-3ubuntu2.2 is to be installed
              Recommends: libssl-doc but it is not going to be installed
```


### 问题分析

libssl安装的版本与系统不兼容, 此时继续用apt重新安装libssl-dev是不行的，通常需要采取降级处理。


### 解决方案

1. 安装aptitude工具： `$ sudo apt-get install aptitude`
2. 用aptitude安装libssl-dev： `$ sudo aptitude install libssl-dev`

	```
	The following NEW packages will be installed:
	  libssl-dev{b} libssl-doc{a} 
	0 packages upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
	Need to get 2,031 kB of archives. After unpacking 7,801 kB will be used.
	The following packages have unmet dependencies:
	 libssl-dev : Depends: libssl1.0.0 (= 1.0.1f-1ubuntu2) but 1.0.1f-1ubuntu2.19 is installed.
	The following actions will resolve these dependencies:

	     Keep the following packages at their current version:
	1)     libssl-dev [Not Installed]
	```

3. 此时询问是否维持原版本，选否： `Accept this solution? [Y/n/q/?] n`

	```
	The following actions will resolve these dependencies:

	     Downgrade the following packages:                                   
	1)     libssl1.0.0 [1.0.1f-1ubuntu2.19 (now) -> 1.0.1f-1ubuntu2 (trusty)]
	```

4. 询问是否接受该降级方案，选是： `Accept this solution? [Y/n/q/?] y`
	
	```
	The following packages will be DOWNGRADED:
	  libssl1.0.0 
	The following NEW packages will be installed:
	  libssl-dev libssl-doc{a} 
	0 packages upgraded, 2 newly installed, 1 downgraded, 0 to remove and 0 not upgraded.
	Need to get 2,857 kB of archives. After unpacking 7,784 kB will be used.
	```

5. 询问是否继续，选是： `Do you want to continue? [Y/n/?] y`
6. 一路到底，完成


### 可能遇到的问题
+ 安装时提示无法从安装源获取资源，如：

	```
	W: 无法下载 http://cn.archive.ubuntu.com/ubuntu/dists/raring-updates/multiverse/binary-i386/Packages 404 Not Found [IP: 202.118.1.64 80]

	E: Some index files failed to download. They have been ignored, or old ones used instead.
	```

	此时参照隔壁[解决ubuntu源更新的问题]({% post_url 2018-03-10-ubuntu-fix-apt-source %})