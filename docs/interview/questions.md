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
