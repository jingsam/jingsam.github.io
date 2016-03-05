---
title: Mapbox矢量瓦片标准
date: 2016-03-04 18:30:15
tags:
---

以下是[《Mapbox Vector Tile Specification》](https://github.com/mapbox/vector-tile-spec/tree/master/2.1)最新的2.1版的中文翻译，以便查阅。

# Vector Tile Specification
# 矢量瓦片标准

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in
this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).
本文档中的“**必须**”、“**必须不**”、“**必备**”、"**应该**"、“**不应该**”、“**建议**”、“**可以**”、“**可选**”的含义参照[RFC 2119](https://www.ietf.org/rfc/rfc2119.txt)。

## 1. Purpose
## 1. 目标

This document specifies a space-efficient encoding format for tiled geographic vector data. It is designed to be used in browsers or server-side applications for fast rendering or lookups of feature data.
本文档规定了一种节省存储空间的矢量瓦片数据编码格式。这种格式应用于客户端或服务端高效渲染或查询要素信息。

## 2. File Format
## 2. 文件格式

The Vector Tile format uses [Google Protocol Buffers](https://developers.google.com/protocol-buffers/) as a encoding format. Protocol Buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.
矢量瓦片文件采用[Google Protocol Buffers](https://developers.google.com/protocol-buffers/)进行编码。Google Protocol Buffers是一种兼容多语言、多平台、易扩展的数据序列化格式。

## 2.1. File Extension
### 2.1. 文件后缀

The filename extension for Vector Tile files SHOULD be `mvt`. For example, a file might be named `vector.mvt`.
矢量瓦片文件的后缀**应该**为`mvt`。例如，`vector.mvt`。

## 2.2. Multipurpose Internet Mail Extensions (MIME)
### 2.2 MIME类型

When serving Vector Tiles the MIME type SHOULD be `application/vnd.mapbox-vector-tile`.
矢量瓦片的MIME类型**应该**设置为`application/vnd.mapbox-vector-tile`。

## 3. Projection and Bounds
## 3. 投影和范围

A Vector Tile represents data based on a square extent within a projection. A Vector Tile SHOULD NOT contain information about its bounds and projection. The file format assumes that the decoder knows the bounds and projection of a Vector Tile before decoding it.
矢量瓦片表示的是投影在正方形区块上的数据。矢量瓦片**不应该**包含范围和投影信息。解码方被假定了解矢量瓦片的范围和投影信息。

[Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator) is the projection of reference, and [the Google tile scheme](http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/) is the tile extent convention of reference. Together, they provide a 1-to-1 relationship between a specific geographical area, at a specific level of detail, and a path such as `https://example.com/17/65535/43602.mvt`.
[Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator)是默认的投影方式，[Google tile scheme](http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/)是默认的瓦片编号方式。两者一起完成了与任意范围、任意精度的地理区域的一一对应，例如`https://example.com/17/65535/43602.mvt`。

Vector Tiles MAY be used to represent data with any projection and tile extent scheme.
矢量瓦片**可以**用来表示任意投影方式、任意瓦片编号方案的数据。

## 4. Internal Structure
## 4. 内部结构

This specification describes the structure of data within a Vector Tile. The reader should have an understanding of the [Vector Tile protobuf schema document](vector_tile.proto) and the structures it defines.
这部分内容描述矢量瓦片的数据结构。读者需要先了解[矢量瓦片protobuf编码方案文件](https://github.com/mapbox/vector-tile-spec/blob/master/2.1%2Fvector_tile.proto)中的结构定义。

### 4.1. Layers
### 4.1. 图层

A Vector Tile consists of a set of named layers. A layer contains geometric features and their metadata. The layer format is designed so that the data required for a layer is contiguous in memory, and so that layers can be appended to a Vector Tile without modifying existing data.
矢量瓦片由一组具名的图层构成。每个图层包含几何要素和元数据信息。设计的图层格式能够保证图层数据能够在内存中顺序排列，由此在图层组末尾添加一个新的图层就不用更改已有的数据。

A Vector Tile SHOULD contain at least one layer. A layer SHOULD contain at least one feature.
每块矢量瓦片**应该**至少包含一个图层。每个图层**应该**至少包含一个要素。

A layer MUST contain a `version` field with the major version number of the Vector Tile specification to which the layer adheres. For example, a layer adhering to version 2.1 of the specification contains a `version` field with the integer value `2`. The `version` field SHOULD be the first field within the layer. Decoders SHOULD parse the `version` first to ensure that they are capable of decoding each layer. When a Vector Tile consumer encounters a Vector Tile layer with an unknown version, it MAY make a best-effort attempt to interpret the layer, or it MAY skip the layer. In either case it SHOULD continue to process subsequent layers in the Vector Tile.
每个图层**必须**包含一个`version`字段表示此图层所遵守的《矢量瓦片标准》的主版本号。例如，某个图层遵守2.1版本的标准，那么它的`version`字段的值则为整数`2`。`version`字段**应该**设定为图层的第一个字段。解码器**应该**首先解析`version`字段，以确定是否能够解析该版本的图层。当遇到一个未知版本的矢量瓦片图层时，解码器**可以**尝试去解析它，或者**可以**跳过该图层。以上两种情况下，解码器都**应该**继续解析后续的图层。

A layer MUST contain a `name` field. A Vector Tile MUST NOT contain two or more layers whose `name` values are byte-for-byte identical. Prior to appending a layer to an existing Vector Tile, an encoder MUST check the existing `name` fields in order to prevent duplication.
每个图层**必须**包含一个`name`字段。每块矢量瓦片**必须不**包含两个或两个以上的图层具有相同`name`值。在向一块矢量瓦片添加一个新的图层之前，编码器**必须**检查已有的`name`值以防止重复。

Each feature in a layer (see below) may have one or more key-value pairs as its metadata. The keys and values are indices into two lists, `keys` and `values`, that are shared across the layer's features.
图层中的每个要素**可以**包含一个或多个key-value作为它的元数据（见下文）。所有要素的key和value被分别索引为两个列表——`keys`和`values`——为图层中的所有要素所共享。

Each element in the `keys` field of the layer is a string. The `keys` include all the keys of features used in the layer, and each key may be referenced by its positional index in this set of `keys`, with the first key having an index of 0. The set of `keys` SHOULD NOT contain two or more values which are byte-for-byte identical.
图层`keys`字段的每个元素都是字符串。`keys`字段包含了图层中所有要素的key，并且每个key可以通过它在`keys`列表中的索引号引用，第一个key的索引号是0 。`keys`列表**必须不**包含两个或两个以上key是一样的。

Each element in the `values` field of the layer encodes a value of any of several types (see below). The `values` represent all the values of features used in the layer, and each value may be referenced by its positional index in this set of `values`, with the first value having an index of 0. The set of `values` SHOULD NOT contain two or more values of the same type which are byte-for-byte identical.
图层`values`字段的每个元素是多种类型的值的编码（见下文）。`values`字段包含了图层中所有要素的value，并且每个value可以通过它在`values`列表中的索引号引用，第一个value的索引号是0 。`values`列表**必须不**包含两个或两个以上value是一样的。

In order to support values of varying string, boolean, integer, and floating point types, the protobuf encoding of the `value` field consists of a set of `optional` fields. A value MUST contain exactly one of these optional fields.
为了支持字符串型、布尔型、整型、浮点型多种类型的值，对`value`字段的编码包含了一组`optional`字段。每个value**必须**包含其中的一个字段。

A layer MUST contain an `extent` that describes the width and height of the tile in integer coordinates. The geometries within the Vector Tile MAY extend past the bounds of the tile's area as defined by the `extent`. Geometries that extend past the tile's area as defined by `extent` are often used as a buffer for rendering features that overlap multiple adjacent tiles.
图层**必须**包含一个`extent`字段，表示瓦片的宽度和高度，以整数表示。矢量瓦片中的几何坐标**可以**超出`extent`定义的范围。超出`extent`范围的几何要素被经常用来作为缓冲区，以渲染重叠在多块相邻瓦片上的要素。

For example, if a tile has an `extent` of 4096, coordinate units within the tile refer to 1/4096th of its square dimensions. A coordinate of 0 is on the top or left edge of the tile, and a coordinate of 4096 is on the bottom or right edge. Coordinates from 1 through 4095 inclusive are fully within the extent of the tile, and coordinates less than 0 or greater than 4096 are fully outside the extent of the tile.  A point at `(1,10)` or `(4095,10)` is within the extent of the tile. A point at `(0,10)` or `(4096,10)` is on the edge of the extent. A point at `(-1,10)` or `(4097,10)` is outside the extent of the tile.
例如，如果一块瓦片的`extent`范围是4096，那么坐标的单位是瓦片长宽的1/4096。坐标0在瓦片的顶部或左边缘，坐标4096在瓦片的底部或右边缘。坐标从1到4095都是在瓦片内部，坐标小于0或者大于4096在瓦片外部。坐标`(1,10)`或`(4095,10)`在瓦片内部。坐标`(0,10)`或`(4096,10)`在瓦片边缘。坐标`(-1,10)`或`(4097,10)`在瓦片外部。

### 4.2. Features
### 4.2. 要素

A feature MUST contain a `geometry` field.
每个要素**必须**包含一个`geometry`字段。

A feature MUST contain a `type` field as described in the Geometry Types section.
每个要素**必须**包含一个`type`字段，该字段将在几何类型章节描述（4.3.4）。

A feature MAY contain a `tags` field. Feature-level metadata, if any, SHOULD be stored in the `tags` field.
每个要素**可以**包含一个`tags`字段。如果存在属于要素级别的元数据，**应该**存储到`tags`字段中。

A feature MAY contain an `id` field. If a feature has an `id` field, the value of the `id` SHOULD be unique among the features of the parent layer.
每个要素**可以**包含一个`id`字段。如果一个要素包含一个`id`字段，那么`id`字段的值**应该**相对于图层中的其他要素是唯一的。

### 4.3. Geometry Encoding
### 4.3. 几何图形编码

Geometry data in a Vector Tile is defined in a screen coordinate system. The upper left corner of the tile (as displayed by default) is the origin of the coordinate system. The X axis is positive to the right, and the Y axis is positive downward. Coordinates within a geometry MUST be integers.
矢量瓦片中的几何数据被定义为屏幕坐标系。瓦片的左上角（显示默认如此）是坐标系的原点。X轴向右为正，Y轴向下为正。几何图形中的坐标**必须**为整数。

A geometry is encoded as a sequence of 32 bit unsigned integers in the `geometry` field of a feature. Each integer is either a `CommandInteger` or a `ParameterInteger`. A decoder interprets these as an ordered series of operations to generate the geometry.
几何图形被编码为要素的`geometry`字段的一个32位无符号型整数序列。每个整数要么是`CommandInteger`或者`ParameterInteger`。解码器解析这些整数序列作为生成几何图形的一系列有序操作。

Commands refer to positions relative to a "cursor", which is a redefinable point. For the first command in a feature, the cursor is at `(0,0)` in the coordinate system. Some commands move the cursor, affecting subsequent commands.
指令涉及到的位置是相对于“游标”的，即一个可重定义的点。对于要素中的第一个指令，游标在坐标系中的位置是`(0,0)`。有些指定能够移动游标，因而会影响到接下来执行的指令。

#### 4.3.1. Command Integers
#### 4.3.1. 指令数

A `CommandInteger` indicates a command to be executed, as a command ID, and the number of times that the command will be executed, as a command count.


A command ID is encoded as an unsigned integer in the least significant 3 bits of the `CommandInteger`, and is in the range 0 through 7, inclusive. A command count is encoded as an unsigned integer in the remaining 29 bits of a `CommandInteger`, and is in the range `0` through `pow(2, 29) - 1`, inclusive.

A command ID, a command count, and a `CommandInteger` are related by these bitwise operations:

```javascript
CommandInteger = (id & 0x7) | (count << 3)
```

```javascript
id = CommandInteger & 0x7
```

```javascript
count = CommandInteger >> 3
```

A command ID specifies one of the following commands:

|  Command     |  Id  | Parameters    | Parameter Count |
| ------------ |:----:| ------------- | --------------- |
| MoveTo       | `1`  | `dX`, `dY`    | 2               |
| LineTo       | `2`  | `dX`, `dY`    | 2               |
| ClosePath    | `7`  | No parameters | 0               |

##### Example Command Integers

| Command   |  ID  | Count | CommandInteger | Binary Representation `[Count][Id]`      |
| --------- |:----:|:-----:|:--------------:|:----------------------------------------:|
| MoveTo    | `1`  | `1`   | `9`            | `[00000000 00000000 0000000 00001][001]` |
| MoveTo    | `1`  | `120` | `961`          | `[00000000 00000000 0000011 11000][001]` |
| LineTo    | `2`  | `1`   | `10`           | `[00000000 00000000 0000000 00001][010]` |
| LineTo    | `2`  | `3`   | `26`           | `[00000000 00000000 0000000 00011][010]` |
| ClosePath | `7`  | `1`   | `15`           | `[00000000 00000000 0000000 00001][111]` |


#### 4.3.2. Parameter Integers

Commands requiring parameters are followed by a `ParameterInteger` for each parameter required by that command. The number of `ParameterIntegers` that follow a `CommandInteger` is equal to the parameter count of a command multiplied by the command count of the `CommandInteger`. For example, a `CommandInteger` with a `MoveTo` command with a command count of 3 will be followed by 6 `ParameterIntegers`.

A `ParameterInteger` is [zigzag](https://developers.google.com/protocol-buffers/docs/encoding#types) encoded so that small negative and positive values are both encoded as small integers. To encode a parameter value to a `ParameterInteger` the following formula is used:

```javascript
ParameterInteger = (value << 1) ^ (value >> 31)
```

Parameter values greater than `pow(2,31) - 1` or less than `-1 * (pow(2,31) - 1)` are not supported.

The following formula is used to decode a `ParameterInteger` to a value:

```javascript
value = ((ParameterInteger >> 1) ^ (-(ParameterInteger & 1)))
```

#### 4.3.3. Command Types

For all descriptions of commands the initial position of the cursor shall be described to be at the coordinates `(cX, cY)` where `cX` is the position of the cursor on the X axis and `cY` is the position of the `cursor` on the Y axis.

##### 4.3.3.1. MoveTo Command

A `MoveTo` command with a command count of `n` MUST be immediately followed by `n` pairs of `ParameterInteger`s. Each pair `(dX, dY)`:

1. Defines the coordinate `(pX, pY)`, where `pX = cX + dX` and `pY = cY + dY`.
   * Within POINT geometries, this coordinate defines a new point.
   * Within LINESTRING geometries, this coordinate defines the starting vertex of a new line.
   * Within POLYGON geometries, this coordinate defines the starting vertex of a new linear ring.
2. Moves the cursor to `(pX, pY)`.

#### 4.3.3.2. LineTo Command

A `LineTo` command with a command count of `n` MUST be immediately followed by `n` pairs of `ParameterInteger`s. Each pair `(dX, dY)`:

1. Defines a segment beginning at the cursor `(cX, cY)` and ending at the coordinate `(pX, pY)`, where `pX = cX + dX` and `pY = cY + dY`.
   * Within LINESTRING geometries, this segment extends the current line.
   * Within POLYGON geometries, this segment extends the current linear ring.
2. Moves the cursor to `(pX, pY)`.

For any pair of `(dX, dY)` the `dX` and `dY` MUST NOT both be `0`.

#### 4.3.3.3. ClosePath Command

A `ClosePath` command MUST have a command count of 1 and no parameters. The command closes the current linear ring of a POLYGON geometry via a line segment beginning at the cursor `(cX, cY)` and ending at the starting vertex of the current linear ring.

This command does not change the cursor position.

#### 4.3.4. Geometry Types

The `geometry` field is described in each feature by the `type` field which must be a value in the enum `GeomType`. The following geometry types are supported:

* UNKNOWN
* POINT
* LINESTRING
* POLYGON

Geometry collections are not supported.

##### 4.3.4.1. Unknown Geometry Type

The specification purposefully leaves an unknown geometry type as an option. This geometry type encodes experimental geometry types that an encoder MAY choose to implement. Decoders MAY ignore any features of this geometry type.

##### 4.3.4.2. Point Geometry Type

The `POINT` geometry type encodes a point or multipoint geometry. The geometry command sequence for a point geometry MUST consist of a single `MoveTo` command with a command count greater than 0.

If the `MoveTo` command for a `POINT` geometry has a command count of 1, then the geometry MUST be interpreted as a single point; otherwise the geometry MUST be interpreted as a multipoint geometry, wherein each pair of `ParameterInteger`s encodes a single point.

##### 4.3.4.3. Linestring Geometry Type

The `LINESTRING` geometry type encodes a linestring or multilinestring geometry. The geometry command sequence for a linestring geometry MUST consist of one or more repetitions of the following sequence: 

1. A `MoveTo` command with a command count of 1
2. A `LineTo` command with a command count greater than 0

If the command sequence for a `LINESTRING` geometry type includes only a single `MoveTo` command then the geometry MUST be interpreted as a single linestring; otherwise the geometry MUST be interpreted as a multilinestring geometry, wherein each `MoveTo` signals the beginning of a new linestring.

##### 4.3.4.4. Polygon Geometry Type

The `POLYGON` geometry type encodes a polygon or multipolygon geometry, each polygon consisting of exactly one exterior ring that contains zero or more interior rings. The geometry command sequence for a polygon consists of one or more repetitions of the following sequence:

1. An `ExteriorRing`
2. Zero or more `InteriorRing`s

Each `ExteriorRing` and `InteriorRing` MUST consist of the following sequence:

1. A `MoveTo` command with a command count of 1
2. A `LineTo` command with a command count greater than 1
3. A `ClosePath` command

An exterior ring is DEFINED as a linear ring having a positive area as calculated by applying the [surveyor's formula](https://en.wikipedia.org/wiki/Shoelace_formula) to the vertices of the polygon in tile coordinates. In the tile coordinate system (with the Y axis positive down and X axis positive to the right) this makes the exterior ring's winding order appear clockwise.

An interior ring is DEFINED as a linear ring having a negative area as calculated by applying the [surveyor's formula](https://en.wikipedia.org/wiki/Shoelace_formula) to the vertices of the polygon in tile coordinates. In the tile coordinate system (with the Y axis positive down and X axis positive to the right) this makes the interior ring's winding order appear counterclockwise.

If the command sequence for a `POLYGON` geometry type includes only a single exterior ring then the geometry MUST be interpreted as a single polygon; otherwise the geometry MUST be interpreted as a multipolygon geometry, wherein each exterior ring signals the beginning of a new polygon. If a polygon has interior rings they MUST be encoded directly after the exterior ring of the polygon to which they belong.

Linear rings MUST be geometric objects that have no anomalous geometric points, such as self-intersection or self-tangency. The position of the cursor before calling the `ClosePath` command of a linear ring SHALL NOT repeat the same position as the first point in the linear ring as this would create a zero-length line segment. A linear ring SHOULD NOT have an area calculated by the surveyor's formula equal to zero, as this would signify a ring with anomalous geometric points.

Polygon geometries MUST NOT have any interior rings that intersect and interior rings MUST be enclosed by the exterior ring.

#### 4.3.5. Example Geometry Encodings

##### 4.3.5.1. Example Point

An example encoding of a point located at:

* (25,7)

This would require a single command:

* MoveTo(+25, +17)

```
Encoded as: [ 9 50 34 ]
              | |  `> Decoded: ((34 >> 1) ^ (-(34 & 1))) = +17
              | `> Decoded: ((50 >> 1) ^ (-(50 & 1))) = +25
              | ===== relative MoveTo(+25, +17) == create point (25,17)
              `> [00001 001] = command id 1 (MoveTo), length 1
```

##### 4.3.5.2. Example Multi Point

An example encoding of two points located at:

* (5,7)
* (3,2)

This would require two commands:

* MoveTo(+5,+7)
* MoveTo(-2,-5)

```
Encoded as: [ 17 10 14 3 9 ]
               |  |  | | `> Decoded: ((9 >> 1) ^ (-(9 & 1))) = -5
               |  |  | `> Decoded: ((3 >> 1) ^ (-(3 & 1))) = -2
               |  |  | === relative MoveTo(-2, -5) == create point (3,2)
               |  |  `> Decoded: ((34 >> 1) ^ (-(34 & 1))) = +7
               |  `> Decoded: ((50 >> 1) ^ (-(50 & 1))) = +5
               | ===== relative MoveTo(+25, +17) == create point (25,17)
               `> [00010 001] = command id 1 (MoveTo), length 2
```

##### 4.3.5.3. Example Linestring

An example encoding of a line with the points:

* (2,2)
* (2,10)
* (10,10)

This would require three commands:

* MoveTo(+2,+2)
* LineTo(+0,+8)
* LineTo(+8,+0)

```
Encoded as: [ 9 4 4 18 0 16 16 0 ]
              |      |      ==== relative LineTo(+8, +0) == Line to Point (10, 10)
              |      | ==== relative LineTo(+0, +8) == Line to Point (2, 10)
              |      `> [00010 010] = command id 2 (LineTo), length 2
              | === relative MoveTo(+2, +2)
              `> [00001 001] = command id 1 (MoveTo), length 1
```

##### 4.3.5.4. Example Multi Linestring

An example encoding of two lines with the points:

* Line 1:
  * (2,2)
  * (2,10)
  * (10,10)
* Line 2:
  * (1,1)
  * (3,5)

This would require the following commands:

* MoveTo(+2,+2)
* LineTo(+0,+8)
* LineTo(+8,+0)
* MoveTo(-9,-9)
* LineTo(+2,+4)

```
Encoded as: [ 9 4 4 18 0 16 16 0 9 17 17 10 4 8 ]
              |      |           |        | === relative LineTo(+2, +4) == Line to Point (3,5)
              |      |           |        `> [00001 010] = command id 2 (LineTo), length 1
              |      |           | ===== relative MoveTo(-9, -9) == Start new line at (1,1)
              |      |           `> [00001 001] = command id 1 (MoveTo), length 1
              |      |      ==== relative LineTo(+8, +0) == Line to Point (10, 10)
              |      | ==== relative LineTo(+0, +8) == Line to Point (2, 10)
              |      `> [00010 010] = command id 2 (LineTo), length 2
              | === relative MoveTo(+2, +2)
              `> [00001 001] = command id 1 (MoveTo), length 1
```

##### 4.3.5.5. Example Polygon

An example encoding of a polygon feature that has the points:

* (3,6)
* (8,12)
* (20,34)
* (3,6) *Path Closing as Last Point*

Would encoded by using the following commands:

* MoveTo(3, 6)
* LineTo(5, 6)
* LineTo(12, 22)
* ClosePath

```
Encoded as: [ 9 6 12 18 10 12 24 44 15 ]
              |       |              `> [00001 111] command id 7 (ClosePath), length 1
              |       |       ===== relative LineTo(+12, +22) == Line to Point (20, 34)
              |       | ===== relative LineTo(+5, +6) == Line to Point (8, 12)
              |       `> [00010 010] = command id 2 (LineTo), length 2
              | ==== relative MoveTo(+3, +6)
              `> [00001 001] = command id 1 (MoveTo), length 1
```

##### 4.3.5.6. Example Multi Polygon

An example of a more complex encoding of two polygons, one with a hole. The position of the points for the polygons are shown below. The winding order of the polygons is VERY important in this example as it signifies the difference between interior rings and a new polygon.

* Polygon 1:
  * Exterior Ring:
    * (0,0)
    * (10,0)
    * (10,10)
    * (0,10)
    * (0,0) *Path Closing as Last Point*
* Polygon 2:
  * Exterior Ring:
    * (11,11)
    * (20,11)
    * (20,20)
    * (11,20)
    * (11,11) *Path Closing as Last Point*
  * Interior Ring:
    * (13,13)
    * (13,17)
    * (17,17)
    * (17,13)
    * (13,13) *Path Closing as Last Point*

This polygon would be encoded with the following set of commands:

* MoveTo(+0,+0)
* LineTo(+10,+0)
* LineTo(+0,+10)
* LineTo(-10,+0) // Cursor at 0,10 after this command
* ClosePath // End of Polygon 1
* MoveTo(+11,+1) // NOTE THAT THIS IS RELATIVE TO LAST LINETO!
* LineTo(+9,+0)
* LineTo(+0,+9)
* LineTo(-9,+0) // Cursor at 11,20 after this command
* ClosePath // This is a new polygon because area is positive!
* MoveTo(+2,-7) // NOTE THAT THIS IS RELATIVE TO LAST LINETO!
* LineTo(+0,+4)
* LineTo(+4,+0)
* LineTo(+0,-4) // Cursor at 17,13
* ClosePath // This is an interior ring because area is negative!

### 4.4. Feature Attributes

Feature attributes are encoded as pairs of integers in the `tag` field of a feature. The first integer in each pair represents the zero-based index of the key in the `keys` set of the `layer` to which the feature belongs. The second integer in each pair represents the zero-based index of the value in the `values` set of the `layer` to which the feature belongs. Every key index MUST be unique within that feature such that no other attribute pair within that feature has the same key index. A feature MUST have an even number of `tag` fields. A feature `tag` field MUST NOT contain a key index or value index greater than or equal to the number of elements in the layer's `keys` or `values` set, respectively.

### 4.5. Example

For example, a GeoJSON feature like:

```json
{
    "type": "FeatureCollection",
    "features": [
        {
            "geometry": {
                "type": "Point",
                "coordinates": [
                    -8247861.1000836585,
                    4970241.327215323
                ]
            },
            "type": "Feature",
            "properties": {
                "hello": "world",
                "h": "world",
                "count": 1.23
            }
        },
        {
            "geometry": {
                "type": "Point",
                "coordinates": [
                    -8247861.1000836585,
                    4970241.327215323
                ]
            },
            "type": "Feature",
            "properties": {
                "hello": "again",
                "count": 2
            }
        }
    ]
}
```

Could be structured like:

```js
layers {
  version: 2
  name: "points"
  features: {
    id: 1
    tags: 0
    tags: 0
    tags: 1
    tags: 0
    tags: 2
    tags: 1
    type: Point
    geometry: 9
    geometry: 2410
    geometry: 3080
  }
  features {
    id: 1
    tags: 0
    tags: 2
    tags: 2
    tags: 3
    type: Point
    geometry: 9
    geometry: 2410
    geometry: 3080
  }
  keys: "hello"
  keys: "h"
  keys: "count"
  values: {
    string_value: "world"
  }
  values: {
    double_value: 1.23
  }
  values: {
    string_value: "again"
  }
  values: {
    int_value: 2
  }
  extent: 4096
}
```

Keep in mind the exact values for the geometry would differ based on the projection and extent of the tile.
