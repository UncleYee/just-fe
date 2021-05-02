# TypeScript

## 1.TS 有什么优势？
* 更好的可维护性和可读性（与别人协同开发，或者维护以前的老代码，没有文档要硬着头皮读代码）
* 引入了静态类型声明，不需要太多的注释和文档，大部分的函数看类型定义就知道如何使用了
* 在编译阶段就能发现大部分因为变量类型导致的错误。

## 2.和 JS 的区别？
* 需要类型定义（不定义会自动进行类型推论）
* 引入第三方库需要声明文件
* 新的类型：接口、枚举、元组、泛型、never、void
* 断言 <> 或 as

## 3.TS 的使用技巧？
* 函数重载
* 抽象方法抽象类
* schema 定义（数据结构的定义）
* 装饰器
```ts
// 重载 padding
function padding(allNumber: number)
function padding(topAndBottom: number, leftAndRight: number)
function padding(top: number, right: number, bottom: number, left: number)
function padding(a: number, b?: number, c?: number, d?: number) {
  // do padding
}
```

## 4.Type 和 interface 有什么区别？ **高频考点**
* Type 是类型别名，interface 是接口类型。
* 两者都可以用来描述对象的形状或功能，但是语法不同。
```ts
interface Point {
  x: number;
  y: number;
}

interface SetPoint {
  (x: number, y: number): void;
}

type Point = {
  x: number;
  y: number;
};

type SetPoint = (x: number, y: number) => void;
```
* 与接口类型不同，类型别名也可以用于其他类型，如并集、元组等。
```ts
// primitive
type Name = string;
// object
type PartialPointX = { x: number; };
type PartialPointY = { y: number; };
// union
type PartialPoint = PartialPointX | PartialPointY;
// tuple
type Data = [number, string];
```
* 两者都可以扩展，但是语法不同。interface 通过继承，Type 通过 &。
```ts
interface PartialPointX { x: number; }
interface Point extends PartialPointX { y: number; }

type PartialPointX = { x: number; };
type Point = PartialPointX & { y: number; };
```
* interface 可以被多次定义，将被视为同一个接口，所有的成员都会被合并。
```ts
// interface Point { x: number; y: number; }
interface Point { x: number; }
interface Point { y: number; }

const point: Point = { x: 1, y: 2 };
```

## 5.装饰器相关

### 5.1 类装饰器
类装饰器表达式会在运行时当作函数被调用，类的`构造函数`作为其唯一的参数。
```ts
@sealed
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  greet() {
    return "Hello, " + this.greeting;
  }
}

function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}
```

### 5.2 方法装饰器
方法装饰器表达式会在运行时当作函数被调用，传入下列3个参数：
* 1.对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
* 2.成员的名字。
* 3.成员的属性描述符。
```ts
function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}
```

### 5.3 访问器装饰器
访问器装饰器表达式会在运行时当作函数被调用，传入下列3个参数：
* 1.对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
* 2.成员的名字。
* 3.成员的属性描述符。
```ts
class Point {
  private _x: number;
  constructor(x: number, y: number) {
    this._x = x;
  }

  @configurable(false)
  get x() { return this._x; }
}

function configurable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.configurable = value;
  };
}
```

### 5.4 属性装饰器
属性装饰器表达式会在运行时当作函数被调用，传入下列2个参数：
* 1.对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
* 2.成员的名字。
```ts
class Greeter {
  @format("Hello, %s")
  greeting: string;
}
```

### 5.5 参数装饰器
参数装饰器表达式会在运行时当作函数被调用，传入下列3个参数：
* 1.对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
* 2.成员的名字。
* 3.参数在函数参数列表中的索引。
```ts
class Greeter {
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }

  @validate
  greet(@required name: string) {
    return "Hello " + name + ", " + this.greeting;
  }
}

function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  // TODO...
}
```

## 6.代码相关

### 6.1 Pick，Record，Required，Partical，Readonly，Omit, Some
```ts
interface Person {
  name: string;
  age?: number;
}

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
}
// type person5 = Pick<Person, "name">;
// person5 === {name: string}

type Record<K extends string, T> = {
  [P in K]: T;
}
// type person6 = Record<'name' | 'age', string>
// person6 === {name: string; age: string}

type Required<T> = {
  [P in keyof T]-?: T[p];
}
// type person3 = Required<Person>;
// person3 === {name: string; age: number}

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
}
// type person4 = Readonly<Person>;
// person4 === {
//        readonly name: string;
//        readonly age?: number;
//  }

type Partial<T> = {
  [P in keyof T]?: T[P];
}
// type person2 = Partial<Person>;
// person2 === {name?: string; age?: number}


type Exclude<T, U> = T extends U ? never : T // 相当于是过滤出 T 中不在 U 的部分
//const a: Exclude<'a' | '1' | '2', 'a' | 'b' | 'c'> = '1' // true -> '1' | '2'


type Omit<T, K> = Pick<T, Exclude<keyof T, K>> 
// type person5 = Omit<Person, 'age'>
// person5 === {
//   name: 'string'
// }

type Some<T, K extends keyof T> = Omit<T, K> & { [P in K]?: T[P] }
//A {
//   a: string,
//   b: number,
//   c: object
// }

// =>

// {
//   a: string,
//   b: number,
//   c?: object
// }

// const a: Some<A, 'c'>

```

### 6.2 接口
```ts
interface Crazy {
  new (): {
    hello: number;
  };
}

class CrazyClass implements Crazy {
  constructor() {
    return { hello: 123 }
  }
}
```