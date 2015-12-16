---
layout: theme:post
title: "Export PATH 可以用通配符吗？"
date: 2015-12-16T15:08:08+08:00
---

我在 Cygwin 上安装 bundler 之后，使用 bundle 命令提示 command not found。以我的经验来看，估计是 bundle 的路径没有添加到 `PATH` 环境变量中。查找 bundle 文件发现它存在于 ~/bin 目录下，把这个目录 export 到 PATH 变量中，命令如下：
```
export PATH=~/bin:$PATH
```

而后发现 bundle 命令还是不行运行，百思不得解之际，突然想到是不是 `PATH` 变量中不支持使用 `~` 这样的通配符？遂改用绝对路径：
```
export PATH=/home/jingsam/bin:$PATH
```
问题解决！

至于 `PATH` 变量到底支不支持通配符，等以后查询到相关信息再来更新。
