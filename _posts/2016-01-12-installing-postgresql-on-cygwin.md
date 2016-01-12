---
layout: theme:post
title: "Cygwin 安装 Postgresql"
date: 2016-01-12T12:28:55+08:00
---

最近开始研究 PostgreSQL, 这篇文章主要讲如何在 Cygwin 上安装和使用 PostgreSQL。


### 为什么选择 PostgreSQL？

市面上有各种各样的数据库系统，商业数据库如 Oracle、SQL Server，开源数据库如 SQLite、MySQL、PostgreSQL，为什么从众多的数据库中我选择了 PostgreSQL 呢？

开源且免费是我考虑的首要因素，其次是要支持多用户并发的大型数据库系统，最后数据库要成熟稳定。满足要求的就只有 MySQL 和 PostgreSQL 了， 两者之前的差别可以从它们首页的标题可以一窥大概：

 > The world's most popular open source database.   (MySQL)  
 > The world's most advanced open source database.  (PostgreSQL)

MySQL 是最流行的数据库系统，说明用的人最多；PostgreSQL 是最先进的数据库系统，说明技术更先进。

MySQL 自从被 Oracle 收购之后，也沾染了些大公司的习气，变得趋于保守。MySQL 社区的一部分成员担心 MySQL 会被大公司主导而偏离了方向，因而 fork 出了一个新的数据库 MariaDB。MariaDB 虽然由社区主导，开发更为激进，但是生态环境远不足 MySQL 丰富。MySQL 的命运与同被 Oracle 收购的 OpenOffice 及其相似，我对它的未来持怀疑态度。 同时，我也感慨 Oracle 公司不要再将魔爪伸向开源社区了，手下留情啊！

PostgreSQL 这边完全是另一番风景，它完全由社区主导，对新技术更为开放。更重要的是，PostgreSQL 对空间数据的支持是最好的，借助 PostGIS 扩展我们可以直接用 SQL 语句进行空间查询。考虑到我后续的工作都会与空间数据有关，所以 PostgreSQL 是我的不二选择。


### 为什么要在 Cygwin 上安装 PostgreSQL?

PostgreSQL 在 Windows 上有原生的安装包，那为什么还要大费周章地在 Cygwin 上安装呢？原因是 Cygwin 提供了与 Linux 一致的开发环境，将来我的程序部署到 Linux 上时更为简单方便。


