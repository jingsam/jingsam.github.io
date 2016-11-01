---
title: 开发 Vue 组件库时为什么不使用 peerDependencies？
date: 2016-11-01 19:16:58
tags:
---

最近在和 Aresn 开发 iView 组件库的时候，关于依赖包 Vue 到底是放在 devDependencies 还是 peerDependencies 有些争论。本着打破砂锅问到底的精神，这篇文章讨论下 package.json 里面的各种 depedencies 字段到底是干嘛的。

npm 的 package.json 包括 5 种 dependencies：
1. `dependencies`：应用能够正常运行所依赖的包。这种 dependencies 是最常见的，用户在使用 `npm install` 安装你的包时会自动安装这些依赖。
2. `devDependencies`：开发应用时所依赖的工具包。通常是一些开发、测试、打包工具，例如 webpack、ESLint、Mocha。应用正常运行并不依赖于这些包，用户在使用 `npm install` 安装你的包时也不会安装这些依赖。
3. `peerDependencies`：应用运行依赖的宿主包。最典型的就是插件，例如各种 jQuery 插件，这些插件本身不包含 jQeury，需要外部提供。用户使用 npm 1 或 2 时会自动安装这种依赖，npm 3 不会自动安装，会提示用户安装。
4. `bundledDependencies`：发布包时需要打包的依赖，似乎很少见。
5. `optionalDependencies`：可选的依赖包。此种依赖不是程序运行所必须的，但是安装后可能会有新功能，例如一个图片解码库，安装了 `optionalDependencies` 后会支持更多的格式。

从以上的定义可以看出，`dependencies` 是程序运行依赖，`devDependencies` 一般是一些开发工具，`peerDependencies` 一般用于插件。

假如我们开发一套 Vue 的组件库 VueXXX，Vue 到底放在那里合适呢？要分析这个问题，我们应该从一个 VueXXX 的使用者的角度来推演。


