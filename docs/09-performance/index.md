# 性能优化

## 0. 常见的性能优化

* 图片懒加载
* 资源 gzip 压缩
* cdn 加速
* 域名收敛
* 雪碧图
* css 放头部，js 放最后
* dns 优化：<link rel="dns-prefetch" href="http://bdimg.share.baidu.com" />

## 1. 组件库按需加载

* babel-plugin-import (andt 团队，把对组件库的引用改为对某个组件的引用)
* tree-shaking (es module，剔除没有用到的代码)

## 2. webpack 如何做动态加载

原理：动态创建 script 标签，通过 jsonp 去请求 chunk

* import() - ES module (推荐)
* require.ensure() - commonjs

```js
import('lodash').then(_ => {
  // Do something with lodash (a.k.a '_')...
})
require.ensure('a', function(require) {
  const b = require('b')
})
```

## 3. service worker

用于离线和缓存开发

监听 fetch 事件，缓存结果，在离线的情况可以直接使用缓存，而不需要依赖接口返回

服务线程，与主线程独立，不可访问 DOM 和 Window，不会阻塞主线程。

### 3.1 pwa

progressive web application : 渐进式网络应用

可以直接添加到手机桌面，利用 service work 缓存，可以实现消息推送等。

## 4. web worker

https://juejin.im/post/59c1b3645188250ea1502e46
https://juejin.im/post/5c10e5a9f265da611c26d634

html5 的规范中实现了 Web Worker 来引入 JavaScript 的 “多线程” 技术。

通过 postMessage 和主线程之间信息交互。

单独的线程计算。可用于：

* 懒加载
* 文本分析
* 流媒体数据处理
* canvas 图形绘制
* 图像处理
* 大数据处理

需要注意：

* 有同源限制
* 无法访问 DOM 节点
* 运行在另一个上下文中，无法使用Window对象
* Web Worker 的运行不会影响主线程，但与主线程交互时仍受到主线程单线程的瓶颈制约。换言之，如果 Worker 线程频繁与主线程进行交互，主线程由于需要处理交互，仍有可能使页面发生阻塞
* 共享线程可以被多个浏览上下文（Browsing context）调用，但所有这些浏览上下文必须同源（相同的协议，主机和端口号）

// main.js
var worker = new Worker('./worker.js');

worker 上下文： WorkerGlobalScope

1. self
2. location
3. close
4. importScripts
5. XMLHttpRequest
6. setTimeout/setInterval,addEventListener/postMessage

结束webWorker： worker.terminate() or self.close()

## 5. 首屏加载

### 5.1 白屏处理

> 白屏时间： 从路由改变 到 首次绘制内容

优化方案：

* html-webpack-plugin: 在html 中插入 loading 图，减少用户等待焦虑
* 伪服务端渲染: prerender-spa-plugin(基于 puppeteer)，打包预先执行打包文件，获取首屏 html
* 开启 http2: 二进制分帧，多路复用，头部表（头部复用）
* 开启浏览器缓存：
  * Expires: 响应头，代表该资源的过期时间。
  * Cache-Control: 请求/响应头，缓存控制字段，精确控制缓存策略。
  * If-Modified-Since: 请求头，资源最近修改时间，由浏览器告诉服务器。
  * Last-Modified: 响应头，资源最近修改时间，由服务器告诉浏览器。
  * Etag: 响应头，资源标识，由服务器告诉浏览器。
  * If-None-Match: 请求头，缓存资源标识，由浏览器告诉服务器。
* splitChunksPlugin: 抽取第三方库，打包成不同的 chunk
  * 问题：chunk id 是自增的，一旦项目引入新的依赖或者删除旧的依赖，chunk 的自增会被打乱，导致有些没有变的 chunk 名称改变。
  * 解决方案：HashedModuleIdsPlugin，使用非自增的方式进行 chunk id 命名。

### 5.2 首次有意义绘制（FMP）

> 白屏结束后，页面开始渲染，但是还只是会出现个别无意义的元素，比如下拉框、乱序的元素，导航等，导致页面闪烁，影响体验

优化方案：

* 骨架屏 （andt 内置）

### 5.3 可交互时间

> 有意义的内容渲染出来以后，用户会尝试页面交互，但这个时候页面并没有加载完毕，还会有JavaScript脚本在密集执行

解决方案：

* 减少脚本体积：tree-shaking(剔除代码中无用的代码)
  * 坑 1：必须使用 ES module，但是 babel 会默认开启 commonjs，需要手动关闭
  * 坑 2：第三方库不可控，尽量使用有 ES module 的第三方库
* polyfill 动态加载，不要打包
* 路由级别拆解代码 React.lazy
* 提高执行速度 - web worker

## 6 组件加载

* 组件懒加载 - code spliting import('./xx') & (React.lazy + Suspense)
* 组件预加载 - lazyWithPreload

```js
function lazyWithPreload(factory) {
  const Component = React.lazy(factory);
  Component.preload = factory;
  return Component;
}
const StockChart = lazyWithPreload(() => import("./StockChart"));
// when pointer hover - preload
StockChart.preload()
```

* keep-alive: 利用状态管理保存状态；display: none
