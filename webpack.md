## 谈谈你对webpack的看法
本质上，webpack是一个现代JavaScript应用程序的静态模块打包器(module bundler)，将项目当作一个整体，通过一个给定的的主文件，webpack将从这个文件开始找到你的项目的所有依赖文件，使用loaders处理它们，最后打包成一个或多个浏览器可识别的js文件。

## webpack构建过程

1. 初始化参数：从配置文件和Shell语句中读取与合并参数，得出最终的参数
2. 开始编译：用上一步得到的参数初始化Compiler对象，加载所有配置的插件，执行对象的run方法开始执行编译
3. 确定入口：根据配置中的 entry 找出所有的入口文件
4. 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
5. 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系
6. 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个Chunk转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
8. 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

## 基本概念
#### entry
入口起点(entry point)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始，可以通过在 webpack 配置中配置 entry 属性，来指定一个入口起点（或多个入口起点）。

#### out
output 属性告诉 webpack 在哪里输出它所创建的 bundles ，以及如何命名这些文件，默认值为 ./dist

#### loader
loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript），用来告诉webpack如何转换某一类型的文件，并且引入到打包出的文件中。

#### plugins
loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量

#### 模式
通过选择 development 或 production 之中的一个，来设置 mode 参数，你可以启用相应模式下的 webpack 内置的优化

#### bundle
bundle是webpack打包出来的文件

#### chunk
chunk是webpack在进行模块的依赖分析的时候，代码分割出来的代码块。

#### module
module是开发中的单个模块。

## webpack与grunt、gulp的不同
三者都是前端构建工具。
grunt 和 gulp 是基于任务和流的。找到一个（或一类）文件，对其做一系列链式操作，更新流上的数据， 整条链式操作构成了一个任务，多个任务就构成了整个web的构建流程。
webpack 是基于入口的。webpack 会自动地递归解析入口所需要加载的所有资源文件，然后用不同的 Loader 来处理不同的文件，用 Plugin 来扩展 webpack 功能。
webpack 与前者最大的不同就是支持代码分割，模块化（AMD,CommonJ,ES2015），全局分析。

## webpack的热更新是如何做到的？说明其原理？
  简单来说就是，监听代码变化，然后通过socket通知浏览器替换相应的代码。
  [HMR](https://github.com/Jocs/jocs.github.io/issues/15)


## 使用过的loader
* babel-loader: 将ES6+转移成ES5-
* css-loader,style-loader：解析css文件，能够解释@import url()等
* file-loader：直接输出文件，把构建后的文件路径返回，可以处理很多类型的文件
* url-loader：打包图片
* url-loader增强版的file-loader，小于limit的转为Base64,大于limit的调用file-loader

## 使用过的plugins

define-plugin：定义环境变量
commons-chunk-plugin：提取公共代码

## Loader和Plugin的不同
loader 加载器
Webpack 将一切文件视为模块，但是 webpack 原生是只能解析 js 文件. Loader 的作用是让 webpack 拥有了加载和解析非 JavaScript 文件的能力，在 module.rules 中配置，也就是说他作为模块的解析规则而存在，类型为数组。

Plugin 插件
扩展 webpack 的功能，让 webpack 具有更多的灵活性，在 plugins 中单独配置。类型为数组，每一项是一个 plugin 的实例，参数都通过构造函数传入。

## 是否写过Loader和Plugin？描述一下编写loader或plugin的思路
编写 Loader 时要遵循单一原则，每个 Loader 只做一种"转义"工作。 每个 Loader 的拿到的是源文件内容（source），可以通过返回值的方式将处理后的内容输出，也可以调用 this.callback() 方法，将内容返回给 webpack 。 还可以通过 this.async() 生成一个 callback 函数，再用这个 callback` 将处理后的内容输出出去
相对于 Loader 而言，Plugin 的编写就灵活了许多。 webpack 在运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果

## 什么是长缓存？在webpack中如何做到长缓存优化？

浏览器在用户访问页面的时候，为了加快加载速度会对用户访问的静态资源进行存储，但是每一次代码升级或更新都需要浏览器下载新的代码，最简单方便的方式就是引入新的文件名称。
webpack中可以在output中指定chunkhash，并且分离经常更新的代码和框架代码。通过NameModulesPlugin或HashedModuleIdsPlugin使再次打包文件名不变。

## 什么是Tree-shaking？CSS可以Tree-shaking？
Tree-shaking是指在打包中取出那些引入了但在代码中没有被用到的死代码。webpack中通过uglifysPlugin来Tree-shaking JS。CSS需要使用purify-CSS。
