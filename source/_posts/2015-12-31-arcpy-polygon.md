---
layout: theme:post
title: "arcpy.Polygon 数据结构"
date: 2015-12-31T15:11:11+08:00
---

`arcpy.Polygon` 是描述多边形的几何类，并不像 ArcEngine 中区分 polygon 和 multipolygon，而是采用同一套数据结构描述两类多边形。

在 arcpy 中，一个多边形（polygon）可能包含多个部件（part），而每个部件除了必须包含一个外环（exterior ring），还可能包含几个内环（interior ring），每个环由多个封合的坐标点（point）构成。polygon的层次结构如下：
```bash
polygon
├── part0
│   ├── ring0
│   │    ├── point0
│   │    ├── point1
│   │    ├── point2
│   │    ├── ...
│   ├── ring1
│   ├── ring2
│   └── ...
├── part1
├── part2
├── ...
```

如下图是一个 multipolygon，它包含 3 个部件，其中第一个部件包含一个外环（ring0）和一个内环（ring1）。

![polygon](/assets/polygon.png)

在 arcpy 中，每个 polygon 是由多个 part 构成的列表表示，每个 part 直接由一系列坐标串表示。在这里我们可以发现，没有表示 ring 的数据结构。其实 arcpy 会环与环之间添加一个 None 坐标点，在遍历坐标串的过程中，通过 None 结点来判断环的结束。如下如图是包含一个内环的 polygon：

![ring](/assets/ring.png)

如果打印这个polygon，我们将会看到：
```
[[(1, 2), (3, 4), (5, 6), (7, 8), None, (8, 9), (6, 7), (5, 4), (3, 2)]]
```

我们同时可以发现对于外环，坐标串是按照顺时针排列的；对于内环，按逆时针排列。当我们用梯形求和的方法计算多边形的面积时，采用这种顺序会使得外环的面积为正，内环为负，这样我们就可以直接对环的面积求和，而不用判断是外环还是内环。

当我们打印坐标点（point）的时候，并不会看到是两个坐标的元组，而是 4 个：
```python
Point (1, 2, #, #)
```
实际上，每个 point 包含 （X, Y, Z, M）4 个属性, 在大地坐标系的情况下，其中 X 表示经度，Y 表示纬度，Z 表示高程，M 表示里程。

最后，附上一段求一个多边形的所有的环的函数：
```python
def get_rings(polygon):
    rings = []
    for part in polygon:
        ring = []
        for point in part:
            if point:
                ring.append([point.X, point.Y])
            else:
                rings.append(ring)
                ring = []
        if ring:
            rings.append(ring)

    return rings
```
