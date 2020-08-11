---
title: 手机上也可以愉快地搞nodejs开发
date: 2017-02-07 17:18:58
tags:
---

## 缘起

在微博上看到尤雨溪的这么一则微博：

{% asset_image 2017-02-07-1.png %}

嗬！在手机上也能跑nodejs，有点儿意思哈。顺手查了查nodejs官网，发现nodejs是支持ARM处理器的，有了这个先决条件，手机上跑nodejs应该没什么大障碍了。本文就来分享一下我在手机上跑nodejs的一点经验。

## 准备工作

首先你得有一部Android手机，iPhone的硬件条件有，但是由于IOS是一个封闭的系统，实际操作起来会很困难。

软件方面需要安装[Termux](1)，这是Android平台下的一个开源的终端模拟器。

另外，我建议安装一个编程键盘[Hacker's Keyboard](2)，因为一般的输入法没有Ctrl、Alt、Tab、Esc这些常用控制键，到时候会很麻烦。安装Hacker's Keyboard，设置为全键盘模式。

## 实施

首先得把nodejs安装到手机上。Termux强大的地方在于它带有一个包管理器`apt`，使用`apt`可以直接安装nodejs：
```
apt update
apt install nodejs
```

这样node和npm都安装好了，node的版本是v6.9.4，版本还比较新。

搞开发嘛，做好把git和vim也安装上：
```
apt install git vim
```

有了npm之后，我们就可以随意安装需要的包了，这里以vue-cli为例，来跑一个vuejs工程。过程与在电脑上是一样的：
```
npm i vue-cli -g
vue init webpack vue-test
cd vue-test
npm i
npm run dev
```

浏览器打开localhost:8080，你就可以看到vuejs的欢迎页面了。

## 注意事项

由于Android权限管理的原因，你并不能随意地在任何位置写入文件。你的活动范围必须在Termux的权限之内，即`data/data/com.termux/files`目录下。虽然你可以写文件到SD卡，但是有些包symbolink的时候会失败，所以保险的做法是所有的操作都在HOME目录下进行，即`data/data/com.termux/files/home`目录。

在HOME目录下操作的坏处是，但你卸载Termux时，HOME下的所有文件也会删除。所以玩玩而已，不要当真，哈哈！

## 总结

实话讲，在手机上不长不方便，我想有一下3点：
1. 屏幕太小。本来屏幕空间就有限，输入法还要占一半，估计可以通过投屏解决；
2. 没有好的编辑器。本来就没人会在手机上搞正经开发嘛，所以不会有好的编辑器，还好Vim可以凑合着用；
3. 输入不方便。这个也是最大的问题，手机键盘真的太不方便了，有点想念诺基亚N97了。

{% asset_image 2017-02-07-2.png %}

关于手机键盘，最好能携带方便，搜了下淘宝，下面两款似乎不错哦：

{% asset_image 2017-02-07-3.png %}


[1]: https://apkpure.com/termux/com.termux
[2]: https://apkpure.com/hacker-s-keyboard/org.pocketworkstation.pckeyboard
