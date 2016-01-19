---
layout: theme:post
title: "Cygwin 安装 PostGIS"
date: 2016-01-17T12:20:33+08:00
---

PostGIS 是一款基于 PostgreSQL 的插件，提供对空间数据的操作。虽然 PostgreSQL 本身可以存储空间数据，但 PostGIS 扩展可以使用户直接使用 SQL 语句进行空间查询、分析和处理。

Cygwin 的官方源中并没有 PostGIS 的二进制发行包，所以要在 Cygwin 上使用 PostGIS 必须自行编译。

PostGIS 依赖于 GEOS、GDAL、XML2、JSON-C、PROJ、PostgreSQL，其中 GEOS 和 GDAL 需要通过源代码安装，[编译GEOS的方法][geos]和[编译GDAl的方法][gdal]可以参考往期博文。其他包可通过如下命令安装：
```
apt-cyg install libxml2-devel libjson-c-devel libproj-devel libreadline-devel libpg-devel
```

安装完依赖之后，到 PostGIS 的[官网][postgis]上下载源代码。解压源代码，进入到解压后的目录，使用 Linux 常用的三条命令进行编译安装：

```bash
./configure
make
make install
```

我在编译的过程中，出现如下错误：
```
/cygdrive/d/Download/postgis-2.2.1/raster/rt_core/rt_raster.c:1966：对‘strnicmp’未定义的引用
```
原因是 `strnicmp` 函数不属于标准库函数，标准库中对应的函数是 `strncasecmp`。因此，将 rt_raster.c 中的 `strnicmp` 替换为 `strncasecmp` 即可。

成功安装 PostGIS 之后，进入需要添加空间扩展的 PostgreSQL 数据库，执行以下命令完成扩展的添加：
```
create extension postgis;
```


[postgis]: http://postgis.net/source/
[geos]: {% post_url 2016-01-17-compiling-geos-on-cygwin %}
[gdal]: {% post_url 2016-01-17-compiling-gdal-on-cygwin %}
