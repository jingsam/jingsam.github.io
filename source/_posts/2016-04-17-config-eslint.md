---
title: Eslint配置实战
date: 2016-04-17 15:11:37
tags:
---


在软件开发中，编码风格的统一，对项目可维护性具有重要作用。不像python、go语言对代码格式有着严格的规范，Javascript的语法非常灵活，开发人员可以写出天马行空的Javascript代码。而且，Javascript编译器的容错度较高，会诱使开发人员随意编写杂乱的代码。不规范的编码风格，不仅增加了理解代码的困难，也常常会产生隐含的bug。因此，Javascript相比于其他语言，更需要制定相应的编码规范。

统一代码规范，最直观的方法是团队成员商定一套统一的代码规范，大家按照这套规范进行代码编写。但是这种约定的方式约束力太弱，总会有人有意或无意地不遵守规范。更好的方式是，借助代码格式检测工具，自动化地进行格式检查，并将格式检查作为代码测试的一部分，集成到项目的持续集成过程中。

主流的Javascript代码格式检查工具有JSLint、JSHint、JSCS、ESLint。这些工具中，ESLint可以灵活配置规则，并以插件机制支持各种扩展，因而比前3种工具在功能上更为全面。本文以ESLint作为代码检查工具，描述如何将其集成到Javascript开发项目中。

## 安装

安装ESLint很简单，可以通过npm安装。ESLint既可以全局安装，也可以本地安装。推荐本地安装，因为这样做的好处一是不污染全局，二是分发比较方便，不会产生外部依赖，因为无法保证其他人的机器上也全部安装了ESLint。安装ESLint的命令如下：
```bash
npm install eslint --save-dev
```

## 初始化
要使ESLint生效，需要在项目根目录下建立一个配置文件，来设定检查规则。ESLint提供了交互式的命令行工具，可以帮助我们快速地配置规则，初始化命令如下：
```bash
./node_modules/.bin/eslint --init
```

## 使用
在package.json的script下添加一个lint命令，并指定要检测的文件或文件夹：
```json
{
  "lint": "eslint *.js models/**"
}
```

命令行中运行如下命令，即可得到检测结果
```bash
npm run lint
```

实践中，通常会将lint集成到测试流程中：
```json
{
  "lint": "eslint *.js models/**",
  "test": "npm run lint && mocha"
}
```


## 例外规则
编码过程中出于实际需要，会不可避免地违反lint规则。ESLint提供了例外规则，供开发人员临时地关闭某些检测规则。

屏蔽文件的某个区块：
```javascript
/* eslint-disable no-console */

console.log("hello, world")

/* eslint-enable */
```

屏蔽一行：
```javascript
/* eslint-disable-line no-console */
console.log("hello, world")

console.log(42) // eslint-disable-line no-console
```

由于项目中会用到测试框架mocha，测试用例包含的全局变量如describe、it等，会造成lint报错。屏蔽这些错误的方法是，在.eslintrc中添加运行环境mocha，配置如下：
```json
{
  "env": {
    "node": true,
    "mocha": true
  }
}
```

如果使用了Chai或者Shouldjs，还会产生no-unused-vars的错误，用例外规则屏蔽之：
```javascript
var should = require('chai').should() // eslint-disable-line no-unused-vars
```


## Sublime集成
上面配置好ESLint之后，根据命令行检测出结果，定位到文件中进行相应的修改，然后再检测在修改。这种命令行方式比较繁琐，更好的办法是集成到代码编辑器中，实时地进行检测。

ESLint可以很好地集成到Sublime中，通过Package Control安装SublimeLinter、SublimeLinter-contrib-eslint两个插件，即可完成对ESLint的集成。

要启用ESLint，还需要配置SublimeLinter的nodejs的路径。具体做法是打开Preference > Package Settings > SublimeLinter > Settings - Default文件，在paths下面填写响应平台下的node路径：
```json
"paths": {
  "linux": [],
  "osx": [],
  "windows": ["D:/Program Files/nodejs/node.exe"]
}
```

配置好路径后，务必**重启Sublime**，才会使得SublimeLinter生效。

我们在编写代码时，SublimeLinter会读取项目目录下的.eslintrc规则文件，实时动态地标出错误代码的位置。

另外我们可以开启SublimeLinter的debug模式，该模式下将会在状态栏给出错误的说明。启用的方法是打开Tools > SublimeLinter > Debug Mode。
