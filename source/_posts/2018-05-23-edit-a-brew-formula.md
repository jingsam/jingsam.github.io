---
title: 手动更新Homebrew formula
date: 2018-05-23 14:24:01
tags:
---

Homebrew是macOS上的软件包管理神器，类似于Ubuntu的apt-get，是使用mac作为开发机时的必装软件之一。Homebrew的软件包术语叫Formula，中文解释就是”配方”。Homebrew有“家酿、自制”的意思，80年代硅谷有一个著名的计算机俱乐部叫“Homebrew Homebrew Computer Club”，苹果的创始人Steve Jobs和Steve Wozniak都曾是这个俱乐部的活跃分子。“家酿”一个东西当然得有“配方”，所以这个取名很形象。

使用homebrew安装软件很方便，一条命令`brew install your-formula-name`就可以搞定。更新formula到最新版本，使用`brew upgrade your-formula-name`即可。

但是，各种formula是人工维护的，当软件包更新后，formula不见得能及时更新到最新版本。本文就以gdal为例，说明如何手动编辑formula文件，以此来将软件更新到最新版本。

# 基本原理

Homebrew中的每个软件包都是通过一个`formula.rb`文件来配置软件的源代码URL、依赖、编译规则和选项，例如以下是gdal的formula文件：

```ruby
class Gdal < Formula
  desc "Geospatial Data Abstraction Library"
  homepage "http://www.gdal.org/"
  url "https://download.osgeo.org/gdal/2.2.4/gdal-2.2.4.tar.xz"
  sha256 "6f75e49aa30de140525ccb58688667efe3a2d770576feb7fbc91023b7f552aa2"
  revision 2

  bottle do
    rebuild 2
    sha256 "00b28455769c3d5d6ea13dc119f213f320c247489cb2ce9d03f7791d4b53919b" => :high_sierra
    sha256 "1365de6a18caeb84d6a50e466a63be9c7541b1fab21edfc3012812157464f2c0" => :sierra
    sha256 "8c0fd81eda5a91c8a75a78795f96b6dd9c53e74974bd38cc004b55a44ae95932" => :el_capitan
  end

  ....

```

以上是一个ruby文件，好在并不需要懂ruby的语法就能看懂formula文件。`desc`和`homepage`都是描述性信息，不对软件安装产生什么影响。`url`是软件源代码的位置，编译安装时从此位置将源代码下载下来。`sha256`是源代码包的校验码，这是保证下载下来的包不被篡改。`revision`是修订版本号，主要用来保持版本号不变的情况下，对软件包打补丁，每打一次补丁，修订版本号就自增一次。当使用`brew install`安装软件包时，除非使用`--build-from-source`强制指定使用源代码安装，大部分情况下，brew会下载编译好的二进制包，这样安装起来更快。`bottle`选项就记录了各版本macOS下预编译的二进制包的校验码，这部分内容是homebrew的自动集成工具自动维护的，我们并不需要编辑修改。

更改上面的`url`和`sha256`，即可将formula的配置更新到任意版本。编辑好后，使用`brew install`或者`brew upgrade`进行安装或者更新升级。

Homebrew实际上是通过git来管理formula配置文件的，因此我们还可以发送Pull request，将我们的更新推送到GitHub上，让别人也能够方便地更新软件包。


# 具体步骤

这部分以更新gdal 2.2.4到2.3.0为例，来说明手动更新formula的步骤。由于写作本文时，gdal已经更新到2.3.0，所以某些步骤的输出可能与本文不一致，但不妨碍理解更新步骤。

## 编辑配置

使用`brew edit gdal`即可打开`gdal.rb`开始编辑，我们将`url`更新为2.3.0版本的源代码链接：

```ruby
class Gdal < Formula
  desc "Geospatial Data Abstraction Library"
  homepage "http://www.gdal.org/"
  url "https://download.osgeo.org/gdal/2.3.0/gdal-2.3.0.tar.xz"
  sha256 "6f75e49aa30de140525ccb58688667efe3a2d770576feb7fbc91023b7f552aa2"
  revision 2

  ...
```

理论上我们还需要更新`sha256`，使它和`url`相匹配。但是`sha256`需要我们使用工具计算或者从发布网站上找，不是很方便。我们可以通过下一步的安装调试，来自动计算`sha256`。

## 安装调试

使用`brew install gdal --verbose --debug --build-from-source`来安装调试gdal的formula，如果已经安装旧版本的gdal，那么使用`brew upgrade gdal --verbose --debug --build-from-source`。`--verbose`表示显示详细输出，便于调试；`--debug`打开调试；`--build-from-source`强制从源代码编译。

安装过程中，会报sha256校验码不匹配的警告，并打印出`url`所指向的源代码包的sha256校验值，这是因为在上一步我们并没有修改`sha256`，配置文件中`sha256`还是gdal 2.2.4版本的。这时候重新使用`brew edit gdal`打开并编辑formula文件，将`sha256`更改为正确的校验值。

最后，再安装调试，经过漫长的编译，成功地安装上了gdal的最新版本。

## 测试

测试包括对软件包的测试和对formula文件的测试。使用`brew test gdal`可以测试gdal的功能是否正常，使用`brew audit --strict gdal`测试formula文件是否正确。运行`brew audit`时，会报告`revision 2`需被删除，这是因为当前homebrew线上库中还没有gdal 2.3.0，意味着本地端的gdal应该是第一版本，不存在修订版本之说。同样地，使用`brew edit gdal`打开并删除`revision`部分，然后再重新测试。

## 推送更新

通过上述步骤，我们完成了gdal的手动更新，如果将更新推送到homebrew的线上库中，那么其他人就可以方便地更新到最新版本。并且推送到线上库后，homebrew的自动集成工具会自动地编译生成二进制包，这样就不需要从源代码编译那么耗时了，可谓是利人利己。

由于homebrew是在GitHub上协作的，所以更新一个formula就和发一个Pull request是一样的，基本步骤如下：

1.使用`cd $(brew --repository homebrew/homebrew-core)`切换到本地的homebrew-core目录；
2.使用`git commit`提交自己的修改；
3.把https://github.com/Homebrew/homebrew-core fork一份；
4.使用`git remote add`命令添加自己的fork的homebrew-core库；
5.使用`git push`推送将本地提交推送到自己的fork的homebrew-core库中；
6.在GitHub网页上发起Pull request。

## 一键更新

上面一步步完成了编辑配置、安装调试、测试、推送更新，操作起来有些繁琐。但其实homebrew还提供了一个工具，能够一键完成上面4个步骤，命令如下：

```
brew bump-formula-pr gdal --URL=https://download.osgeo.org/gdal/2.3.0/gdal-2.3.0.tar.xz --audit --strict
```

`brew bump-formula-pr`自动的修改formula配置文件、检查文件错误、提交并推送更新，其中提交并推送更新的过程需要使用`hub`来在终端上操作GitHub，可以使用`brew install hub`来安装这个工具。
