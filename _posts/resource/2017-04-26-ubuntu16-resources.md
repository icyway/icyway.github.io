---
layout: post
title: Ubuntu阿里源
category: 资源
tags: Ubuntu
keywords: Ubuntu
description:
---

## Ubuntu版本名称

一开始的时候，我就想只记录一下Ubuntu的阿里源。提笔之时，突然发现有必要做一些linux版本
常识的说明。

**linux**一般有两种版本，一个是核心（kernel)版，一个是发行(distribution)版。核心版的序号由三部分数字构成，其形式为：

major.minor.patchlevel

其中，majoro为主版本号，minor为次版本号，二者共同构成了当前核心版本号。patchlevel表示对当前版本的修订次数。例如，2.2.11表示对核心作用2.2 版本的第11次修订。根据约定，次版本号为奇数时，表示该版本加入新内容，但不一定稳定，相当于测试版；次版本号为偶数时，表示这是一个可以使用的稳定版本。

**linux的内核**具有两种不同的版本号，实验版本（开发版）和产品化版本（稳定版）。要确定linux版本的类型，只要查看一下版本号：每一个版本号由三位数字组成。第一位数是发布的内核主版本。第二位偶数表示稳固版本；奇数表示开发中版本。第三位是错误修补的次数。

Ubuntu每一年会发行一个主版本，每两年会发行一LTS（Long Term Support）版本。个人而言，比较喜欢用主版本是偶数的版本，如：12.04/14.04/16.04。

有意思的是ubuntu每个版本都有它的开发代号，开发代号有三个热点：
- 都是动物
- 都是两个词，并且两个词的首字母相同
- 从6.06开始，首字母从D开始递增

如：
ubuntu16.04，它的代号是Xenial Xerus（好客的非洲地松鼠）

鉴于国内的资源访问，我感觉阿里源的速度还是不错的，

地址：
```
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
##测试版源
deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
# 源码
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
##测试版源
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
# Canonical 合作伙伴和附加
deb http://archive.canonical.com/ubuntu/ xenial partner
deb http://extras.ubuntu.com/ubuntu/ xenial main
```

其实，我爱debian更多一些。

嘿嘿。。。
