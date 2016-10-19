---
title: Nodejs网站部署到DigtialOcean
date: 2016-04-20 17:27:21
tags:
---

最近基于Express + Mongodb开发了一个纯API服务，现在要部署到公网上。我选择的是DigitialOcean的主机，但整个部署过程不仅仅适用于DigitialOcean，其他VPS也可参考。

## 创建主机
假设我们现在已经注册了DigitialOcean账号，并且通过了邮箱验证和信用卡绑定。
首先创建一个主机，登录后点击`Create Droplets`，即可开始创建主机。Droplets的意思是小水滴，DigitialOcean是数字海洋，两者之间的关系还是蛮形象的。
在创建主机界面，我们可以选择主机的操作系统、服务器的位置、服务器的硬件配置等。我选择的是Ubuntu 15.10，San Franscisco的机房，以及最便宜5美元一个月的配置：1 CPU + 512M RAM + 20G SSD。
选择完配置之后，先别忙着点确定。里面有一项叫SSH key，可以将你的SSH的公钥填进去，这样我们稍后登录主机的时候就不需要填密码了。当然，如果你不知道怎么生成SSH key或者嫌麻烦，也可以通过密码登录，密码会在你创建完主机之后发到你的注册邮箱。

## 创建用户
当主机创建完之后，我们可以通过SSH远程登录到主机上：
```bash
ssh root@xxx.xxx.xxx.xxx
```
这里有个问题就是，我们是以root身份登录到系统中，而我们网站不能在root权限下部署。否则，要是有黑客获取了网站控制权限，就掌握了主机的控制权限，这是很危险的。所以，我们需要创建一个普通权限的用户，以普通用户的身份部署网站。

使用下面的命令添加新用户和密码，其中`-m`表示在`/home`目录下创建用户文件夹。
```bash
add user -m judy
passwd judy
```

为了之后安装软件方便，我们将新用户添加到`sudo`组中：
```bash
sudo adduser judy sudo
```

现在我们完成了新用户的创建。有些人在使用SSH登录到DigitalOcean后会发现很卡，打个命令半天才回显，在这里我推荐一款很好用的SSH替代软件：mosh。SSH的机制是我们输入一个字符，这个字符会发送到远程服务器，然后远程服务器收到后再发送回终端显示。在这个传输过程中，如果网络条件不好，会产生明显地卡顿。而mosh在你输入一个字符时，立即在终端上显示，然后再后台自动地将命令发送到远程服务器。因此即使在较差的网络条件下，mosh也可以很流畅。

mosh需要在客户端和服务端同时安装，安装的的命令如下：
```
apt-get install mosh
```

退出root用户，然后用mosh重连，你将会发现比之前流畅多了，mosh连接的命令和SSH一样：
```
mosh judy@xxx.xxx.xxx.xxx
```

## 安装nodejs

由于我们的网站是基于Express的，因此需要安装nodejs。Ubuntu的官方源上没有nodejs，需要添加nodesource源，安装的命令如下：
```bash
sudo apt-get install curl
curl -sL https://deb.nodesource.com/setup_5.x | sudo -E bash -
sudo apt-get install nodejs
```
`sudo`命令是临时借用root的权限去执行命令。

## 安装mongodb和git

网站的数据库采用的是mongodb，幸运的是Ubuntu的官方源收录了mongodb，所以我们可以直接用apt安装：
```bash
sudo apt-get install mongodb
```
mongodb安装完之后会自动启动守护程序mongod，并设置了开机自启动。检查mongodb是否成功启动，可以运行`mongo`连接数据是否成功。

网站的源代码托管在GitHub上，所以要把代码下载下来，我们还需要安装git：
```bash
sudo apt-get install git
```

## 部署

通过git将网站源代码拉取下来，并安装相应的依赖：
```
git clone https://github.com/{your_git_repo} site
cd site
npm install
```

网站的软件环境基本上准备好了，如果代码有测试的话，可以跑一遍测试看网站是否各项功能正常。

Express网站开发的时候，默认的服务端口是3000。由于服务是公开发布的，总不能让用户还在网址后面加个端口号吧。所以，我们将网站切换到80端口：
```bash
PORT=80 node ./bin/www
```

不幸的是，你将马上看到出错信息，信息显示80端口只有root用户才能使用。实际上，任何小于1024的端口，都只能由root用户使用。所以我们用采取一定的措施，使得网站服务可以使用80端口。方法很简单，我们可以借助cap2实现：
```bash
sudo apt-get install libcap2-bin
sudo setcap cap_net_bind_service=+ep `which nodejs`
```
后一条命令中的`which nodejs`是找出nodejs可执行文件的路径，一定要是nodejs的实际路径，不能是符号链接。

执行以上命令后，再次测试就会发现能够使用80端口了。


## 更改时区
新建的主机默认的时区是0时区，更改为北京时间可用交互式的命令行工具`tzselect`:
```
tzselect
sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 总结
通过以上过程，基本上完成了对服务器的配置。但是还有很多工作要做，例如nodejs进程自动重启、网站制动部署等，这将在以后的博文中逐步说明。
