---
layout: post
title:  "windows编程之vs陷阱"
date:   2016-05-12 11:04:00 +0800
categories: ktfun update
---

### 1. windows上ICE的DEBUG版本和RELEASE版本不同。
* DEBUG版本是iced.lib、iceutild.lib
* RELEASE版本是ice.lib、iceutil.lib
使用错误的话，编译是正常的，但是运行的时候会报错。

### 2. vs2010里面提示socket相关的结构体重定义
在出问题的源文件开头加上 #include<WinSock2.h>

### 3. 后缀名为hpp的文件，包含的时候要谨慎
* 不应该在.h文件里面包含
* 应该在.cpp文件里面包含