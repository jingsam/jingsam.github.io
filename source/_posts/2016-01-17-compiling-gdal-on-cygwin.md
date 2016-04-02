---
layout: theme:post
title: "在 Cygwin 上编译 GDAL"
date: 2016-01-17T00:55:13+08:00
---

GDAL (Geospatial Data Abstraction Library) 是最广泛使用的空间数据操作库，包括栅格数据和矢量数据的读写。

Cygwin 的官方源中并没有 GDAL 的二进制发行包，所以要在 Cygwin 上使用 GDAL 必须自行编译。

编译的第一步，需要到 GDAL 的[官网][gdal]上下载源代码。

源代码解压后，进入到解压后的目录，使用 Linux 常用的三条命令进行编译安装：

```bash
./configure
make
make install
```

我在编译的过程中，出现如下错误：

```bash
/home/Sam/download/gdal-2.0.1/frmts/o/.libs/dgif_lib.o: In function `DGifOpenFileHandle':
/cygdrive/d/Download/gdal-2.0.1/frmts/gif/giflib/dgif_lib.c:111: undefined reference to `setmode'
/cygdrive/d/Download/gdal-2.0.1/frmts/gif/giflib/dgif_lib.c:111:(.text+0x8e1): relocation truncated to fit: R_X86_64_PC32 against undefined symbol `setmode'
/home/Sam/download/gdal-2.0.1/frmts/o/.libs/egif_lib.o: In function `EGifOpenFileHandle':
/cygdrive/d/Download/gdal-2.0.1/frmts/gif/giflib/egif_lib.c:137: undefined reference to `setmode'
/cygdrive/d/Download/gdal-2.0.1/frmts/gif/giflib/egif_lib.c:137:(.text+0x4c7): relocation truncated to fit: R_X86_64_PC32 against undefined symbol `setmode'
```

查询得知，`setmode()` 函数并不是标准函数库的一部分，它是一个 BSD 函数。使用 `setmode()` 函数，需要包含以下头文件：
```c++
#include <io.h>
```

因此解决方案是：找到 dgif_lib.c 和 egif_lib.c，分别在 `setmode()` 函数前添加头文件 `#include <io.h>` 。

更新：在 GDAL 2.0.2 中，编译 gdalserver.c 时会报 fd_set 未定义的错误，解决方法是在该文件的46行下面添加 `#include <sys/select.h>` 。

[gdal]: http://trac.osgeo.org/gdal/wiki/DownloadSource
