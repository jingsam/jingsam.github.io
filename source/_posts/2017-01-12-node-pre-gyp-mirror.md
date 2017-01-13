---
title: 使用node-pre-gyp加速二进制包安装
date: 2017-01-12 10:36:34
tags:
---

node-pre-gyp是一个分发nodejs二进制程序包的工具，负责将预编译好的二进制程序直接下载到用户目录。它介于npm与node-gyp之间，只在相应平台二进制包不存在时才调用node-gyp编译。

node-pre-gyp存在的意义是什么呢？一些简单的nodejs C++扩展直接从源代码编译安装问题不大，但复杂的扩展编译环境难搭建、编译耗时长，因而从源代码安装非常麻烦。node-pre-gyp能够将预编译好的二进制包直接下载到用户目录，只在必要的时候才调用node-gyp从源代码编译，大大加快了nodejs C++扩展的安装速度。

node-pre-gyp需要开发者将各平台编译好的二进制包上传到网络上，并在package.json的`binary`字段指明二进制包的位置。然而，很多开发者选择将二进制包上传到aws上，导致国内无法正常下载（被墙）。幸好，可以在npm中设置`--{module_name}_binary_host_mirror`选项来指定二进制包的位置。例如，安装v8-profiler可以使用如下命令安装：
```
npm install v8-profiler --profiler_binary_host_mirror=https://npm.taobao.org/mirrors/node-inspector/
```

为了让国内开发者也能享受到node-pre-gyp带来的好处，我使用阿里云做了一个镜像，镜像的地址是：
```
https://foxgis.oss-cn-shanghai.aliyuncs.com
```

如果要安装sqlite3，执行以下命令：
```
npm install sqlite3 --sqlite3_binary_host_mirror=https://foxgis.oss-cn-shanghai.aliyuncs.com
```

目前是放到阿里云OSS上的，速度还可以，镜像上面包有：
```
mapnik mapbox-gl-native sqlite3 fontnik gdal osrm zipfile
```

如果有其他包需要放到上面的，请给我留言或者发邮件（abc#whu.edu.cn）。


