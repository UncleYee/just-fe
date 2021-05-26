# webpack

## 1.webpack 原理

* 1.初始化参数：从配置文件和 Shell 语句中读取、合并参数。
* 2.开始编译：使用参数初始化 compiler 对象，加载所有配置的插件，执行 run 方法开始编译。
* 3.确定入口：根据配置中的 entry 找到所有入口文件，执行 make 方法分析入口文件。
* 4.编译模块：从入口文件触发，使用不同的 loader 对模块及其依赖模块进行翻译处理（生成 ATS 抽象语法树，通过 ATS 分析和递归加载模块）。
* 5.完成编译：编译完所有模块后，得到每个模块翻译后的内容和依赖关系。
* 6.输出资源：根据入口和模块之间的关系，组装成一个包含多个模块的 chunk，再把每个 chunk 转化成一个单独的文件加入到输出列表，这里是可以修改输出内容的最后机会。
* 7.完成输出：根据配置的路径和文件名，把输出资源内容写到文件中。

> 简化：参数解析，找到入口文件，调用 Loader 编译文件，遍历 AST 收集依赖，生成 Chunk，输出文件
> 在上面的过程中，webpack 会在特定的时间点广播特定的事件，插件会在监听到广播后执行特定的逻辑，插件也可以根据 webpack 提供的 api 来改变 webpack 的执行结果。

## 2.webpack 性能优化 - （打包体积和打包速度）

* 体积分析：webpack-bundle-analyzer
* 速度分析：speed-measure-webpack-plugin

### 2.1 影响打包速度的问题

* 1.开始打包，搜索所有的依赖项，需要一定的时间
* 2.解析所有的依赖（loader)
* 3.将所有的依赖打包到一个文件（ES6 -> AST -> AST ->ES5），设计到大量的计算，容易卡住
* 4.二次打包：当更改项目中的一小点时，所有文件都需要重新打包，然而很多文件都没变，尤其是第三方库

### 2.2 解决打包速度的问题

* 1. 优化解析时间：由于运行在 nodejs 上的 webpack 是单线程的，开启多线程可以加快速度。
  * 1. thred-loader: thread-loader 使用起来也非常简单，只要把 thread-loader 放置在其他 loader 之前， 那 thread-loader 之后的 loader 就会在一个单独的 worker 池(worker pool)中运行。
  * 2. HappyPack: 多进程处理
* 2. 合理利用缓存：
  * 1. cache-loader, 使用方式和 thred-loader 一样
  * 2. HardSourceWebpackPlugin，或者 babel-loader 的 cacheDirectory 标识
* 3. 优化压缩时间：
  * 1. webpack3: 启动打包时加上 --optimize-minimize
  * 2. webpack4: 默认内置使用 terser-webpack-plugin
* 4. 优化搜索时间：
  * 1. 优化 loader 配置：可以通过 test 、 include 、 exclude 三个配置项来命中 Loader 要应用规则的文件
* 5. 使用外部扩展 externals, 将不怎么需要更新的第三方库脱离webpack打包，不被打入bundle中，从而减少打包时间,比如jQuery用script标签引入

### 2.3 如何提高 webpack 的构建速度？

* 1. 多入口的情况下，使用CommonsChunkPlugin来提取公共代码
* 2. externals 提取常用库
* 3. happyPack(多进程) 多线程加速
* 4. 使用webpack-uglify-parallel来提升uglifyPlugin的压缩速度。 原理上webpack-uglify-parallel采用了多核并行压缩来提升压缩速度
* 5. tree-shaking 和 Scope Hoisting(作用域提升，webpack 会把引入的 js 文件“提升到”它的引入者顶部) 来剔除多余代码。

```js
// tree-shaking 使用方法，必须 webpack 4
// package.json 中，配置 sideEffects: false，表示所有文件都没有副作用，可以安全删除
// 或者 sideEffects: ['./xxxx.js']，指明副作用文件
```

## 3. webpack loader

### 3.1 常见的 loader

* file-loader：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件
* url-loader：和 file-loader 类似，但是能在文件很小的情况下以 base64 的方式把文件内容注入到代码中去
* source-map-loader：加载额外的 Source Map 文件，以方便断点调试
* image-loader：加载并且压缩图片文件
* babel-loader：把 ES6 转换成 ES5
* css-loader：加载 CSS，支持模块化、压缩、文件导入等特性
* style-loader：把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS。
* eslint-loader：通过 ESLint 检查 JavaScript 代码

### 3.2 loader 的工作原理

webpack 只能直接处理 JavaScript 代码，任何非 js 代码都必须先转换为 js 代码，才能参与打包，loader 就是这样一个代码转换器。
它由 loader runner 调用，接收原始资源数据作为参数（可多个 loader 叠加），最终输出 JavaScript 代码。

### 3.3 loader 分类

* pre: 前置 loader
* normal: 普通 loader
* inline: 内联 loader
* post: 后置 loader

### 3.4 loader 执行顺序

* 不同优先级：pre > normal > inline > post
* 相同优先级：从右向左，从下到上

### 3.5 loader 是一个导出一个函数的 node 模块

```js
module.exports = function loader (source) {
  console.log('simple-loader is working');
  return source;
}
```

## 4. webpack plugin

### 4.1 常见的 plugin

* define-plugin：定义全局变量 ***
* html-webpack-plugin：简化html文件创建 ***
* uglifyjs-webpack-plugin：通过UglifyES压缩ES6代码 ***
* webpack-parallel-uglify-plugin: 多核压缩,提高压缩速度
* webpack-bundle-analyzer: 可视化webpack输出文件的体积 ***
* mini-css-extract-plugin: CSS提取到单独的文件中,支持按需加载
* spulitChunksPlugin: 抽取第三方库，打包成不同的 bundle ***

## 5. webpack，gulp，grunt

* Grunt、Gulp是基于任务运行的工具，自动执行指定的任务，就像流水线，把资源放上去然后通过不同插件进行加工。
* Webpack是基于模块化打包的工具，自动化处理模块,webpack把一切当成模块，当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

## 6. bundle，chunk，module 分别是什么?

* bundle：是由webpack打包出来的文件
* chunk：代码块，一个chunk由多个模块组合而成，用于代码的合并和分割
* module：是开发中的单个模块，在webpack的世界，一切皆模块，一个模块对应一个文件，webpack会从配置的entry中递归开始找出所有依赖的模块

## 7. Loader和Plugin的不同？

不同的作用：

* Loader直译为"加载器"。Webpack将一切文件视为模块，但是webpack原生是只能解析js文件，如果想将其他文件也打包的话，就会用到loader。 所以Loader的作用是让webpack拥有了加载和解析_非JavaScript文件_的能力。
* Plugin直译为"插件"。Plugin可以扩展webpack的功能，让webpack具有更多的灵活性。 在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

不同的用法：

* Loader在module.rules中配置，也就是说他作为模块的解析规则而存在。 类型为数组，每一项都是一个Object，里面描述了对于什么类型的文件（test），使用什么加载(loader)和使用的参数（options）
* Plugin在plugins中单独配置。类型为数组，每一项是一个plugin的实例，参数都通过构造函数传入。

## 8. 编写 loader 和 plugin 的思路

* 编写Loader时要遵循单一原则，每个Loader只做一种"转义"工作。每个Loader的拿到的是源文件内容（source），可以通过返回值的方式将处理后的内容输出，也可以调用this.callback()方法，将内容返回给webpack。还可以通过 this.async()生成一个callback函数，再用这个callback将处理后的内容输出出去。 此外webpack还为开发者准备了开发loader的工具函数集——loader-utils。
* webpack在运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。
