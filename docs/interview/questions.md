# 前端面试题收集

1. ('b' + 'a' + + 'a' + 'a').toLowerCase() 的输出是什么？

```js
// banana
('ba' + NaN + 'a').toLowerCase()
```

2. 连续赋值问题

```js
var a = {n: 1}; 
var b = a; 
a.x = a = {n: 2}; 
 
a.x; // 这时 a.x 的值是多少 undefined
b.x; // 这时 b.x 的值是多少 {n: 2}

// 解析：先从左到右解析各个引用，然后计算最右侧的表达式的值，最后把值从右到左赋给各个引用。
// 0. a = b = {n: 1}, 
// 1. 首先到 “a.x”的内存位置创建一个引用”ref2“，赋给undefined，并记住这个引用。
// 2. 重置 a 指针，指向地址 {n: 2}, a.x = undefined
// 3. 给 “ref“ 指向 {n: 2} => b.x = { n: 1, x: { n: 2 } }
```

3. 实现一个run方法，使得run(fucArr)能顺序输出1、2、3

```js
const fucArr = [
  next => { setTimeout(() => { console.log(1); next() }, 300),
  next => { setTimeout(() => { console.log(2); next() }, 200),
  next => { setTimeout(() => { console.log(3); next() }, 100)
]
// 解析
const fn = arr => { if (arr.length) { return arr.shift()(() => fn(arr)) } }
```

4. 实现JS限流调度器，方法add接收一个返回Promise的函数，同时执行的任务数量不能超过两个。

```js
class Scheduler {
  async add(promiseFunc: () => Promise<void>): Promise<void> {
  }
}

const scheduler = new Scheduler()
const timeout = (time) => {
  return new Promise(r => setTimeout(r, time))
}
const addTask = (time, order) => {
  scheduler.add(() => timeout(time))
    .then(() => console.log(order))
}

addTask(1000, 1)
addTask(500, 2)
addTask(300, 3)
addTask(400, 4)
// log: 2 3 1 4
```

解析：

```js
class Scheduler {
  // 一个数组记录所有的任务
  private queue: any[] = [];
  // 一个Set记录运行中的任务
  private run: Set<any> = new Set([]);

  async add(promiseFunc: () => Promise<void>): Promise<void> {
    this.queue.push(promiseFunc);
    return this.schedule();
  }

  private schedule(): Promise<any> {
    if (this.run.size < 2 && this.queue.length) {
      const task = this.queue.shift();
      const promise = task().then(() => {
        this.run.delete(promise);
      });
      this.run.add(promise);
      return promise;
    } else {
      return Promise.race(this.run).then(() => this.schedule());
    }
  }
}

```

5. `delete` 操作结果是什么?

```js
const name = "Lydia";
age = 21;

console.log(delete name);
console.log(delete age);

// 答案：false true
// 解析：delete 操作符返回一个布尔值： true指删除成功，否则返回false，但是通过 var, const 或 let 关键字声明的变量无法用 delete 操作符来删除。
// name变量由const关键字声明，所以删除不成功: 返回 false. 而我们设定age等于21时,我们实际上添加了一个名为age的属性给全局对象。对象中的属性是可以删除的，全局对象也是如此，所以delete age返回true.
```

6. `String` 与 `new String`

```js
const str = 'a' // 字符串 -- 原始资料类型，储存于栈中，向包装对象借属性和方法
const str1 = String('a') // 字符串 -- 原始资料类型，储存于栈中，向包装对象借属性和方法
const str2 = new String('a') // 对象 -- 包装对象，栈中存放堆指针，堆中存放内容
str === str1 // true
str === str2 // false
str1 === str2 // fase
```

7. 类型转换问题

```js
if ([] == false) {console.log(1);};  // true 1  [].toString() -> '' -> 0 -> false
if ({} == false ) {console.log(2);}; // false
if ([]) {console.log(3);}; // true 3
if ([1] == [1]) {console.log(4);}; // false
```

8. JSON.stringfy 第二参数

```js
const settings = {
  username: "lydiahallie",
  level: 19,
  health: 90
};

const data = JSON.stringify(settings, ["level", "health"]);
console.log(data); // {"level":19, "health":90}
// 解析：JSON.stringify的第二个参数是 替代者(replacer). 替代者(replacer)可以是个函数或数组，用以控制哪些值如何被转换为字符串。

// 如果替代者(replacer)是个 数组 ，那么就只有包含在数组中的属性将会被转化为字符串。在本例中，只有名为"level" 和 "health" 的属性被包括进来， "username"则被排除在外。 data 就等于 "{"level":19, "health":90}".

// 而如果替代者(replacer)是个 函数，这个函数将被对象的每个属性都调用一遍。 函数返回的值会成为这个属性的值，最终体现在转化后的JSON字符串中（译者注：Chrome下，经过实验，如果所有属性均返回同一个值的时候有异常，会直接将返回值作为结果输出而不会输出JSON字符串），而如果返回值为undefined，则该属性会被排除在外。
```

9. 类型转换

```js
const name = "Lydia Hallie";
const age = 21;

console.log(Number.isNaN(name));
console.log(Number.isNaN(age));

console.log(isNaN(name));
console.log(isNaN(age));

// 答案：false false true false
// 解析：通过方法 Number.isNaN，你可以检测你传递的值是否为 数字值 并且是否等价于 NaN。name 不是一个数字值，因此 Number.isNaN(name) 返回 false。age 是一个数字值，但它不等价于 NaN，因此 Number.isNaN(age) 返回 false.

// 通过方法 isNaN， 你可以检测你传递的值是否一个 number。name 不是一个 number，因此 isNaN(name) 返回 true. age 是一个 number 因此 isNaN(age) 返回 false.
```

10. 正则 & 原生方法实现 `1234567890 -> 1,234,567,890`

```js
// 正则
str.replace(/\B(?=(\d{3})+(?!\d))/g, ',')
// /\B(?=(\d{3})+(?!\d))/g : 正则匹配边界\B，边界后面必须跟着(\d{3})+(?!\d);
// (\d{3})+: 必须是一个或多个的3位数字；
// (?!\d)：第二步中的三位数字后面不允许跟数字；
// (\d{3})+(?!\d))：所有匹配的边界后面必须跟着3*n的数字

// 原生
(123456789).toLocaleString('en-US')
```
