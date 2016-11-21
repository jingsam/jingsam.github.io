---
title: 开发组件库时 Vue 应该放哪儿：devDependencies or peerDependencies？
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

下表是我在 awesome-vue 中找到的 28 个 Vue 组件库，统计了 Vue 在这些库中的位置。其中，11 个库选择放到 `dependencies`，10 个库选择放到 `devDependencies`，4 个库选择放到 `peerDependencies`，2 个库选择不放 Vue 依赖，最后还有 1 个库选择在 `devDependencies` 和 `peerDependencies` 同时加上 Vue 依赖。

| UI                       | dependencies        | devDependencies | peerDependencies | none |
| ------------------------ | ------------------- | --------------- | ---------------- | ---- |
| vue-strap                | ✔                  |                 |                  |      |
| gritcode-components      | ✔                  |                 |                  |      |
| Keen-UI                  | ✔                  |                 |                  |      |
| material-ui-vue          | ✔                  |                 |                  |      |
| vue-admin                | ✔                  |                 |                  |      |
| vue-carbon               | ✔                  |                 |                  |      |
| vue-impression           | ✔                  |                 |                  |      |
| vuikit                   | ✔                  |                 |                  |      |
| vue-material             | ✔                  |                 |                  |      |
| vue-kit                  | ✔                  |                 |                  |      |
| vue-material-design      | ✔                  |                 |                  |      |
| vuestrap-base-components |                     | ✔              |                  |      |
| vux                      |                     | ✔              |                  |      |
| vue-materialize          |                     | ✔              |                  |      |
| mint-ui                  |                     | ✔              |                  |      |
| N3-components            |                     | ✔              |                  |      |
| vue-desktop              |                     | ✔              |                  |      |
| vue-beauty               |                     | ✔              |                  |      |
| radon-ui                 |                     | ✔              |                  |      |
| vue-antd                 |                     | ✔              |                  |      |
| bootstrap-vue            |                     | ✔              |                  |      |
| iview                    |                     |                 | ✔               |      |
| wovue                    |                     |                 | ✔               |      |
| vue-bulma                |                     |                 | ✔               |      |
| quasar                   |                     |                 | ✔               |      |
| vueboot                  |                     |                 |                  | ✔   |
| material-components      |                     |                 |                  | ✔   |
| element                  |                     | ✔              | ✔               |      |

下面分析下各种选择的优劣：
- `dependencies`：放到 `dependencies` 的好处是安装组件库的时候 Vue 会自动安装。问题是已有的工程项目中往往已经安装了 Vue，会导致在 npm 1 或 2 中重复安装 Vue。虽然此问题在 npm 3 中可以规避，但是如果已安装的 Vue 版本与组件库所依赖的版本不兼容时，Vue 仍然会重复安装。
- `devDependencies`：由于组件库单元测试时会用到 Vue，所以严格来说 Vue 是属于开发工具包，放到 `devDependencies` 合情合理。但是，在 `devDependencies` 中的 Vue 依赖不会自动安装，所以需要用户去查文档来确定到底应该安装哪个版本的 Vue，对用户似乎不太友好。
- `peerDevDependencies`：Vue 组件库本质上是 Vue 的插件，依赖于外部提供的 Vue。把 Vue 放到这里，npm 能够提示用户需要安装哪个版本的 Vue，这点比 `devDependencies`  友好。问题在于，从 npm 3 开始，`peerDevDependencies` 中的依赖不会自动安装，会导致自动集成测试失败。
- 不添加依赖：这基本上是最差的选择了，用户甚至不能知道组件库到底依赖哪个版本的 Vue，只能靠猜。
- `devDependencies`、`peerDevDependencies` 同时添加：这是我和 Element UI 的开发者讨论后最优的做法。`peerDevDependencies` 中的 Vue 是组件库运行所依赖的 Vue 的最低版本，而 `devDependencies` 运行测试时需要的 Vue 版本，一般情况下 `devDependencies >= peerDevDependencies`。这种做法保证了 npm 1、2、3 都不会出问题，并且当用户安装组件库是给予友好的提示信息。

到这里，我们可以得出以下结论：**在开发 Vue 组件库是，应当同时在 `devDependencies` 和 `peerDevDependencies` 添加 Vue 依赖。**

最后，可能有人会说：哥们，何必较真呢，能用不就行了吗？我的观点是，如果大家都按照规范来，会减少很多不必要的麻烦。想想看，如果大家在 npm 上发布包的时候不遵守 semver 规范，你还有信心说，你的程序只要 npm install 就能正常运行么？

按老罗的话说，我不是为了输赢，我就是认真！
