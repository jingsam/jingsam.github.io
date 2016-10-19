---
title: Nodejs开发工具遇到的坑
date: 2016-04-14 16:36:25
tags:
---


最近在开发一个在线制图平台，开发语言为Javascript，前端用Vuejs + Material Design Lite做界面，后端用Express做地图服务。

开发主要在Windows上进行。看过我博客的人，应该知道我对Cygwin的爱，因此采用Cygwin作为终端，而不是常用的cmd。Cywin是一个在Windows运行的Unix环境，是典型的身在曹营心在汉。Cygwin的用户还是比较小众的，很多跨平台软件开发时往往是忽视Cygwin的兼容性，造成很多问题。这些问题大部分发生于Cywin中执行Windows命令，例如nodejs不提供Cygwin版本，因此只能调用Windows下安装的Nodejs。大部分npm安装的包都能够在Cygwin中执行，但是有些工具没有考虑到Cygwin的兼容性，导致执行出错。

以下是我在开发中遇到的一些坑，在此记录一下，以后会持续更新：

~~1. nodemon无法在Cygwin下运行，只能在cmd下运行。~~
1. nodemon在windows下运行需要调用cmd.exe, 所以在Cygwin下运行要将`/cygdrive/c/Windows/system32`添加到PATH环境变量中;
2. istanbul和mocha集成的时候，在Windows运行的时候命令要写成`istanbul cover ./node_modules/mocha/bin/_mocha`，直接用`istanbul cover _mocha`是不行的。
