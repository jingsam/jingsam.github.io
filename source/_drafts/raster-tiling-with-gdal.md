---
title: 使用GDAL完成影像切片
tags:
---

gdal_transform -of GPKG chengdu.tif chengdu.gpkg -co TILING_SCHEME=InspireCRS84Quad

gdaladdo chengdu.gpkg