---
title: 手动修改Homebrew formula
date: 2018-05-23 14:24:01
tags:
---


Formula -- 配方

```
brew --repository homebrew/homebrew-core

brew edit gdal
edit url hash revison
brew install gdal --verbose --debug --build-from-source
brew test gdal
brew audit --strict gdal
git commit / git push set-upstream jingsam gdal-2.3.0

brew bump-formula-pr --URL=https://download.osgeo.org/gdal/2.3.0/gdal-2.3.0.tar.xz --audit --strict --dry-run
```
