---
layout: theme:post
title: "CentOS 安装 Nodejs"
date: 2015-12-01T12:46:51+08:00
---

最近实验室的一台服务器上需要安装 nodejs，记录下折腾的过程。如果没时间看我废话，直接跳到**总结**部分即可找到解决方案。

## 修复 yum 源

安装 nodejs，理所当然地认为只要执行下面的命令就好了。
```
yum install nodejs
```
结果提示：
```bash
Loaded plugins: fastestmirror, refresh-packagekit
You need to be root to perform this command.
```
这个简单，加上`sudo`执行：
```
sudo yum install nodejs
```
结果还是有错误，仔细一看，发现里面有一条关键提示：
```
http://mirrors.163.com/centos/%24releasever/os/x86_64/repodata/repomd.xml: [Errno 14] PYCURL ERROR 22 - "The requested URL returned error: 404 Not Found"
```
`%24releasever`是个什么鬼？Google 了一下发现是系统版本号变量`$releasever`。`$releasever`本应该被替换为 CentOS 的版本号 6.1 的，不知道为什么没有获取到。`$releasever`变量是从`/etc/yum.conf`的`distroverpkg`获取到, 我的`distroverpkg=centos-release`。改成`distroverpkg=redhat-release`后，`$releasever`变为`6Server`，再执行`yum install`仍然不行。

看来该换一种思路，分析问题发生的原因，即是`repomd.xml`文件获取不到。通过到网站上查询，发现这个文件存在于
```
http://mirrors.163.com/centos/6/os/x86_64/repodata/repomd.xml
```
因此，我就直接更改`/etc/yum.repos.d/CentOS6-Base-163.repo`文件，将里面的`$releasever`替换为 6，然后执行：
```bash
sudo yum clean all
sudo yum makecache
```
一切顺利，yum 源修复成功。虽然直接修改`$releasever`不是很优雅，只要能运行就行，不折腾了。


## 添加 EPEL 源

再次尝试`sudo yum install nodejs`，提示：
```
No package node available.
```
原来 CentOS 的官方源并没有 Nodejs 的安装包，安装需要添加 EPEL 源，执行以下命令安装：
```bash
sudo rpm -ivh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
```
最后执行以下命令，终于成功安装 Nodejs.
```
sudo yum -y install nodejs npm --enablerepo=epel
```


## 总结

Nodejs 不存在官方的源中，所以安装需要添加 EPEL 源后再安装，完整的命令如下：
```bash
sudo rpm -ivh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
sudo yum -y install nodejs npm --enablerepo=epel
```
