# JS 基础

## 0. JS 的运行机制
* JS 是单线程的，同一时间只能做一件事
* 代码执行的时候，会创建执行上下文，代码都是在执行上下文中运行的。
* 不同的函数会被压到执行栈中等待主线程执行，保证顺序
* 同步代码执行完之后，会把异步事件的回调函数放到不同的任务队列中等待执行。
* 宏任务队列和微任务队列，基于事件循环，会把不同的任务压到执行栈中等待执行。
* JS 事件循环，基于任务队列，宏任务、微任务，只有一个微任务队列，但是可以有 N个宏任务队列

## 0.5 执行上下文
执行上下文：执行上下文是评估和执行 JavaScript 代码的环境的抽象概念。每当 Javascript 代码在运行的时候，它都是在执行上下文中运行。

执行上下文的创建阶段，会分别生成变量对象，建立作用域链，确定this指向。

执行上下文的类型：全局执行上下文、函数执行上下文、eval 执行上下文

* 全局执行上下文 — 这是默认或者说基础的上下文，任何不在函数内部的代码都在全局上下文中。它会执行两件事：创建一个全局的 window 对象（浏览器的情况下），并且设置 this 的值等于这个全局对象。一个程序中只会有一个全局执行上下文。
* 函数执行上下文 — 每当一个函数被调用时, 都会为该函数创建一个新的上下文。每个函数都有它自己的执行上下文，不过是在函数被调用时创建的。函数上下文可以有任意多个。每当一个新的执行上下文被创建，它会按定义的顺序（将在后文讨论）执行一系列步骤。
* Eval 函数执行上下文 — 执行在 eval 函数内部的代码也会有它属于自己的执行上下文，但由于 JavaScript 开发者并不经常使用 eval，所以在这里我不会讨论它。

## 1. 闭包
> 一个可以访问外部作用域中变量的内部函数，就是闭包。

* 闭包只存储外部变量的引用，而不会拷贝这部分的值。
* 这些被引用的变量直到闭包被销毁时才会被销毁。
* 闭包的应用场景：装饰器、私有变量（避免使用全局变量，达到封装性）
* 闭包使得 timer 定时器，事件处理，AJAX 请求等异步任务更加容易。
* 作用域链（Scope Chain）：每一个作用域都有对其父作用域的引用。

## 2. 基础类型
* 原始类型 String,Number,Boolean,undefined,null,symbol,
* 复杂类型 Object,Function(最新的标准增加了 Function)
* 原始类型都是存在栈中，会被频繁调用，占据固定空间大小
* 复杂类型存储在栈和堆中，在栈中存放指针，在堆中存放数据

## 3. 原型链
* 每个构造函数都有 prototype 属性，指向原型对象
* 原型对象有 constructor 指向构造函数
* 由构造函数创建出来的实例，有 \__proto__ 属性指向原型对象
* 原型对象的 \__proto__ 指向原型对象的 \__proto__，最终指向 Object.prototype
* 构造函数也是对象，他的 \__proto__ 指向 Function.prototype
* Function.\__proto__ 指向 Function.prototype
* Object.\__proto__ 指向 Function.prototype
* Function.prototype.\__proto__ 指向 Object.prototype
* Object.prorotype.\__proto__ 指向 null

> 类是构造函数的语法糖

## 4. this 的指向
* 调用时才能确定 this 指向，谁调用指向谁
* 改变 this 的几种方式：call,apply,bind
* 箭头函数的 this

## 5. 深拷贝、浅拷贝
* 深拷贝：JSON.parse(JSON.stringfy({})), _.cloneDeep({})
* 浅拷贝：{...}, Object.assign()

## 6. 事件冒泡和事件代理

### 6.1 事件冒泡
浏览器事件的执行需要经过三个阶段，捕获阶段-目标元素阶段-冒泡阶段。
* 捕获：从外层到内层 window -> document -> html -> body -> 目标元素
* 冒泡：从内层到外层

### 6.2 addEventListener 的第三个参数
* element.addEventListener(event, function, useCapture)
* 第三个参数解释：事件是否在捕获或冒泡阶段执行。true: 捕获阶段即执行；false: 冒泡阶段再执行。

### 6.3 事件代理
> 如果要对一个列表下面的各个子元素增加事件监听，则会增加过多的事件监听。
> 因此可以将事件绑定在父节点上，在冒泡的时候就可以监听到。
> 再根据 target 找到实际被点击的元素，就可以达到预期的效果。

### 6.4 阻止冒泡
* 给子级加 event.stopPropagation() - 只阻止冒泡，不阻止本身的事件
* 在事件处理函数中返回 false - 阻止冒泡，同时也阻止事件本身(默认事件)
* event.target==event.currentTarget，让触发事件的元素等于绑定事件的元素，也可以阻止事件冒泡；

### 6.5 阻止默认事件
* event.prevevtDefault()
* 监听的回调函数返回 false

## 7. Object.defineProperty
```js
Object.defineProperty(object, key, descriptor) // configurable，enumable, writable 默认都是 false
```

## 8. new String('a')；String('a')；'a'
```js
const str = 'a' // 字符串 -- 原始资料类型，储存于栈中，向包装对象借属性和方法
const str1 = String('a') // 字符串 -- 原始资料类型，储存于栈中，向包装对象借属性和方法
const str2 = new String('a') // 对象 -- 包装对象，栈中存放堆指针，堆中存放内容
str === str1 // true
str === str2 // false
str1 === str2 // fase
```

## 9. JSON.stringfy & JSON.parse
```js
JSON.stringfy(object, replacer, space) // 对象，(函数: (key, value): any  | 数组 ['a, b, c']），缩进空格
JSON.parse(string[, receiver]) // 转换器, 如果传入该参数(函数)，可以用来修改解析生成的原始值，调用时机在 parse 函数返回之前。
```

## 10. 暂时性死区
带有`let`关键字（和`const`）的变量被提升，但是与`var`不同，它不会被***初始化***。 在我们声明（初始化）它们之前，无法访问它们。 这称为“暂时性死区”。 当我们尝试在声明变量之前访问变量时，JavaScript会抛出`ReferenceError: Cannot access 'name' before initialization`。

## 11.内置对象
* 全局变量：NaN, undefined
* 全局函数：parseInt(), parseFloat()
* 全局对象：Math, Date, Object, RegExp,

## 12.获取原型的方法
* obj.__proto__
* obj.constructor.prototype
* Object.getPrototypeOf(obj)

## 13.内存泄露
* 未使用的全局变量
* 未清除的定时器
* 闭包 私有变量
* 脱离 DOM 的引用：如绑定在 DOM 上的引用，但是 DOM 被删除了

## 14.arguments 对象
* 函数中传递的参数值的集合
* 有 length 属性
* 不是数组
* 可以使用 Array.prototype.slice.call(arguments) 来转化为数组
* 箭头函数没有 arguments 对象

## 15.函数式编程
* （通常缩写为FP）是通过编写纯函数，避免共享状态、可变数据、副作用 来构建软件的过程。
* 函数式编程是声明式 的而不是命令式 的，应用程序的状态是通过纯函数流动的

## 16.高阶函数
> 以函数为入参或者返回值为函数的函数

## 17.为什么函数是一等公民
> 在JavaScript中，函数不仅拥有一切传统函数的使用方式（声明和调用），而且可以做到像简单值一样:

* 赋值（var func = function(){}）、
* 传参(function func(x,callback){callback();})、
* 返回(function(){return function(){}})，

> 不仅如此，JavaScript中的函数还充当了类的构造函数的作用，同时又是一个Function类的实例(instance)。

## 18.正则表达式

### 基础语法

### 练习
```js
// ab-cd-ef -> abCdEf 
function camelcase(str) {
  return str.replace(/\-\w/g, (v) => v.replace(/\-/g, '').toUpperCase())
}
```

## 19.Promise
* Pormise 构造函数是同步执行的，then方法是异步执行的


# ES 新概念

## 1.Map
* Map: 类似于对象，但是它的 key 可以是对象。普通对象 key 必须是字符串。
* 方法：get、set、delete、has、clear、size
* WeakMap: key必须是对象。(null 除外)

## 2.Set
* Set: 类似于数组，但是 Set内不允许有重复的元素。（可以用于数组去重）。
* 方法：add、delete、has、clear、size
* WeakSet: 元素必须为对象。

## 3.Proxy 代理

- **get(target, propKey, receiver)**：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
- **set(target, propKey, value, receiver)**：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
- **has(target, propKey)**：拦截`propKey in proxy`的操作，返回一个布尔值。
- **deleteProperty(target, propKey)**：拦截`delete proxy[propKey]`的操作，返回一个布尔值。
- **ownKeys(target)**：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。
- **getOwnPropertyDescriptor(target, propKey)**：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
- **defineProperty(target, propKey, propDesc)**：拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
- **preventExtensions(target)**：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
- **getPrototypeOf(target)**：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
- **isExtensible(target)**：拦截`Object.isExtensible(proxy)`，返回一个布尔值。
- **setPrototypeOf(target, proto)**：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- **apply(target, object, args)**：拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
- **construct(target, args)**：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。

```js
// 代理示例
const handler = {
	set: () => console.log("Added a new property!"),
	get: () => console.log("Accessed a property!")
};

const person = new Proxy({}, handler);

person.name = "Lydia";
person.name;
```

与 Object.defineProperty 对比

* Object.definedProperty 的作用是劫持一个对象的属性，劫持属性的getter和setter方法，在对象的属性发生变化时进行特定的操作。而 Proxy 劫持的是整个对象。
* Proxy 会返回一个代理对象，我们只需要操作新对象即可，而 `Object.defineProperty` 只能遍历对象属性直接修改。
* Object.definedProperty 不支持数组，更准确的说是不支持数组的各种API，因为如果仅仅考虑arry[i] = value 这种情况，是可以劫持的，但是这种劫持意义不大。而 Proxy 可以支持数组的各种API。
* 尽管 Object.defineProperty 有诸多缺陷，但是其兼容性要好于 Proxy.

## 4.iterator 迭代器

> Iterator（迭代器）是一种接口，也可以说是一种规范。为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。

```js
// 迭代器语法
const obj = {
  [Symbol.iterator]:function(){}
  [Symbol.asyncIterator]: function*() { yield 1 yield 2}
}
```

> 迭代器的遍历方法是首先获得一个迭代器的指针，初始时该指针指向第一条数据之前，接着通过调用 next 方法，改变指针的指向，让其指向下一条数据

每一次的 next 都会返回一个对象，该对象有两个属性
* value 代表想要获取的数据
* done 布尔值，false表示当前指针指向的数据有值，true表示遍历已经结束
* Iterator 的作用有三个：
  * 为各种数据结构，提供一个统一的、简便的访问接口；
  * 使得数据结构的成员能够按某种次序排列；
  * ES6 创造了一种新的遍历命令for…of循环，Iterator 接口主要供for…of消费。

遍历过程：

* 创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。
* 第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。
* 第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。
* 不断调用指针对象的next方法，直到它指向数据结构的结束位置。


## 5.Generator
> Generator函数可以说是Iterator接口的具体实现方式。Generator 最大的特点就是可以控制函数的执行。
> Generator是一个迭代器生成函数，其返回值是一个迭代器（Iterator），可用于异步调用。

```js
async function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield Promise.resolve(i);
  }
}

(async () => {
  const gen = range(1, 3);
  for await (const item of gen) {
    console.log(item);
  }
})();
```

