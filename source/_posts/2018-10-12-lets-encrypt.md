---
title: 手动挡模式申请Let's Encrypt通配符证书
date: 2018-10-12 12:00:00
tags:
---

本篇文章要讲的内容是申请Let's Encrypt通配符证书，但是标题中加一个“手动挡”模式是什么意思呢？我们拿学车为例，当我们学会了开手动挡，开自动挡自然不在话下。同理，如果我们弄明白了手动申请Let's Encrypt证书的步骤，以后使用自动化工具自然也是手到擒来。

网上关于申请Let's Encrypt证书的文章多如牛毛，本篇文章的不同在于以下几点：

1. 使用手动模式申请证书；
2. 申请的是通配符证书，一个证书适用多个域名；
3. 使用Docker来完成申请操作，避免安装一大堆软件污染系统；
4. 可以在本地电脑上完成申请操作，更灵活。


# 验证域名的三种模式

申请证书要解决的一个关键问题是：如何证明域名是你所拥有的？Let's Encrypt提供了三种模式：http、dns、tls-sni。

http模式就是Let's Encrypt给你一个随机字符串，你需要在Web服务器的`/.well-known/acme-challenge/`服务路径下放置一个以该字符串命名的文件，当Let's Encrypt能够访问到这个文件时，证明你是这个域名的所有者。

dns模式同样是Let's Encrypt给你一个随机字符串，你需要以该字符串建立一个DNS TXT记录，当Let's Encrypt查询域名的TXT记录时，发现得到的字符串一致，则证明你是这个域名的所有者。

tls-sni模式没有仔细研究过，似乎通过SSL加密传输。http走的是80端口，tls-sni走的是443端口。由于当前的tls-sni似乎有漏洞，该模式一度遭禁用，所以不推荐采用此模式，本文也不加以讨论。

http模式和dns模式各有优劣，适用于不同的场景。http模式不需要操作DNS记录，只需要新建一个文件就可以完成验证，带来的限制就是验证过程必须操作服务器；dns模式不需要操作服务器，只需添加DNS TXT记录就行，缺点是必须登录到域名提供商的页面上修改DNS记录。不管是http模式还是dns模式，申请证书的操作都是可以在任意电脑上完成，申请完之后再将证书复制到服务器上。http模式验证过程需要操作服务器，dns模式验证过程需要操作DNS。

由于我们要申请的是通配符证书，必须使用dns模式验证，为什么呢？以通配符域名`*.example.com`为例，它包含的域名千千万万，不太可能使用http模式去验证每个域名。dns模式的验证能力更强，如果用户具有操作`example.com`域名的DNS记录权限，那当然是拥有通配符域名`*.example.com`。

搞清楚申请域名关键问题，接下来说明申请域名的步骤。


# 申请域名的步骤

申请let's encrypt的客户端软件很多，这里采用的是官方推荐的客户端certbot。运行certbot需要python环境，还需要在系统全局环境安装一些python包。考虑到证书的有效期是90天，申请证书并不是一个频繁的操作，我不想因为一个低频使用的软件污染我的全局系统，因此采用Docker作为运行环境。

申请let's encrypt只需要一条命令，之后certbot通过交互的方式询问申请信息，很人性化：

```
$ docker run -it --rm --name certbot \
    -v $PWD:/etc/letsencrypt \
    certbot/certbot:v0.27.1 \
    certonly --manual --preferred-challenges=dns-01 \
    --server=https://acme-v02.api.letsencrypt.org/directory
```

上述命令中，`-v $PWD:/etc/letsencrypt`表示把当前文件夹映射到docker中的`/etc/letsencrypt`文件夹，这样certbot生成的证书将出现在当前文件夹中；`certonly`表示证书申请子命令，还有`renew`、`revoke`、`delete`等其他子命令；`--manual`是手动申请模式；`--preferred-challenges=dns-01`表示采用dns模式验证，默认采用http模式验证；`--server=https://acme-v02.api.letsencrypt.org/directory`表示指向ACME V2版本服务器，默认指向ACME V1版本服务器，但只有ACME V2才支持通配符证书。

接着需要提供一个email地址用于接收证书更新和安全问题提醒：

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): XXX@XX.com
```

需要同意用户协议：

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A
```

询问你需不需要接收电子前哨基金会（EFF）的新闻、活动，可以选择不接收：

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
```

接下来就需要填写要申请证书的域名，这里需要注意一点的是`*.example.com`并不包括裸域名`example.com`，如果申请的证书需要囊括`example.com`，就必须要同时包含`example.com`和`*.example.com`。

```
Please enter in your domain name(s) (comma and/or space separated)  (Enter 'c'
to cancel): example.com, *.example.com
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for example.com
dns-01 challenge for example.com
```

申请证书的电脑的IP会被记录，可能是防治滥用吧，需要同意记录IP：

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
```

然后开始使用dns模式验证域名，这时候需要登录到你的域名服务商的DNS编辑页面上，在`_acme-challenge.example.com`增加两个TXT记录，一个用来验证`example.com`，另一个用来验证`*.example.com`。TXT记录的内容就是命令行中提供的随机字符串：

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.example.com with the following value:

FvMdm......6scC4

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.example.com with the following value:

H1UHs......Lc90en4dY

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

TXT记录修改好之后，使用`dig -t TXT _acme-challenge.example.com`来验证TXT记录是否生效了。如果确认生效了，则按回车完成证书的申请：

```
Press Enter to Continue
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2019-01-10. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

上面的信息告诉我们，证书保存在`/etc/letsencrypt/live/example.com/fullchain.pem`，证书的秘钥保存在`/etc/letsencrypt/live/example.com/privkey.pem`，我们可以把这两个文件拷贝到服务器上用于设置https。

证书的有效期是90天，我们可以在证书失效前30天更新证书的有效期：

```
$ docker run -it --rm --name certbot \
    -v $PWD:/etc/letsencrypt \
    certbot/certbot:v0.27.1 \
    renew
```


# 总结

本文通过手动模式一步一步操作，完成了通配符域名的申请。只要理解了let's encrypt验证域名的http模式和dns模式的原理，申请域名就能做到心中有数。下一篇将讲一讲在nginx环境下，如何使用申请到的证书配置https。


