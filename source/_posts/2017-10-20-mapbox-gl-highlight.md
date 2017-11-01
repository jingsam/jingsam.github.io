---
title: mapbox-gl如何高效地高亮要素
date: 2017-10-20 08:42:01
tags:
---

mapbox-gl基于矢量瓦片的前端渲染技术，使得要素高亮变得简单。要素高亮具体如何实现，有如下三种方法：

# 第一种：动态过滤

这种方法的基本思路是添加一个高亮图层，然后根据鼠标hover的要素id，动态地改变filter条件，实现要素的高亮。

使用到的主要接口是`map.on('mousemove', layer, e)`，其中`e`可以获取到当前鼠标位置的features，效果如下：

<iframe width="100%" height="500" src="//jsfiddle.net/jingsam/f3au0qLo/4/embedded/result,js,html,css" frameborder="0"></iframe>

测试中发现，在图斑比较密集的情况下，高亮非常卡，滞后严重，效率并不高。


# 第二种：数据源镜像

第二种方法是对第一种方法的改进，思路如下：对原始数据源做一个镜像，即添加一个新的数据源，名称不通但指向的是同一套数据,例如下面示例中的`source-mirror`，高亮图层的数据源指向`source-mirror`。改进后的效果如下：

<iframe width="100%" height="500" src="//jsfiddle.net/jingsam/rj16bqa4/4/embedded/result,js,html,css" frameborder="0"></iframe>

测试可以发现，稍微改进一下，高亮的效率提升很大。

但为什么做一个数据源镜像就可以显著地提高高亮的效率呢？我猜想是mapbox-gl在绘制时，会按照数据源对图层分组。指向同一个数据源的图层组中，任意一个图层的渲染条件改变，将会以整个图层组为粒度重新进行运算。在第一种方法中，高亮图层和其他图层都指向同一个数据源，动态地改变高亮图层的过滤条件，导致了很大的运算开销。而第二种方法，为高亮图层单独分配一个数据源，动态地改变高亮图层的过滤条件，也只会导致高亮图层的重绘。

# 第三种：动态生成高亮数据源

我们能不能再进一步提高高亮的效率呢？有。第三种方法的思路是：为高亮图层生成一个空的GeoJSON数据源，然后将鼠标hover到的要素动态地填充到数据源中。

使用的接口只要是`map.getSource(source).setData(geojson)`，其中`getSource`用于根据sourceId获取数据源，`setData`用于动态更新数据。效果如下：

<iframe width="100%" height="500" src="//jsfiddle.net/jingsam/ykoyet0w/4/embedded/result,js,html,css" frameborder="0"></iframe>

测试可以发现，这种方法甚至比第二种方法效率更高。原因在于要高亮的要素往往很小，前段切成瓦片可能就一两张，因此不用去整体从原始数据源的瓦片种过滤，效率会更高。


# 总结

第一种方法最常用，小批量数据效率还行，但涉及到大数据量情况下，效率就不太理想了。

第二种方法通过小小的改进，就可以极大地提高效率，实现起来也很简单。

第三种方法效率最高，但其中引入了`turf.union`去合并features，带来了额外的依赖。因此，从简单和优雅的角度来说，我更推荐使用第二种方法。


