---
title: linux进程后台运行
date: 2018-05-07 17:25:15
tags:
---

在linux上启动Web服务，当退出终端后，Web服务进程也会随着关闭。产生这种问题的原因在于，当用户注销或者网络断开后，终端后收到挂断信号（SIGHUP）,并向子进程广播SIGHUP信号，子进程收到SIGHUP信号而关闭。因此，让linux后台持续运行的方法有以下几种：

1.改变子进程的所属的父进程，只要父进程不关闭，子进程也不会关闭；
2.让子进程忽略挂断信号，即使收到SIGHUP信号，也任性地继续运行；
3.不向子进程广播SIGHUP信号，子进程收不到SIGHUP信号，因而不会关闭。

# 第一种方式

使用`setsid`可以新开一个session运行进程，此session不从属于当前终端，因此终端关闭时进程也不会退出。

```
>setsid ping www.baidu.com
>ps -ef | grep www.baidu.com
501 57697     1   0  5:55下午 ttys000    0:00.01 ping www.baidu.com
```

从上面可以看出，ping进程的父进程是1，即init进程，因此只要电脑不关机，ping进程就不会停止。

linux下自带`setsid`这个命令，但是macOS上并没有这个命令。此时，可以结合使用`()`和`&`达到同样的效果。`()`可以新开一个subshell，`&`让命令后台运行。

```
>(ping www.baidu.com &)
>ps -ef | grep www.baidu.com
501 57781     1   0  6:01下午 ttys000    0:00.00 ping www.baidu.com
```

可以看到，效果与`setsid`是一样的。


# 第二种方式

使用`nohup`可以使进程忽略SIGHUP信号，这种方式也是最常用的。例如：

```
>nohup ping www.baidu.com &
[1]  + 59193 exit 127   nohup www.baidu.com

>ps -ef | grep www.baidu.com
501 59193 39100   0  6:05下午 ttys000    0:00.01 ping www.baidu.com
```

可以看到，ping进程的父进程并不为1，nohup是让进程忽略SIGHUP信号实现进程不退出的。

需要注意的是，当在`zsh`中使用`nohup`时，退出终端时会提示：

```
zsh: you have running jobs.
```

再次强行退出，那么进程仍然会被干掉。这时候，采用以下命令：

```
nohup ping www.baidu.com &!
```


# 第三种方式

使用nohup的前提是，进程以nohup来启动。但是，如果启动时忘记了以nohup启动，有什么方法在不停止进程的情况，让它继续后台运行呢？接下来就要将另外一个命令：`disown`。`disown`的原理是，将子进程从终端任务队列中移除，所以即使终端挂断，子进程也收不到SIGHUP信号。

假设现在使用ping命令：

```
>ping www.baidu.com
64 bytes from 14.215.177.38: icmp_seq=0 ttl=51 time=19.642 ms
64 bytes from 14.215.177.38: icmp_seq=1 ttl=51 time=97.976 ms
64 bytes from 14.215.177.38: icmp_seq=2 ttl=51 time=88.996 ms
...
```

这时采用`Ctrl + z`使它进入后台，使用`jobs`查看后台进程：

```
>jobs
[1]  + suspended  ping www.baidu.com
```

可以看到虽然ping进程进入后台，但是进程被挂起了，没有继续运行。使用`bg`命令可以使他在后台继续运行：

```
>bg %1
[1]  + 58174 continued  ping www.baidu.com
>jobs
[1]  + running    ping www.baidu.com
```

通过组合`Ctrl + z`和`bg`，成功地将前台进程变为了后台进程。为了让进程不随终端关闭而终止，还差最后一步：

```
>disown %1
>jobs
```

上面使用`jobs`命令查看任务队列，发现ping进程不在任务队列中，意味着进程不会收到SIGHUP信号。


# 注意事项

以上我们通过三种方式，避免进程随着终端关闭而被杀掉：

1. `setsid`改变父进程，只要父进程不关闭，进程就可以持续运行；
2. `nohup`使得进程忽略SIGHUP信号，父进程即使发送挂断信号，进程也不会终止；
3. `disown`将进程从任务队列中移除，保证进程收不到SIGHUP信号。

但是，以上种种方法只是避免了进程受到SIGHUP信号的影响，进程的持续运行还需要一些其他环境，例如stdin、stdout以及stderr。通常从终端启动的进程，会继承终端的stdin、stdout和stderr。当终端断掉之后，stdin、stdout和stderr也会随着消失，若此时后台进程需要读写stdin、stdout、stderr，该进程将会暂停或者挂住。所以，为保证进程正常后台运行，最好启动时对输入输出重定向：

```
>ping www.baidu.com > a.log 2>&1 &
```

此时，将stdout和stderr重定向到文件`a.log`中，文件`a.log`不受终端关闭的影响。如果进程依赖于stdin，意思是进程需要由于键盘输入，那就说明这是个交互式程序，交互式程序后台运行就没多大意义了。

# 参考链接
[Linux 技巧：让进程在后台可靠运行的几种方法](https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/index.html)
[Difference between nohup, disown and &](https://unix.stackexchange.com/questions/3886/difference-between-nohup-disown-and)
