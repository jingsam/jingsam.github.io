---
layout: theme:post
title: Netlogo读写GIS数据的方法
---
使用NetLogo进行地理过程模拟时，不可避免地要对GIS数据进行读写，本文主要讲栅格数据的读写，矢量数据留待以后再讲。

Netlogo目前只支持ESRI ASCII Grid(后缀通常为.asc或.grd)格式的栅格数据，所以其他格式的数据请先使用ArcGIS里面的“Raster to ASCII”工具进行转换。废话不多说，上代码：

```netlogo
extensions [gis]      ;读写GIS数据需要用到gis扩展

patches-own [value]

;读取
to import-data [file]
  gis:load-coordinate-system (word projection ".prj")    ;设置坐标系（非必须）
  let data gis:load-dataset file
  gis:set-world-envelope (gis:envelope-of data)          ;将窗口范围设置为数据框范围

  gis:apply-raster data value                            ;将数据读取至patch-own的value变量
end

;显示
to display-data
  ask patches[
    ifelse value = 0
    [set pcolor yellow]
    [ifelse value = 1
      [set pcolor black]
      [set pcolor white]    ;NoData以及其他值显示为白色
    ]
  ]
end

;保存
to save-data [data file]
  gis:store-dataset data file
end
```
