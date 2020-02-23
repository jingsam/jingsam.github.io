---
title: Docker容器访问宿主机网络
date: 2018-10-16 10:26:26
tags:
---

# 缘起

最近部署一套系统，使用nginx作反向代理，其中nginx是使用docker方式运行：

```
$ docker run -d --name nginx $PWD:/etc/nginx -p 80:80 -p 443:443 nginx:1.15
```

需要代理的API服务运行在宿主机的`1234`端口，`nginx.conf`相关配置如下：

```
server {
  ...

  location /api {
    proxy_pass http://localhost:1234
  }
  ...
}
```

结果访问的时候发现老是报`502 Bad Gateway`错误，错误日志显示无法连接到upstream。

仔细想一想，`nginx.conf`中的`localhost`似乎有问题。由于nginx是运行在docker容器中的，这个`localhost`是容器的localhost，而不是宿主机的localhost。

到这里，就出现了本文要解决的问题：如何从容器中访问到宿主机的网络？通过搜索网络，有如下几种方法：


# 使用宿主机IP

在安装Docker的时候，会在宿主机安装一个虚拟网卡`docker0`，我们可以使用宿主机在`docker0`上的IP地址来代替`localhost`。

首先，使用如下命令查询宿主机IP地址：

```
$ ip addr show docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:d5:4c:f2:1e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:d5ff:fe4c:f21e/64 scope link
       valid_lft forever preferred_lft forever
```

可以发现宿主机的IP是`172.17.0.1`，那么将`proxy_pass http://localhost:1234`改为`proxy_pass http://172.17.0.1:1234`就可以解决`502 Bad Gateway`错误。

macOS下并没有`docker0`虚拟网卡，宿主机IP默认是`192.168.65.1`，也可以使用`host.docker.internal`这个特殊的DNS名称来解析宿主机IP。

由此发现，不同系统下宿主机的IP是不同的，所以使用宿主机IP，不能跨环境通用。


# 使用host网络

Docker容器运行的时候有`host`、`bridge`、`none`三种网络可供配置。默认是`bridge`，即桥接网络，以桥接模式连接到宿主机；`host`是宿主网络，即与宿主机共用网络；`none`则表示无网络，容器将无法联网。

当容器使用`host`网络时，容器与宿主共用网络，这样就能在容器中访问宿主机网络，那么容器的`localhost`就是宿主机的`localhost`。

在docker中使用`--network host`来为容器配置`host`网络：

```
$ docker run -d --name nginx --network host nginx
```

上面的命令中，没有必要像前面一样使用`-p 80:80 -p 443:443`来映射端口，是因为本身与宿主机共用了网络，容器中暴露端口等同于宿主机暴露端口。

使用host网络不需要修改`nginx.conf`，仍然可以使用`localhost`，因而通用性比上一种方法好。但是，由于`host`网络没有`bridge`网络的隔离性好，使用`host`网络安全性不如`bridge`高。


# 总结

本文提出了使用宿主机IP和使用host网络两种方法，来实现从容器中访问宿主机的网络。两种方法各有优劣，使用宿主机IP隔离性更好，但通用性不好；使用host网络，通用性好，但带来了暴露宿主网络的风险。
