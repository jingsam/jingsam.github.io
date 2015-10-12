---
layout: theme:post
title: Netlogo读写GIS数据的方法
---
使用NetLogo进行地理过程模拟时，不可避免地要对GIS数据进行读写，本文主要讲栅格数据的读写，矢量数据留待以后再讲。

Netlogo目前只支持ESRI ASCII Grid(后缀通常为.asc或.grd)格式的栅格数据，所以其他格式的数据请先使用ArcGIS里面的“Raster to ASCII”工具进行转换。

NetLogo读写GIS数据需要gis扩展支持，因此需要代码页前部声明：
```netlogo
extensions [gis]
```

读取数据前，首先需要设置坐标系，但并不是必须的。NetLogo支持地理坐标系和投影坐标系，以WKT格式描述坐标系，需提供坐标系文件(后缀通常为.prj)，代码如下：
```netlogo
gis:load-coordinate-system "Transverse_Mercator.prj"
```

对于投影坐标系，NetLogo目前只支持有限的几种坐标系：

|Albers_Conic_Equal_Area      |Lambert_Conformal_Conic_2SP |Polyconic          |
|Lambert_Azimuthal_Equal_Area |Mercator_1SP                |Robinson           |
|Azimuthal_Equidistant        |Miller                      |Stereographic      |
|Cylindrical_Equal_Area       |Oblique_Mercator            |Transverse_Mercator|
|Equidistant_Conic            |hotine_oblique_mercator
|Gnomonic                     |Orthographic

设置完坐标系后，即可利用函数`gis:load-dataset`读取数据：
```netlogo
let data gis:load-dataset dem.asc
```

将数据读取至变量`data`之后，一般还需要将数据写入瓦片：
```netlogo
gis:set-world-envelope (gis:envelope-of data)
gis:apply-raster data patch-own-variable
```

目前NetLogo仅仅支持栅格数据的保存，代码如下：
```netlogo
gis:store-dataset data mydata.asc
```

最后附上完整代码：

```netlogo
extensions [gis]      ;读写GIS数据需要用到gis扩展

patches-own [patch-own-variable]

;读取
to import-data [file]
  gis:load-coordinate-system (word projection ".prj")    ;设置坐标系（非必须）
  let data gis:load-dataset file

  gis:set-world-envelope (gis:envelope-of data)          ;将窗口范围设置为数据框范围
  gis:apply-raster data patch-own-variable               ;将数据读取至patch-own-variable变量
end

;保存
to save-data [data file]
  gis:store-dataset data file
end
```
