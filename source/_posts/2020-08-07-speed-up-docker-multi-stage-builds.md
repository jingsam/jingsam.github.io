---
title: 加速Docker多阶段构建的方法
date: 2020-08-07 12:18:56
tags:
---



使用多阶段构建（multi-stage build）是减小docker镜像体积很有效的方法，例如拿一个常见的Java应用为例：

```
FROM maven:3.6-jdk-8-alpine
WORKDIR /app
COPY pom.xml .
RUN mvn -e -B dependency:resolve
COPY src ./src
RUN mvn -e -B package
CMD ["java", "-jar", "/app/target/app.jar"]
```

粗看之下似乎没有改进的空间，但是细看之后发现镜像里安装了完整的jdk环境，包含java的sdk和runtime。实际上，运行jar包只需要java runtime，sdk是多余的。

通过使用多阶段构建，将应用的编译环境和运行环境分离，可以极大地减少最终镜像的体积。例如，对将上面的Dockerfile修改为多阶段构建：

```
FROM maven:3.6-jdk-8-alpine AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -e -B dependency:resolve
COPY src ./src
RUN mvn -e -B package

FROM openjdk:8-jre-alpine
COPY --from=builder /app/target/app.jar /
CMD ["java", "-jar", "/app.jar"]
```

以上Dockerfile生成的最终镜像只包含runtime，不再有sdk。

但是凡事皆有两面性，多阶段构建虽然能够减小镜像体积，但是构建的速度慢了许多。原因在于：一是相比于原先的单阶段构建，多了一些构建步骤；二是缓存失效，多阶段编译之后只保留最终镜像的缓存，中间镜像的缓存丢失。其中缓存失效的问题在CI环境中尤为显著。

加快多阶段构建的措施有两项：并行构建和保留缓存。

## 并行构建

如果把多阶段构建中各阶段之间的依赖关系画出来，实际上是一个有向无环图（DAG, Directed Acyclic Graph）。在图中，有些节点之间是没有前后关系的，意味着某些阶段可以并行构建。

从Docker 18.09开始引入了并行构建，启用方法有两种：

1. 临时启用：设置环境变量`DOCKER_BUILDKIT=1`；
2. 默认启用：在`/etc/docker/daemon.json`中设置`{ "features": { "buidkit": true }}`。

## 保留缓存

保留缓存意思是不仅保留最终镜像的缓存，还保留中间镜像的缓存。

`docker build`有两个与缓存相关的参数：`--cache-from`和`BUILDKIT_INLINE_CACHE=1`。`--cache-from`表示可以指定镜像作为缓存源，可以指定多个镜像，指定后会从镜像仓库自动拉取本地不存在的镜像。`BUILDKIT_INLINE_CACHE=1`表示在构建时将缓存的元数据打包到镜像中，使之可以作为缓存源。默认构建的镜像不包含缓存的元数据，不能被`--cache-from`使用。

还是拿文章开头的java应用为例：

首先将编译阶段的镜像进行构建和上传：

```
docker build --build-arg BUILDKIT_INLINE_CACHE=1 \
  --cache-from=builder \
  --target builder \
  --tag builder .
docker push builder
```

然后将最终阶段利用前一阶段和当前阶段的缓存进行构建：

```
docker build --build-arg BUILDKIT_INLINE_CACHE=1 \
  --cache-from=builder \
  --cache-from=app \
  --tag app .
docker push app
```

通过以上过程，就可以在多阶段构建过程中充分利用缓存来加快构建速度。

本文完。