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


# 使用`npm bundle`

那有什么方法相比于上一种方法更干净呢？答案是使用npm-bundle工具将pm2的所有依赖打包，然后到目标服务器上使用`npm install <tarball file>`安装。

首先在联网机器上安装npm-bundle工具：

```
$ npm install -g npm-bundle
```

然后打包pm2：

```
$ npm-bundle pm2
```

上面的命令会生成一个tgz的包文件，复制到目标服务器上安装：

```
$ npm install -g ./pm2-3.2.2.tgz
```

npm-bundle的本质是借助`npm pack`来实现打包的。`npm pack`会打包包本身以及`bundledDependencies`中的依赖，npm-bundle则是将pm2的所有`dependencies`记录到`bundledDependencies`，来实现所有依赖的打包。

这种方式不需要安装多余的`devDependencies`，并且不需要克隆pm2的源码，比第一种方法更方便。

更新：`npm-bundle`对于scoped packages的处理有bug，不能正确地打包，这时考虑采用第一种方式。


[1]: /2018/11/24/npm-package-offline-install.html
