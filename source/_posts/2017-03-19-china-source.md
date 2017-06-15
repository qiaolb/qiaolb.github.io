---
layout:     post
title:      "常用国内源汇总"
subtitle:   "在开发和实施中，经常需要使用各种源"
date:       2017-03-19 00:00:00
author:     "Joe"
header-img: "img/post-bg-2015.jpg"
tags:
    - NPM
    - Linux
---

在开发和实施中，经常需要调整各种源，以到达最快的速度，今天对我常用的源做一个汇总。

<!-- more -->

### Node.js

可以直接从[阿里源](http://npm.taobao.org/mirrors/node)下载

### NPM

NPM阿里的源： https://npm.taobao.org/
可以按照网页中的描述按照`cnpm`，但我更喜欢直接使用`npm`，只是将源修改为阿里的方法如下：

```
npm config set registry https://registry.npm.taobao.org
```

### phantomjs

`PhantomJS` 是一个基于 `WebKit` 的服务器端 JavaScript API。在npm安装时经常会用到，但国外的源下载很慢，修改为国内的源的方法如下：
增加环境变量：

```
PHANTOMJS_CDNURL=http://npm.taobao.org/mirrors/phantomjs
```

### CentOS

163源，使用方法参考： http://mirrors.163.com/.help/centos.html

下载对应版本repo文件, 放入/etc/yum.repos.d/(操作前请做好相应备份)

[CentOS7](http://mirrors.163.com/.help/CentOS7-Base-163.repo)
[CentOS6](http://mirrors.163.com/.help/CentOS6-Base-163.repo)
[CentOS5](http://mirrors.163.com/.help/CentOS5-Base-163.repo)

运行以下命令生成缓存

```
yum clean all
yum makecache
```

另外，[163源](http://mirrors.163.com/)还提供了`debian`，`Ubuntu`等源镜像。

### 常用js库功能CDN

[bootcdn](http://www.bootcdn.cn)，相对比较全
[七牛云存储](https://www.staticfile.org/)
[CNDJS](http://www.cdnjs.cn) 从官方CNDJS镜像：http://www.cdnjs.com/

