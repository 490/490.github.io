---
title: Windows通过xrdp远程连接Linux桌面
date: 2019-06-08 10:39:45
tags: 系统
---
一般情况下我们用ssh客户端远程登陆Linux系统，至于图形界面下的linux远程登陆工具，我们一般都会想到vnc，但它的安全性不够，在这里，我将介绍XRDP的安装配置方法。我们可以很方便的通过Windows远程桌面Linux。

<!--more-->

1.在Linux系统上安装桌面系统

`#yum groupinstall "Desktop"`

2.安装xrdp

`#yum install xrdp`

　　xrdp的配置文档在/etc/xrdp目录下的xrdp.ini和sesman.in，一般选择默认。

3.由于xrdp需要vncserver，在此我们安装tigervnc-server

`#yum install tigervnc-server`

4.启动xrdp

`#service xrdp start`

通过以上四步，xrdp已经安装成功，接下来配置Windows

运行Windows的mstsc

