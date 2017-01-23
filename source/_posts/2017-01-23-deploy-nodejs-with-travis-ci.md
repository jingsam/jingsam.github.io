---
title: 利用travis-ci持续部署nodejs应用
date: 2017-01-23 11:26:23
tags:
---

Travis-ci是一款持续集成（Continuous Integration）服务，它能够很好地与Github结合，每当代码更新时自动地触发集成过程。

Travis-ci配置简单，很多nodejs项目都用它做自动测试。然而，对于持续集成，仅做到自动测试是不够的，还要有后续的自动部署，才能完成“提交代码 => 自动测试 => 自动部署”的集成链条。

本文以nodejs应用为例，来谈谈如何利用travis-ci完成自动部署。

## 基本原理

从自动测试到自动部署的核心问题是测试机与生产服务器的信任问题，即如何安全地把程序包传输到生产服务器。市面上的部署工具如scp、ansible、chef，都绕不开这个核心问题。

以scp为例，测试机登录生产服务器的方式有两种：密码和秘钥。密码登录方式需要交互式地输入密码，总不能每次测试的时候，人为地输入密码吧，所以密码方式行不通。

秘钥的方式可以实现自动登录，但首次将测试机的公钥传输给生产服务器仍然需要密码。似乎走入了死胡同，但办法总是有的。我们知道开发机是可以登录到生产服务器的，那么我们就可以**将开发机的公钥复制到生产服务器，将开发机的私钥复制到测试机，测试机通过私钥来伪装成开发机，自动地登录到生产服务器**。

解决了自动登录的问题，另一个问题是怎么将开发机的私钥复制到测试机上。由于测试机每次都是新开的一个虚拟机，这个新开的虚拟机IP不固定，所以没办法直接登录上去。解决办法是将私钥文件作为代码库的一部分提交，这样测试机每次从代码库上拉取代码的同时也获取到了秘钥文件，通过这种方式就实现了私钥从开发机复制到测试机。

将私钥文件提交到代码库有一个很严重的安全性问题，即任何人只要得到了这个私钥文件，他就可以随心所欲的操纵生产服务器。幸好，travis-ci提供了加密方案，它能够将私钥文件加密，加密后的文件只在当前代码库有效。

总的来说，通过复制私钥完成自动登录以及对私钥加密来保障安全性，我们就可以建立起测试机与生产服务器的信任通道，测试机就可以安全地操作生产服务器完成自动部署。

## 配置

现在我以scp方式部署nodejs应用为例，来说明travis-ci做自动部署的配置。

首先，建立起开发机与生产服务器的信任关系：
```
ssh-copy-id username@host
```

然后，加密你的私钥，私钥文件通常在`~/.ssh/id_rsa`。加密私钥文件需要使用travis这个命令行工具，它是一个ruby包，使用gem安装：
```bash
gem install travis
travis login
```

输入账号密码登录成功后，使用`travis encrypt-file`加密：
```
travis encrypt-file ~/.ssh/id_rsa --add
```

上面命令执行完后，会生成一段解密命令并添加到`.travis.yml`中：
```yml
before_install:
  - openssl aes-256-cbc -K $encrypted_830d3b21a25d_key -iv $encrypted_830d3b21a25d_iv
    -in ~/.ssh/id_rsa.enc -out ~/.ssh/id_rsa -d
```

接下来，把加密后的私钥文件（id_rsa.enc）复制到代码库中，千万要注意不要错把未加密的私钥文件（id_rsa）复制到你的代码库中。然后把上面的解密命令的`-in ~/.ssh/id_rsa.enc`改为`-in id_rsa.enc`。

通过上面的过程就基本建立测试机与生产服务器的信任关系，但还有一些小细节要处理。例如，降低`id_rsa`文件的权限，否则ssh处于安全方面的原因会拒绝读取秘钥；将生产服务器地址加入到测试机的信任列表中，否则连接时会询问是否信任服务器。更改后的配置如下:
```yml
before_install:
  - openssl aes-256-cbc -K $encrypted_830d3b21a25d_key -iv $encrypted_830d3b21a25d_iv
    -in id_rsa.enc -out ~/.ssh/id_rsa -d
  - chmod 600 ~/.ssh/id_rsa
  - echo -e "Host 102.201.64.94\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
```

最后，测试机就可以愉快地操作生产服务器了，例如下面是一个nodejs应用的`.travis.yml`文件配置：
```yml
language: node_js
node_js:
  - '4.4.4'
before_install:
  - openssl aes-256-cbc -K $encrypted_830d3b21a25d_key -iv $encrypted_830d3b21a25d_iv
    -in id_rsa.enc -out ~/.ssh/id_rsa -d
  - chmod 600 ~/.ssh/id_rsa
  - echo -e "Host 102.201.64.94\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
script:
  - npm run test
after_success:
  - npm prune --production  # 删除devDependencies
  - tar -jcf indoor-server.tar.bz2 *    # 打包并压缩代码
  - scp indoor-server.tar.bz2 jingsam@102.201.64.94:~/  # 复制到生产服务器上
  - ssh jingsam@102.201.64.94 'mkdir -p indoor-server && tar -jxf indoor-server.tar.bz2 -C indoor-server'   # 解压
  - ssh jingsam@102.201.64.94 'cd indoor-server && pm2 startOrReload pm2.json'  # 重启pm2
```

## 总结

本篇文章讲的自动部署其实与nodejs关系不大，完全适用于各种语言的自动部署，其原理都是相通的。
