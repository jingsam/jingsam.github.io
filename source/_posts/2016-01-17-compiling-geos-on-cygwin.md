---
layout: theme:post
title: "在 Cygwin 上编译 GEOS"
date: 2016-01-17T00:34:45+08:00
---

GEOS (Geometry Engine - Open Source) 是 Java Topology Suite (JTS) 的 C++ 实现，提供大量的几何操作函数。

Cygwin 的官方源中并没有 GEOS 的二进制发行包，所以要在 Cygwin 上使用 GEOS 必须自行编译。

编译的第一步，需要到 GEOS 的[官网][geos]上下载源代码。

源代码解压后，进入到解压后的目录，使用 Linux 常用的三条命令进行编译安装：

```bash
./configure
make
make install
```

我在编译的过程中，出现如下错误：
```bash
libtool: compile:  g++ -DHAVE_CONFIG_H -I. -I../include -I../include/geos -I../include -DGEOS_CAPI_VERSION=\"3.5.0-CAPI-1.9.0\" -DGEOS_JTS_PORT=\"1.13.0\" -DGEOS_INLINE -pedantic -Wall -ansi -Wno-long-long -ffloat-store -g -O2 -MT libgeos_c_la-geos_ts_c.lo -MD -MP -MF .deps/libgeos_c_la-geos_ts_c.Tpo -c geos_ts_c.cpp  -DDLL_EXPORT -DPIC -o .libs/libgeos_c_la-geos_ts_c.o
geos_ts_c.cpp: 在成员函数‘void GEOSContextHandle_HS::NOTICE_MESSAGE(std::string, ...)’中:
geos_ts_c.cpp:225:81: 错误：‘vsnprintf’在此作用域中尚未声明
       int result = vsnprintf(msgBuffer, sizeof(msgBuffer) - 1, fmt.c_str(), args);
                                                                                 ^
geos_ts_c.cpp: 在成员函数‘void GEOSContextHandle_HS::ERROR_MESSAGE(std::string, ...)’中:
geos_ts_c.cpp:246:81: 错误：‘vsnprintf’在此作用域中尚未声明
       int result = vsnprintf(msgBuffer, sizeof(msgBuffer) - 1, fmt.c_str(), args);
```
错误提示 `vsnprintf()` 函数未声明，查询得知此函数出现于 C++11 标准。而上面的编译命令中有 `-ansi`，限定了以 C++98 标准进行编译，所以无法找到 `vsnprintf()` 函数的声明和定义。

解决方案很简单：进入到源代码目录下的 capi 子目录，找到 Makefile 文件，去掉编译命令中的 `-ansi` 选项。


[geos]: http://trac.osgeo.org/geos/