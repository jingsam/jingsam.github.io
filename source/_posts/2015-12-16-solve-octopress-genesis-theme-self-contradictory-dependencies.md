---
layout: theme:post
title: "解决octopress-genesis-theme安装失败的问题"
date: 2015-12-16T14:46:18+08:00
---

最近在一台新机器上搭建 Octopress 博客时，bundle 安装组件时无限 Resolving dependencies。我的 `Gemfile` 如下：
```ruby
source 'https://ruby.taobao.org/'

gem 'octopress', '~> 3.0.0'

group :jekyll_plugins do
  gem 'octopress-genesis-theme'
  gem 'octopress-codefence'
  gem 'octopress-solarized'
  gem 'octopress-feeds'
end

```

通过排除法，最终发现是 octopress-genesis-theme 这个组件出现了问题，从提示信息来看，问题是出在它的有些依赖组件需要 jekyll 2，而我安装的是 jekyll 3。所以解决方法就简单了，直接将 jekyll 的版本锁定到 2，更改后的 `Gemfile` 如下：
```ruby
source 'https://ruby.taobao.org/'

gem 'octopress', '~> 3.0.0'
gem 'jekyll', '~> 2.0'

group :jekyll_plugins do
  gem 'octopress-genesis-theme'
  gem 'octopress-codefence'
  gem 'octopress-solarized'
  gem 'octopress-feeds'
end
```

以上的方法只是个简单的 Hack，作为强迫症的我，还是希望其他组件支持最新的 jekyll 3。组件维护者没有更新的情况下，就只能忍忍了。
