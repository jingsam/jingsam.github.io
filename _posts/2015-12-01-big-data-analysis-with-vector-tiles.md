---
layout: theme:post
title: "全世界的公路有多长？基于矢量瓦片的大数据分析"
date: 2015-12-01T15:53:09+08:00
---

随着地理位置服务（LBS）的兴起，当今地理数据的增长速度远超以往。地理数据的增长不仅表现在数据量的增长，其种类也多种多样，面面俱到。地理大数据以更精细的数据描述整个地球，使我们以更精细地尺度了解整个世界成为了可能。

然而，地理大数据带来了机会也同时带来了挑战。巨量的数据使得传统地理分析方法无能为力，因此要提出与大数据相适应的理计算方法。针对基于单机、基于数据库的传统地理分析方法，一些研究人员探索并行处理的方法。但是这些方法仍然基于传统的数据模型和格式，当数据量增大到一定程度时最终会遇到处理瓶颈。为了处理地理大数据，需要突破传统地理分析方法的思维模式，扩展出新的数据模型和与之相适应的地理计算方法。

## 矢量瓦片

矢量瓦片（vector tiles）是一种新颖的数据格式，是将矢量要素按规则格网切割而形成的瓦片数据。矢量瓦片的数据组织方式，非常便于算法分块并行处理。矢量瓦片支持分而冶之的计算方法，使地理计算方法易于从单核到多核、从单机向集群的扩展。总而言之，矢量瓦片简直就是为处理地理大数据而生。

接下来，我将通过统计全世界的公路里程这个示例，向大家展示矢量瓦片的威力。全世界的公路到底有多长？恐怕大多数人心里没底。要解决这个问题有个简单方法——查询统计数据，[维基百科][1]上面有世界各国的公路里程。但是，我们不免怀疑统计数据的可信程度，俗话说相信别人比不上相信自己，我们为什么不亲自动手计算下呢？

巧妇难为无米之炊，要统计公路里程首先要有公路的数据。OpenStreetMap（OSM）是一个由全世界志愿者贡献地理数据的网络地图项目，堪称地理数据的维基百科。OSM 公开了全世界地理矢量数据集，任何人都可以免费地下载使用。我下载了全球矢量瓦片（zoom 12）数据集 [OSM-QA-Tiles][2] ，数据集大小有 22 GB。这个数据集里面的矢量数据都是未经过制图综合，所以数据精度是有保障的。


## TileReduce

然后，我要选择一个合适的计算框架来处理如此巨量的数据。[TileReduce][3] 是 Mapbox 的一个开源项目，它被设计用来对矢量瓦片做 MapReduce 操作。TileReduce 基于 Javascript 开发，可以借助 Nodejs 运行在各个平台上。运行以下命令，完成开发环境的搭建：
```
mkdir tile-reduce-test
cd tile-reduce-test
npm init
npm install tile-reduce --save
```

接下来，新建`index.js`，负责规约操作。其中`mbtiles`指定了数据的位置，所以记得将它指向你下载的数据的位置。
```javascript
// index.js

'use strict';

var tileReduce = require('tile-reduce');
var path = require('path');

var total = 0;

var remoteSources = [
  {
    name: 'osm',
    mbtiles: path.join(__dirname, '/latest.planet.z12.mbtiles'),
    layers: ['osm']
  }
];

tileReduce({
    bbox: [-180.0,-85.1,180.0,85.1],
    zoom: 12,
    map: path.join(__dirname, '/count.js'),
    sources: remoteSources
  })
  .on('reduce', function(num) {
    total += num;
  })
  .on('end', function() {
    console.log('total: %d', total);
  });

```

新建`count.js`，负责对每个瓦片进行处理。里面用到了`turf`来计算道路线段的长度，所以需要`npm install turf --save`来安装这个库。
```javascript
// count.js

'use strict';

var turf = require('turf');

module.exports = function(tilelayers, tile, writeData, done) {
  var length = 0;
  var layer = tilelayers.osm.osm;

  var roadTypes = [
    'motorway',
    'trunk',
    'primary',
    'secondary',
    'tertiary',
    'unclassified',
    'residential',
    'road',
  ];

  for (var i = 0; i < layer.features.length; i++) {
    var feature = layer.features[i];
    if (roadTypes.indexOf(feature.properties.highway) > -1) {
      length += line_length(feature, 'kilometers');
    }
  }

  done(null, length);
};


function line_length(feature, units) {
  var length = 0;

  if (feature.geometry.type == 'LineString') {
    length = turf.lineDistance(feature, units);
  }

  if (feature.geometry.type == 'MultiLineString') {
    for (var i = 0; i < feature.geometry.coordinates.length; i++) {
      var linestring = turf.linestring(feature.geometry.coordinates[i]);
      length += turf.lineDistance(linestring, units);
    }
  }

  return length;
}

```


大功即将告成，运行以下命令开始计算：
```
node index.js
```

我在一台 32 核服务器上运行了 30 min 58 s，处理了 1680 万个矢量瓦片，最终得到了全球公路总里程有 3215 万公里。这个速度我想当满意，毕竟有这么大的数据量。

查询维基百科上的数据得到的总里程是 6428 万公里，少了近一倍。我觉得有两方面的原因：
1. 我计算的道路只包括 OSM 中 Tag 为 `motorway, trunk, primary, secondary, tertiary, unclassified, residential, road`的公路，实际上还包括其他类型的公路没有统计；
2. OSM 的数据完善程度在世界各地区不一样，欧洲和美洲比较完善，而中国和非洲的数据密度偏低，所以会造成部分道路未统计到。


## 总结

虽然计算结果并不准确，重要的是我们验证了应用矢量瓦片进行地理大数据分析的技术可行性。在本文中只是做了一个简单的公路里程统计，矢量瓦片可以支持更深层次统计分析，这方面还有广泛的研究与应用价值。

注：本文的所用的示例代码可以在[这里][4]找到。

[1]: https://zh.wikipedia.org/wiki/%E5%90%84%E5%9B%BD%E5%85%AC%E8%B7%AF%E9%87%8C%E7%A8%8B%E5%88%97%E8%A1%A8
[2]: http://osmlab.github.io/osm-qa-tiles/
[3]: https://github.com/mapbox/tile-reduce
[4]: https://github.com/jingsam/tile-reduce-test
