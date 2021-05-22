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
// a.x指向的是b.x的位置，但是a被重新赋值了，所以会有上面的答案
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
