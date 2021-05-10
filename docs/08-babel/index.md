# babel

## 1.babel 原理

* 解析: 将代码(其实就是字符串)转换成 AST(抽象语法树)
* 转换: 访问 AST 的节点进行变换操作生成新的 AST
* 生成: 以新的 AST 为基础生成代码

AST: 抽象语法树

## 2 ES6 转译过程

* ES6 代码输入
* babylon 进行解析得到 AST
* plugin 用 babel-traverse 对 AST 树进行遍历转译，得到新的 AST 树
* 用 babel-generator 通过 AST 树生成 ES5 代码

## 3. 代码如何转化成 AST ？

词法分析（Tokenizer - 词法分析器） & 语法分析

* 词法分析: 将代码(字符串)分割为token流,即语法单元成的数组
  * 数字：JavaScript 中的科学记数法以及普通数组都属于语法单元.
  * 括号：『(』『)』只要出现,不管任何意义都算是语法单元
  * 标识符：连续字符,常见的有变量,常量(例如: null true),关键字(if break)等等
  * 运算符：+、-、*、/等等
  * 当然还有注释,中括号等
* 语法分析: 分析token流(上面生成的数组)并生成 AST

## 5. babel 插件

Babel的插件模块需要你暴露一个function，function内返回visitor

```js
var babel = require('babel-core');
var t = require('babel-types');

const visitor = {
  BinaryExpression(path) {
    const node = path.node;
    let result;
    // 判断表达式两边，是否都是数字
    if (t.isNumericLiteral(node.left) && t.isNumericLiteral(node.right)) {
      // 根据不同的操作符作运算
      switch (node.operator) {
        case "+":
          result = node.left.value + node.right.value;
          break
        case "-":
          result = node.left.value - node.right.value;
          break;
        case "*":
          result =  node.left.value * node.right.value;
          break;
        case "/":
          result =  node.left.value / node.right.value;
          break;
        case "**":
          let i = node.right.value;
          while (--i) {
            result = result || node.left.value;
            result =  result * node.left.value;
          }
          break;
        default:
      }
    }

    // 如果上面的运算有结果的话
    if (result !== undefined) {
      // 把表达式节点替换成number字面量
      path.replaceWith(t.numericLiteral(result));
    }
  }
};

module.export = function(babel){
  return {
    visitor:{
    }
  }
}
```
