---
layout: theme:post
title: "Octopress 3.0博客搭建教程"
date: 2015-10-21T21:16:53+08:00
---

Octopress是一款专为黑客准备的博客系统，它简洁美观且定制性强。相信大家已经在网上看到过很多Octopress 2.0博客教程，但本文所要描述的是Octopress 3.0的博客搭建过程。

Octopress 3.0相对于2.0有很大的不同。Octopress 2.0是通过git分发的，需要修改Octopress源代码来构建博客系统，因此难以合并未来的更新。Octopress的作者Mathis认为git并不是一个合适的软件分发方式，因此从3.0开始通过gem方式分发。并且增加了一系列octopress命令，代替之前的rake命令。

## 准备工作
Octopresss基于Jekyll，而jekyll是用Ruby开发的，因此系统要安装有Ruby。新建一个blog文件夹，作为博客的主目录。在主目录下新建文件`Gemfile`，文件内容如下：
```ruby
source 'https://ruby.taobao.org/'  # 更改为淘宝镜像，下载更稳定

gem 'octopress', '~> 3.0.0'        # 安装Octopress 3.0，依赖包包括jekyll会自动安装

group :jekyll_plugins do           # 安装插件，具体内容后面再讲
  gem 'octopress-genesis-theme'    # 主题插件
  gem 'octopress-codefence'        # GitHub风格代码块
  gem 'octopress-solarized'        # 代码块高亮采用solarized dark主题
  gem 'octopress-feeds'            # 订阅功能
end
```

更多Octopress插件可在[这里](https://github.com/octopress)找到。

在主目录下，通过`bundle`命令安装所有需要gem包。如果提示`bundle`命令不存在，则需要先安装bundler，命令如下
```bash
gem install bundler
```

## 初始化
在主目录下，应用如下命令进行初始化：
```bash
octopress init .
```

初始化完成后会在目录下生成`_templates`文件夹，里面包含draft、page、post三个文件，这些文件是用来作为自定义模板的。其实还有一个`octopress new`初始化命令，其效果等同于`jekyll new` + `octopress init`。不推荐使用这种方式，因为`jekyll new`产生的文件octopress用不着。

## 配置yml
在主目录下，新建`_config.yml`文件，添加如下配置
```yml
title: your blog
url: http://your-site.com

date_format: "%Y-%m-%d"
time_format: "%H:%M"

# Default extension for new posts and pages
post_ext: md
page_ext: html

# Default templates for posts and pages
# Found in _templates/
post_layout: theme:post
page_layout: theme:page

# Format titles with titlecase?
titlecase: true

# Change default template file (in _templates/)
post_template: post
page_template: page
draft_template: draft
```

octopress-genesis-theme主题插件的配置文件需在_plugins/theme下新建config.yml，我的配置文件内容如下
```yml
# Settings for main header
title: jingsam's blog
subtitle: A geospatial hacker

# Links for main navigation
nav:
  - { url: '/', title: 'Post' }
  - { url: '/archive/', title: 'Archive' }
  - { url: '/feed/', title: 'Subscribe' }

# Link labels
permalink_label: "Permalink"
read_more_label: "Continue Reading →"

# Show excerpts on post index
excerpt_posts: true
# Excerpt linkposts on index
excerpt_linkposts: false

search: google

sharing:
  - facebook
  - twitter
  - gplus
  - email

# Defaults to sharing with links (for speed and privacy)
# To use javascript share buttons, set share_with: buttons
share_with: links

# Embed comments, options: false, facebook, disqus
comments: disqus

# Center the text in post and page headings.
center_headings: true
```

## 开始写作
写作新的博文使用如下命令：
```bash
octopress new post "my title"
```

如果从以前的博客系统迁移过来，只需将所有的markdown格式的文章放到`_posts`目录下。

## 发布
Octopress支持rsync、s3、git方式发布。以发布到GitHub上为例，首先初始化deploy参数：
```bash
octopress deploy init git git@github.com:user/project
```

以上命令将会生成一个`_deploy.yml`文件，填写相应的参数后完成配置。需注意的是如果`_deploy.yml`包含密码信息，请在`.gitignore`中设置忽略，避免密码泄露。另外，我们需要建立两个分支，`master`分支用来发布网页，`source`分支用来管理源代码，切不可直接deploy到`source`分支上破坏了源代码。如果发布分支和源代码分支是统一分支，deploy时Octopress会出现警告。

最后使用如下命令完成发布：
```
octopress deploy
```

## 总结
本教程介绍了Octopress 3.0博客搭建流程。Octopress 3.0相对于2.0几乎是重新设计，所做的修改不会污染到源代码，因此更安全地升级。然而Octopress 3.0的文档还不是很完善，需要长时间地摸索，用户可通过如下命令查看octopress及其插件的文档：
```
octopress docs
```

