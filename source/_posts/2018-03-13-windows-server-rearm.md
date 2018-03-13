---
title: Windows Server 2012评估版本延长使用期限
date: 2018-03-13 15:00:51
tags:
---

最近一台Windows Server 2012 R2数据库服务器隔一个小时就关机，恰巧碰到要演示，这种突发情况让我手忙脚乱。（别问我为什么要用Windows Server做服务器，客户要求的）网上搜了下，发现是Windows授权到期了，需要激活。

网上确实有很多Windows Server破解激活工具，但是有各种各样的工具，不知道哪个能起作用。而且，这些工具大部分杀毒软件都报有病毒，不太敢用。最重要的是，破解激活涉及到版权问题，道义上过不去。

由于我使用的是Windows Server评估版本，评估版本可以重置5次试用期，所以加上安装的那次，那么理论了最多可以试用1080天，差不多3年。

重置方法很简单，以管理员身份打开命令提示符，输入以下命令即可重置试用期，获得180天试用：

```
slmgr.vbs /rearm
```
**重置成功后需要重启一次才能生效。**

通过以下命令可以查看剩余的试用时间和可重置次数：

```
slmgr.vbs /dlv
```

那么重置次数试用完之后怎么办呢？据说可以通过改注册表的方式获得额外的重置次数，但是这种方法可能违反了系统试用协议。具体的做法是使用`regedit`打开注册表编辑器，找到：

```
HKEY_LOCAL_MACHINE -> SOFTWARE -> Microsoft -> Windows NT -> CurrentVersion -> SoftwareProtectionPlatform
```

将键`SkipRearm`的值设为1，再用`slmgr.vbs /rearm`继续重置，据说这种方法可以使用8次。

# 参考链接

[Windows Server 2012 評估版與延長使用期限](http://www.vixual.net/blog/archives/180)
