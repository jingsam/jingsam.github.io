---
layout: theme:post
title: "在 Cygwin 上编译 GDAL"
---

/home/Sam/download/gdal-2.0.1/frmts/o/.libs/dgif_lib.o: In function `DGifOpenFileHandle':
/cygdrive/d/Download/gdal-2.0.1/frmts/gif/giflib/dgif_lib.c:111: undefined reference to `setmode'
/cygdrive/d/Download/gdal-2.0.1/frmts/gif/giflib/dgif_lib.c:111:(.text+0x8e1): relocation truncated to fit: R_X86_64_PC32 against undefined symbol `setmode'
/home/Sam/download/gdal-2.0.1/frmts/o/.libs/egif_lib.o: In function `EGifOpenFileHandle':
/cygdrive/d/Download/gdal-2.0.1/frmts/gif/giflib/egif_lib.c:137: undefined reference to `setmode'
/cygdrive/d/Download/gdal-2.0.1/frmts/gif/giflib/egif_lib.c:137:(.text+0x4c7): relocation truncated to fit: R_X86_64_PC32 against undefined symbol `setmode'

#include <io.h>