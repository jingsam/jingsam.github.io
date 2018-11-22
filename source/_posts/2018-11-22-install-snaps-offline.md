---
title: Ubuntu离线部署snap软件包
date: 2018-11-22 21:03:07
tags:
---

Ubuntu借助包管理器apt-get安装软件包很方便，前提是服务器要能够联网。政府或企业内网的服务器，通常是不与互联网连通的，这时候部署软件只能借助文件拷贝的方式，感觉回到了原始时代。更大的问题是，要部署的软件包需要先安装很多依赖，依赖自己可能还有依赖，并且各种依赖还有版本要求。如果通过手动下载各个依赖包的方法部署，对于复杂的软件，变成了不可能的任务。

在离线部署方面，Windows明显比Linux做得好，Windows软件包通常会将软件所需的依赖打包，部署时只需拷贝一个软件安装包即可。那Linux有没有类似Windows软件安装包的东西呢？幸运的是，Ubuntu提供了snap软件包机制，可以用来简化离线部署。

snap软件包类似于windows的软件安装包，将所需的依赖都统一打包到软件包中，部署时只需拷贝snap文件。另外，snap也加强了安全隔离机制，通过注册软件包的签名和权限控制信息，使得snap软件运行在“沙盒”环境中。

从Ubuntu 16.04起，snap环境是自带的，可以直接使用。如果是早于16.04的版本且服务器不能联网，安装snap环境很困难，你只能自求多福了。下面以安装docker为例，来说明离线安装snap包的方法。


# 下载

首先，我们需要在能联网的机器上将相关的snap包下载下来。不仅要下载docker软件包，还需要下载core软件包。core软件包是snap的核心运行时，几乎所有的snap包都依赖core运行时，Ubuntu 16.04自带了snap环境却没安装core运行时，实在是让人有些搞不懂。

下载有snap包两种方式。一种方法是在能联网的Ubuntu上使用`snap download`命令下载：

```
$ snap download core
Fetching snap "core"
Fetching assertions for "core"
Install the snap with:
   snap ack core_5897.assert
   snap install core_5897.snap
$ snap download docker
Fetching snap "docker"
Fetching assertions for "docker"
Install the snap with:
   snap ack docker_321.assert
   snap install docker_321.snap
```

以上命令将会得到`.assert`和`.snap`两类文件，其中`.assert`是软件包的元数据信息，包括签名和权限控制信息，`.snap`是实际的安装文件。

另外一种方法是到[uApp Explorer](https://uappexplorer.com/snaps)网站上下载，好处是不需要有Ubuntu环境，缺点是只能下载`.snap`文件，无法下载`.assert`文件。


# 安装

安装snap包的方法很简单：将软件包拷贝到服务器上，安装时首先注册`.assert`，然后再安装`.assert`。对于首次安装，需要安装core和docker两个软件包：

```
$ sudo snap ack core_5897.assert
$ sudo snap install core_5897.snap
$ sudo snap ack docker_321.assert
$ sudo snap install docker_321.snap
```

这样就完成了docker的安装。

要是我们是从[uApp Explorer](https://uappexplorer.com/snaps)网站上下载的软件包，缺少`.assert`文件怎么办？其实我们可以使用`snap install --dangerous`方式安装：

```
$ sudo snap install core.snap --dangerous
$ sudo snap install docker.snap --dangerous
```

如`--dangerous`所提示的是，这种模式有些“危险”。这是因为缺少`.assert`文件所描述的签名信息和权限控制信息，意味着软件不是在“沙盒”环境下执行的，运行过程不受控。其实，大可不必紧张，只要snap包来源可信，一般没什么问题，毕竟以前咱们没有snap包的时候不都是这么干的么。
