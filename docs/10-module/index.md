# 前端模块化

* amd - require.js AMD 推崇依赖前置、提前执行 采用异步加载，模块的加载不影响后面的语句执行。所有依赖这个模块的语句，都定义在一个回调用，当模块加载完成后，执行这个回调。

* cmd - sea.js CMD推崇依赖就近、延迟执行。 在申明依赖的模块时会在第一之间加载并执行模块内的代码。 define([a,b,c,d,e], function(a,b,c,d,e) {// DO SOME THINGS)

* commonjs： nodejs 的规范，同步加载模块，是对模块的浅拷贝，在服务端，文件都在本地，IO 很快，不会有问题，但是在浏览器端，由于网络的问题，更合理使用异步加载

性能问题：运行时才会调用
es6： ES6的模块不是对象,在编译时就引入模块代码。(性能比 commonjs 好)

CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
ES6 Module是静态的，也就是说它是在编译阶段运行，和var以及function一样具有提升效果（这个特点使得它支持tree shaking）

自动采用严格模式（顶层的this返回undefined）

ES6 Module支持使用export {<变量>}导出具名的接口，或者export default导出匿名的接口
