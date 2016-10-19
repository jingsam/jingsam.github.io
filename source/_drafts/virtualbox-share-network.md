---
title: VirtualBox共享Windows网络
tags:
---

1. 网卡1：仅主机（Host-Only）适配器，网卡2：网络地址转换

2. sudo vim /etc/network/interfaces
```
auto eth1
iface eth1 inet dhcp
```

3.sudo ifup eth1或重启
