---
title: 利用ssh反向穿透实现内网服务器自动部署
date: 2019-06-14 14:36:17
tags:
---

当前持续集成的流行，大大提高了开发迭代的速度。自动化部署作为持续集成的最后一公里，对于完成整个闭环至关重要。如果生产服务器部署至公网且开通了ssh端口，那么测试服务器完成测试打包后，可以很方便地利用ssh将部署包推送到生产服务器上完成部署。但是，如果生产服务器部署在内网，测试服务器部署在外网，那么测试服务器就无法得知生产服务器的IP，无法利用ssh远程登录实现自动部署。

本文利用ssh的反向穿透技术，来实现内网服务器的自动部署。

## 技术原理

ssh一般用来客户端远程登录到服务器上，而ssh反向穿透“反其道而行之”，由服务端主动发起请求连接客户端，然后在客户端打开一个端口，之后发往客户端的数据包将会转发到服务端。例如在服务端执行：

```bash
ssh -NR 22022:localhost:22 clientUser@clientMachine
```

连接成功后，发往客户端`22022`的数据包将被转发到服务端的`22`端口上。这时，如果客户端具有公网IP，那么我们就可以利用客户端机器作为跳板，远程登录到内网服务器上，命令如下：

```bash
ssh -p 22022 serverUser@clientMachine
```

通过ssh的反向穿透技术，让内网服务器借用公网客户机的IP，实现内网服务器公网可见。在此基础上，实现内网服务器自动部署的原理如下图：

{% asset_img ssh-reverse-proxy.png %}


上述原理很简单，但具体操作起来，还是有许多细节要考虑，下面对此一一说明。


## 持久化连接

通过ssh在第①步建立了生产服务器`prodServer`与跳板机`jumpServer`的连接隧道，但是这条隧道如果长时间没有数据包传输，那么ssh会主动关闭连接，之后测试服务器`testServer`就不能再通过跳板机连接到生产服务器。

为了保持生产服务器与跳板机的连接隧道不断开，需要ssh自动重连。ssh不支持自动重连功能，幸好有autossh帮我们完成这项工作。

autossh不属于系统自带工具，我们需要在生产服务器安装它，Ubuntu上的安装命令如下：

```bash
sudo apt-get install autossh
```

安装好之后，执行以下命令：

```bash
autossh -M 0 -f -o "ServerAliveInterval=30" -o "ServerAliveCountMax=3" -o "ExitOnForwardFailure=yes" -NR 22022:localhost:22 jump@jumpServer
```

上述命令中，除了`-M 0`和`-f`是autossh的参数外，其他参数都原样传递给ssh，其中`-M 0`表示不另开端口监测ssh，`-f`表示后台运行。auotssh以前是靠`-M`另开一个端口发送心跳数据包，由于新版ssh（protocol 2）内建了心跳功能，所以不再推荐另开端口。上面的命令使用`ServerAliveInterval`和`ServerAliveCountMax`两个参数，表示客户端向服务端每30秒发送一次心跳数据包，如果发3次还没响应，那么断开连接。我们也可以在服务端的`/etc/ssh/sshd_config`配置文件中添加`ClientAliveInterval 30`和`ClientAliveCountMax 3`参数后，重启sshd，表示由服务端向客户端发送心跳数据包。`ExitOnForwardFailure`表示ssh转发失败后，关闭连接并退出，这样autossh才能监测到错误并重启ssh连接。

执行上述命令后，我们只能在`jumpServer`上登录到`prodServer`。如果我们想要从任意机器上远程登录到`prodServer`，需要在`jumpServer`的`/etc/ssh/sshd_config`中添加`GatewayPorts yes`参数并重启sshd，之后执行：

```bash
ssh -p 22022 prod@jumpServer
```

如果`prodServer`重启后，我们希望`autossh`也能随系统启动，此时需要将`autossh`添加到启动项中，以下是以systemd为例，新建一个`/lib/systemd/system/autossh.service`配置文件来添加autossh启动项：

```
[Unit]
Description=autossh
Wants=network-online.target sshd.service
After=network-online.target sshd.service

[Service]
Type=simple
User=geoeye
Environment="AUTOSSH_GATETIME=0"
ExecStart=/usr/bin/autossh -M 0 -o "ServerAliveInterval=30" -o "ServerAliveCountMax=3" -NR 22022:localhost:22 jingsam@www.foxgis.com
ExecStop=/bin/kill $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

注意上面启动autossh的命令中，要把`-f`选项去掉，并且autossh使用绝对路径。

最后执行以下命令执行来启动autossh：

```bash
sudo systemctl enable autossh
sudo systemctl start autossh
```


## 免密登录

不管是`prodServer`在系统启动时反向连接到`jumpServer`，还是`testServer`完成自动测试后推送部署包到`prodServer`，这些过程都是后台自动执行，我们没有机会去输入登录密码。因此，我们需要在三者之间配置ssh key，实现免密登录。

首先是`prodServer`连接到`jumpServer`，这一步比较简单，只需在`prodServer`使用`ssh-copy-id`命令将公钥复制到`jumpServer`即可。

然后是`testServer`连接到`prodServer`，这一步稍微有点复杂，主要是因为`testServer`一般是动态创建的虚拟机或容器，测试完就删除了，所以没办法提前将`testServer`的公钥复制到`prodServer`。解决的思路是使用任意机器登录一次`prodServer`并将该机器的公钥复制到`prodServer`，然后将该该机器的私钥复制到`testServer`，让`testServer`伪装成为该机器登录`prodServer`。Gitlab、Travis-ci、Circle CI各自有不同的方法安全地传输ssh key，具体参考相应的文档。Gitlab可以在project和group中的“设置->CI/CD->Variables”中设置环境变量，如下图：

{% asset_img gitlab-variables.png %}

设置好之后，配置`.gitlab-ci.yml`：

```yml
deploy:
  stage: deploy
  before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add - > /dev/null
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan www.foxgis.com >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
  script:
    - ssh -p 22022 prod@jumpServer # 这里写具体的部署逻辑
  only:
    - master
```


## 总结

本文借助ssh反向穿透技术，实现了内网服务器与测试服务器的互通，并且利用autossh解决了ssh持久连接的问题，以及利用复制私钥来实现测试服务器到内网服务器的免密登录。最终，结合这些技术，实现了内网服务的自动化部署。

在实际测试过程中，ssh的反向穿透连接速度慢而且不是很稳定，下一步研究webhook技术来实现自动部署。