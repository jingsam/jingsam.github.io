---
layout: theme:post
title: "Cygwin 安装 Postgresql"
date: 2016-01-12T12:28:55+08:00
---

最近开始研究 PostgreSQL, 这篇文章主要讲如何在 Cygwin 上安装和使用 PostgreSQL。


### 为什么选择 PostgreSQL？

市面上有各种各样的数据库系统，商业数据库如 Oracle、SQL Server，开源数据库如 SQLite、MySQL、PostgreSQL，为什么从众多的数据库中我选择了 PostgreSQL 呢？

开源且免费是我考虑的首要因素，其次是要支持多用户并发的大型数据库系统，最后数据库要成熟稳定。满足要求的就只有 MySQL 和 PostgreSQL 了， 两者之间的差别可以从它们首页的标题可以一窥大概：

 > The world's most popular open source database.   (MySQL)  
 > The world's most advanced open source database.  (PostgreSQL)

MySQL 是最流行的数据库系统，说明用的人最多；PostgreSQL 是最先进的数据库系统，说明技术更先进。

MySQL 自从被 Oracle 收购之后，也沾染了些大公司的习气，变得趋于保守。MySQL 社区的一部分成员担心 MySQL 会被大公司主导而偏离了方向，因而 fork 出了一个新的数据库 MariaDB。MariaDB 虽然由社区主导，开发更为激进，但是生态环境远不足 MySQL 丰富。MySQL 的命运与同被 Oracle 收购的 OpenOffice 极其相似，我对它的未来持怀疑态度。 同时，我也感慨 Oracle 公司不要再将魔爪伸向开源社区了，手下留情啊！

PostgreSQL 这边完全是另一番风景，它完全由社区主导，对新技术更为开放。更重要的是，PostgreSQL 对空间数据的支持是最好的，借助 PostGIS 扩展我们可以直接用 SQL 语句进行空间查询。考虑到我后续的工作都会与空间数据有关，所以 PostgreSQL 是我的不二选择。


### 为什么要在 Cygwin 上安装 PostgreSQL?

PostgreSQL 在 Windows 上有原生的安装包，那为什么还要大费周章地在 Cygwin 上安装呢？原因是 Cygwin 提供了与 Linux 一致的开发环境，将来我的程序部署到 Linux 上时更为简单方便。

PostgreSQL 可以直接从 Cygwin 的软件仓库中安装。一种方式是通过 Cygwin 的安装程序安装，另一种方式是通过包管理工具安装。后一种方式需要先安装 apt-cyg，安装的方法可以从我的另一篇博文中找到。安装好 apt-cyg 之后，运行以下命令：
```
apt-cyg install postgresql
```

接下来运行 `cygserver-config`，使 Cygwin 新建一个 Windows 服务。必须注意的是，这条命令必须在管理员权限下运行，因此 Cygwin 命令窗口需以管理员身份打开。然后运行以下命令开启服务：
```
cygrunsrv -S cygserver
```

### 配置 PostgreSQL

首次使用 PostgreSQL 数据库，需要运行以下命令完成数据库初始化：
```
initdb -D /var/lib/postgresql/data
```
其中参数 `-D` 指定数据库文件的存储位置

然后使用 `pg_ctl` 来开启数据库服务，命令如下：
```
pg_ctl start -D /var/lib/postgresql/data -l /var/log/postgresql.log
```
其中参数`-l`指定日志文件的位置。

接下来我们就可以新建一个数据库：
```
createdb mydb
```

通过以下命令连接到数据库控制台：
```
psql mydb
```


### 关闭 PostgreSQL 数据库

当我们使用完数据库之后，不希望数据库服务占用系统资源，因此需要关闭数据相关服务。

同样使用 `pg_ctl` 来关闭数据库服务，命令如下：
```
pg_ctl stop -D /var/lib/postgresql/data
```

最后终止 cygserver：
```
cygrunsrv -E cygserver
```

### 总结

以下是本文安装与配置 PostgreSQL 时，所有涉及到的命令：
```bash
// 安装
apt-cyg install postgresql

// 初始化 cygserver
cygserver-config

// 启动 cygserver
cygrunsrv -S cygserver

// 初始化数据库
initdb -D /var/lib/postgresql/data

// 开启数据库服务
pg_ctl start -D /var/lib/postgresql/data -l /var/log/postgresql.log

// 新建数据库
createdb mydb

// 连接数据库
psql mydb

// 关闭数据库服务
pg_ctl stop -D /var/lib/postgresql/data

// 终止 cygserver
cygrunsrv -E cygserver
```