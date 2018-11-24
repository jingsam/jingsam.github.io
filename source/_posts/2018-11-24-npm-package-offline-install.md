---
title: 离线安装npm包的几种方法
date: 2018-11-24 16:56:19
tags:
---

这段时间的工作主题就是Linux
下的“离线部署”，包括mongo、mysql、postgresql、nodejs、nginx等软件的离线部署。平常在服务器上借助apt-get就能轻松搞定的事情，在离线环境下就变得异常艰难。[上一篇文章](1)讲了使用snap离线安装软件的方式，但对于npm包怎么离线部署，snap是无能为力的。本篇文章就来讲一讲离线安装npm包的几种方法。

接下来的部分，我将以离线安装pm2为例来进行说明。pm2是一个进程守护程序，用于启动node集群和服务进程出错时自动重启，在生产环境下部署nodejs应用一般都会使用到。


# 使用`npm link`

使用`npm link`的方式是最常用的方法，具体做法是在联网机器上下载pm2的源码并安装好依赖，拷贝到离线服务器上，最后借助`npm link`将pm2链接到全局区域。

首先，将pm2的源代码克隆下来：

```
$ git clone https://github.com/Unitech/pm2.git
```

然后进入到pm2项目中，安装好所有的依赖：

```
$ cd pm2
$ npm install
```

将安装好依赖的pm2文件夹拷贝到目标服务器上，进入pm2目录链接到全局区域：

```
$ cd pm2
$ npm link
```

这种方式最关键的是借助`npm link`完成链接，但`npm link`这条命令本意是设计给开发人员调试用的。但开发人员开发某个全局命令工具的时候，通过将命令从本地工程目录链接到全局，这样调试的时候，可以实时查看本地代码在全局环境下的执行情况。所以，`npm link`的项目需要安装所有的依赖，包括`dependencies`以及`devDependencies`，而我们如果只是使用而不是开发某个包的话，正常情况下不应该安装`devDependencies`。

总而言之，这种方式优点是比较简单，缺点是安装了不需要的`devDependencies`，对于有“洁癖”的人是难以忍受的。


# 使用`npm install <folder>`

那有什么方法相比于上一种方法更干净呢？答案是`npm install <folder>`直接从文件夹安装。

同样以pm2为例，首先我们需要准备pm2包，可以在联网的机器上执行：

```
$ npm install pm2 --global-style
```

上面的`--global-style`很关键，表示将pm2安装到node_modules中一个单独的pm2文件夹中，这样我们可以方便地将pm2及其所有相关依赖都拷贝出来。也可以使用`npm install pm2 -g`安装到全局的node_modules，其文件布局是一样。

然后，将pm2文件拷贝到目标机器上，使用以下命令安装：

```
$ npm install pm2/ -g
```

这种方式不需要安装多余的`devDependencies`，并且不需要克隆pm2的源码，比第一种方法更干净环保。


[1]: /2018/11/24/npm-package-offline-install.html
