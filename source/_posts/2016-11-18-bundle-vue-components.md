---
title: 打包 Vue 组件库的正确姿势
date: 2016-11-18 00:15:28
tags:
---


为了方便其他开发者使用组件库，开发的 Vue 组件库在发布之前需要对其打包。本文基于 Webpack 讨论打包 Vue 组件库的正确方法。

## 选择正确的打包格式

首先，我们必须明确组件库的使用场景。有些场景是直接使用 `<script>` 在 HTML 中引入，有些场景是使用打包工具在后台构建。作为组件库，应该兼容这些使用场景。组件库应该保持中立，不应该限定于某种使用方式或者打包工具。例如，虽然 Webpack 很流行，组件库不能声明只支持 Webpack 方式使用，忽略了其他选择。原因在于，打包工具并不只有 webpack，还有 browserify、rollup 等。另外，前端工具发展很快，今年流行的工具明年可能就没人用了，你肯定不希望你的组件库会随着某个工具不流行而消逝吧。回顾下曾经流行的 grunt、glup，一大堆基于它们的插件随着工具本身的不流行而被扔进了垃圾箱。

为了支持多种使用场景，我们需要选择合适的打包格式。常见的打包格式有 CMD、AMD、UMD，CMD只能在 Node 环境执行，AMD 只能在浏览器端执行，UMD 同时支持两种执行环境。显而易见，我们应该选择 UMD 格式。Webpack 中指定输出格式的设置项为 `output.libraryTarget`，其支持的格式有：

- "var" - 以一个变量形式输出： var Library = xxx (default)；
- "this" - 以 this 的一个属性输出： this["Library"] = xxx；
- "commonjs" - 以 exports 的一个属性输出：exports["Library"] = xxx；
- "commonjs2" - 以 module.exports 形式输出：module.exports = xxx；
- "amd" - 以 AMD 格式输出；
- "umd" - 同时以 AMD、CommonJS2 和全局属性形式输出。

以下是 `webpack.config.js` 中 `output` 设置的示例：
```javascript
output: {
    path: path.resolve(__dirname, '../dist'),
    publicPath: '/dist/',
    filename: 'iview.js',
    library: 'iview',       // 模块名称
    libraryTarget: 'umd',   // 输出格式
    umdNamedDefine: true    // 是否将模块名称作为 AMD 输出的命名空间
}
```

到此，我们解决了组件库输出的问题。

## 如何打包组件依赖

在[前一篇][1]文章中我们讨论了组件库实质上是 Vue 的插件，Vue 应该是组件库的外部依赖。组件库的使用者会自行导入 Vue，打包的时候，不应该将 Vue 打包进组件库。

在 webpack 中，我们可以将 Vue 设置为 `externals`，以避免将 Vue 打包进组件库，相应的设置如下：
```javascript
externals: {
    vue: 'vue'
}
```

啊哈，我们搞定了组件依赖问题。至此，读者可能很皱起眉头开始埋怨我了：这么简单的问题，查下 webpack 文档不就得了，还用得着我啰里啰嗦地写这么多！

事实上，问题往往没有我们想得那么简单！如果你将打包后的组件库以 `<script>` 标签形式直接引入，你会发现并不能正常执行，提示 vue 未定义。

为了分析问题，我们将打包的代码前几行拿出来看看：
```javascript
(function webpackUniversalModuleDefinition(root, factory) {
    if(typeof exports === 'object' && typeof module === 'object')
        module.exports = factory(require("vue"));
    else if(typeof define === 'function' && define.amd)
        define("iview", ["vue"], factory);
    else if(typeof exports === 'object')
        exports["iview"] = factory(require("vue"));
    else
        root["iview"] = factory(root["vue"]);
})(this, function(__WEBPACK_EXTERNAL_MODULE_157__) {
    // ....
})
```

我们可以看见，打包后的代码以 4 种形式声明了 Vue 依赖：
1. `module.exports = factory(require("vue"))` - commonjs2 形式；
2. `define("iview", ["vue"], factory)` - AMD 形式；
3. `exports["iview"] = factory(require("vue"))` - commonjs 形式；
4. `root["iview"] = factory(root["vue"])` - 全局变量形式。

以 `<script>` 标签形式使用组件时，会同样使用 `<script>` 标签导入 Vue。Vue 导入的变量是 "window.Vue" 而不是 "window.vue"，因此会出现 vue 未定义的错误。

幸好，webpack 可以为各种导入形式设置不同名称，设置如下：
```javascript
externals: {
    vue: {
        root: 'Vue',
        commonjs: 'vue',
        commonjs2: 'vue',
        amd: 'vue'
    }
}
```

再次打包，你可以发现打包的组件库不管是 `<script>` 标签方式还是后端构建，都可以正常工作了。

最后，帖一个打包 iView 组件库的 webpack 配置：
```javascript
var path = require('path');
var webpack = require('webpack');

module.exports = {
    entry: {
        main: './src/index.js'
    },
    output: {
        path: path.resolve(__dirname, '../dist'),
        publicPath: '/dist/',
        filename: 'iview.js',
        library: 'iview',
        libraryTarget: 'umd',
        umdNamedDefine: true
    },
    externals: {
        vue: {
            root: 'Vue',
            commonjs: 'vue',
            commonjs2: 'vue',
            amd: 'vue'
        }
    },
    resolve: {
        extensions: ['', '.js', '.vue']
    },
    module: {
        loaders: [{
            test: /\.vue$/,
            loader: 'vue'
        }, {
            test: /\.js$/,
            loader: 'babel',
            exclude: /node_modules/
        }, {
            test: /\.css$/,
            loader: 'style!css!autoprefixer'
        }, {
            test: /\.less$/,
            loader: 'style!css!less'
        }, {
            test: /\.(gif|jpg|png|woff|svg|eot|ttf)\??.*$/,
            loader: 'url?limit=8192'
        }, {
            test: /\.(html|tpl)$/,
            loader: 'vue-html'
        }]
    },
    plugins: [
        new webpack.DefinePlugin({
            'process.env': {
                NODE_ENV: '"development"'
            }
        })
    ]
}
```

[1]: /2016/11/01/peerDependencies-in-Vue-components.html
