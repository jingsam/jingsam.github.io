---
title: 简单搞定Nginx日志分割
date: 2019-01-15 18:32:52
tags:
---

nginx日志分割是很常见的运维工作，关于这方面的文章也很多，通常无外乎两种做法：一是采用cron定期执行shell脚本对日志文件进行归档；二是使用专门日志归档工作logrotate。

第一种写shell脚本的方法用得不多，毕竟太原始。相比之下，使用logrotate则要省心得多，配置logrotate很简单。关于如何配置logrotate不是本文要讲的内容，感兴趣的话可以自行搜索。

虽然大多数Linux发行版都自带了logrotate，但在有些情况下不见得安装了logrotate，比如nginx的docker镜像、较老版本的Linux发行版。虽然我们可以使用包管理器安装logrotate，但前提是服务器能够访问互联网，企业内部的服务器可不一定能够联网。

其实我们有更简单的方法，从nginx 0.7.6版本开始`access_log`的路径配置可以包含变量，我们可以利用这个特性来实现日志分割。例如，我们想按天来分割日志，那么我们可以这样配置：

```
access_log logs/access-$logdate.log main;
```

那么接下来的问题是我们怎么提取出`$logdate`这个变量？网上有建议使用下面的方法：

```
if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})") {
  set $year $1;
  set $month $2;
  set $day $3;
}

access_log logs/access-$year-$month-$day.log main;
```

上面的方法有两个问题：一是如果`if`条件不成立，那么`$year`、`$month`和`$month`这三个变量将不会被设置，那么日志将会记录到`access-$year-$month-$day.log`这个文件中；二是`if`只能出现在`server`和`location`块中，而`access_log`通常会配置到顶层的`http`块中，这时候`if`就不适用。

如果要在`http`块中设置`access_log`，更好的方法是使用`map`指令：

```
map $time_iso8601 $logdate {
  '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
  default                       'date-not-found';
}

access_log logs/access-$logdate.log main;
```

`map`指令通过设置默认值，保证`$logdate`始终有值，并且可以出现在`http`块中，完美地解决了`if`指令的问题。

最后，为了提高日志的效率，建议配置`open_log_file_cache`，完整的日志分割配置如下：

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

map $time_iso8601 $logdate {
  '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
  default                       'date-not-found';
}

access_log logs/access-$logdate.log main;
open_log_file_cache max=10;
```

本文完。
