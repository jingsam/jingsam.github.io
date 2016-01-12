---
layout: theme:post
title: "Windows 上最好的Linux环境：Cygwin"
---

由于写论文离不开 Office，我使用的操作系统是 Windows。但是，我的开发项目大部分与 Linux 相关，所以需要一个 Linux 开发环境。一种选择是开虚拟机，但是虚拟机的操作流畅度不佳。另外一种选择是使用 MinGW 或者 Cygwin，模拟 Linux 环境。

MinGW 将 Linux 的函数调用转换为 Windows 函数调用，实现 Windows 运行 Linux 程序；而 Cygwin 则提供一套 Linux 兼容层，使得 Linux 程序不加改动就可运行在 Windows 上。 

用一个很浅显的例子来说明 MinGW 和 Cygwin 之间的区别：假如老张去美国，怎么样才能让他快速地适应美国的生活呢？一种方法是找个人教他说英语，了解美国的生活习惯（MinGW方式）；另一种方式就是将他放到唐人街，他还是说中文、吃米饭馒头（Cygwin方式）。但是老张毕竟是中国人，让他完全适应美国的生活是很困难的；放到唐人街就不一样了，老张的生活习惯可以保持不变，睡一觉醒来还以为自己在国内呢！所以，MinGW 的兼容性没有 Cygwin 好。

相比之下，Cygwin 是比 MinGW 更好的 Linux 环境。这时不免有人站出来说，MinGW 编译的程序性能比 Cygwin 高。
