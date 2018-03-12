---
layout:     post
title:      "linux笔记: 在ubuntu上安装jekyll"
subtitle:   ""
date:       2018-03-12 03:00:00
author:     "Yuna"
header-img: "img/home-bg.jpg"
tags:
    - linux
    - 安装笔记
---

*最开始搭本地jekyll环境是在win上，一切顺利。然而换到linux上却踩了一堆的坑，只好借此记录下目前成功的安装途径*

**本教程基于ubuntu发行版，采用第三方安装工具安装，非源码编译！**


## 安装ruby-build工具

1. 获取安装包： `$ wget -O ruby-install-0.6.1.tar.gz https://github.com/postmodern/ruby-install/archive/v0.6.1.tar.gz`
2. 解压安装包： `$ tar -xzvf ruby-install-0.6.1.tar.gz`
3. 进入安装目录，执行安装： 
	
	```
	$ cd ruby-install-0.6.1/
	$ sudo make install
	```


## 安装ruby

1. 在系统上安装ruby稳定版（默认安装路径：/usr/local/): `$ sudo ruby-install --system ruby`
2. 检查安装版本号：
	
	```
	$ ruby -v
	$ gem -v
	```


## 安装jekyll

1. 安装jekyll： `$ sudo gem install jekyll`
2. 检查安装版本号： `$ jekyll -v`
2. 运行jekyll（即时显示修改）： `$ jekyll serve --watch`
3. 若运行不成功提示缺少依赖，则补充安装依赖，如： `$ sudo gem install jekyll-paginate`，然后重新尝试运行


## 可能遇到的坑
+ 安装ruby时失败，显示类似如下信息：

	```
	The following packages have unmet dependencies:
	 libssl-dev : Depends: libssl1.0.0 (= 1.0.1-4ubuntu5.7) but 1.0.1c-3ubuntu2.2 is to be installed
	              Recommends: libssl-doc but it is not going to be installed
	```

	此时参照隔壁[解决ubuntu安装libssl-dev失败]({% post_url 2018-03-10-ubuntu-fix-libssl-dev %})

+ 安装时提示无法从安装源获取资源，如：

	```
	W: 无法下载 http://cn.archive.ubuntu.com/ubuntu/dists/raring-updates/multiverse/binary-i386/Packages 404 Not Found [IP: 202.118.1.64 80]

	E: Some index files failed to download. They have been ignored, or old ones used instead.
	```

	此时参照隔壁的隔壁[解决ubuntu源更新的问题]({% post_url 2018-03-10-ubuntu-fix-apt-source %})