# 设计模式

## 0. 设计原则

* 单一职责原则：一个程序只做好一件事
* 开放/封闭原则：对扩展开放，对修改封闭
* 里氏替换原则：子类能覆盖父类
* 接口隔离原则：保持接口的单一独立
* 依赖倒转原则：面向接口编程，依赖于抽象而不依赖于具体实现

## 1. 工厂模式

> 工厂模式定义一个用于创建对象的接口，这个接口由子类决定实例化哪一个类。
```js
class Product {
    constructor(name) {
        this.name = name
    }
    init() {
        console.log('init')
    }
    fun() {
        console.log('fun')
    }
}

class Factory {
    create(name) {
        return new Product(name)
    }
}

// use
let factory = new Factory()
let p = factory.create('p1')
p.init()
p.fun()
```

## 2. 单例模式

> 保证只有一个实例

```js
// 单例构造器
const FooServiceSingleton = (function () {
  // 隐藏的Class的构造函数
  function FooService() {}

  // 未初始化的单例对象
  let fooService;

  return {
    // 创建/获取单例对象的函数
    getInstance: function () {
      if (!fooService) {
        fooService = new FooService();
      }
      return fooService;
    }
  }
})();
```

## 3. 适配器模式

## 4. 装饰器模式

## 5. 代理模式

* 是为一个对象提供一个代用品或占位符，以便控制对它的访问
* 应用场景：DOM事件代理

## 6. 外观模式

> 为子系统中的一组接口提供一个统一的高层接口，使子系统更容易使用

```js
// 兼容浏览器绑定
let addMyEvent = function (el, ev, fn) {
  if (el.addEventListener) {
    el.addEventListener(ev, fn, false)
  } else if (el.attachEvent) {
    el.attachEvent('on' + ev, fn)
  } else {
    el['on' + ev] = fn
  }
}; 
```

## 7. 观察者模式/发布订阅模式

### 发布订阅

* 订阅者把自己想订阅的事件注册到调度中心，当该事件触发时候，发布者发布该事件到调度中心，由调度中心统一调度订阅者注册到调度中心的处理代码。
* 发布者 和 订阅者，需要一个中间人来管理发布和注册的事件，观察者和发布者都不知道对方的存在，他们之间是异步的。

### 观察者

* 目标和观察者是基类，目标提供维护观察者的一系列方法，观察者提供更新接口。具体观察者和具体目标继承各自的基类，然后具体观察者把自己注册到具体目标里，在具体目标发生变化时候，调度观察者的更新方法。
* 观察者 需要注册到被观察者内，被观察者会主动触发观察者更新，同步的。

```js
// 被观察者
function Subject() {
  this.observers = [];
}

Subject.prototype = {
  // 订阅
  subscribe: function (observer) {
    this.observers.push(observer);
  },
  // 取消订阅
  unsubscribe: function (observerToRemove) {
    this.observers = this.observers.filter(observer => {
      return observer !== observerToRemove;
    })
  },
  // 事件触发
  fire: function () {
    this.observers.forEach(observer => {
      observer.call();
    });
  }
}
```

## 9. 迭代器模式

> 迭代器模式简单的说就是提供一种方法顺序一个聚合对象中各个元素，而又不暴露该对象的内部表示。

```js
const item = [1, 'red', false, 3.14];

function Iterator(items) {
  this.items = items;
  this.index = 0;
}

Iterator.prototype = {
  hasNext: function () {
    return this.index < this.items.length;
  },
  next: function () {
    return this.items[this.index++];
  }
}
```
