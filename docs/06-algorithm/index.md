## 算法篇

### Hash表 ？
* hash 表结构
* hash 函数类型
* 解决冲突的办法


## 0. 时间复杂度、空间复杂度
* 时间复杂度：花费时间的大小：O(1) O(n) O(n2)  O(nLogn) O(n3)
* 空间复杂度：占用空间的大小：O(1) O(n) O(n2)

## 1. 递归实现 fill -- 递归
```js
function fill(n, m) {
  n--;
  if (n > 0) {
    return [m].concat(fill(n, m))
  } else {
    return m
  }
}

console.log(fill(3, 4))    // [4,4,4]
```

## 2. 大数相加
```js
function bigAdd(a, b) {
  const stringA = a.toString()
  const stringB = b.toString()

  const arrayA = stringA.split('')
  const arrayB = stringB.split('')

  // tip： 这里可以考虑用 String.padStart(10, '0') 来向前面补全
  const zeroLength = Math.abs(arrayA.length - arrayB.length)
  if (zeroLength) {
    for (let i = 0; i < zeroLength; i++) {
      (arrayA.length > arrayB.length ? arrayB : arrayA).unshift('0')
    }
  }

  const result = []
  let tag = false
  for (let i = arrayA.length - 1; i >= 0; i--) {
    const currentA = +arrayA[i]
    const currentB = +arrayB[i]
    const sum = currentA + currentB + (tag ? 1 : 0)
    if (sum < 10) {
      result.unshift(sum.toString())
      tag = false
    } else {
      result.unshift((sum - 10).toString())
      tag = true
    }
    if (i === 0) {
      result.unshift(1)
    }
  }

  const total = result.join('')
  return total;
}
```

## 3.爬楼梯（递归 or 动态规划） -- 递归
```js
// 爬楼梯 递归
function getWays(n) {
  if (n < 1 ) return 0;
  if (n === 1) return 1;
  if (n === 2) return 2;

  return getWays(n - 1) + getWays(n - 2)
}

// 爬楼梯 动态规划
function getWays(n) {
  if (n < 1) return 0;
  if (n == 1) return 1;
  if (n == 2) return 2;

  // a保存倒数第二个子状态数据，b保存倒数第一个子状态数据， temp 保存当前状态的数据
  let a = 1, b = 2;
  let temp;
  for (let i = 3; i <= n; i++) {
    temp = a + b;
    a = b;
    b = temp; 
  }
  return temp; 
}
```

## 4. 反转单链表
```js
function reverse(head) {
  if (!head || !head.next) return head

  let pre = null
  let current = head
  let next
  while(current) {
    next = current.next
    current.next = pre
    pre = current
    current = next
  }
  return pre
}
```

## 5. 判断回文数
```js
function judge(num) {
  const strA = num.toString()
  const strB = strA.split('').reverse().join('')
  return strA === strB;
}
```

## 6.判断是否对称二叉树 
```js
// 1 -- 递归
function isSymmetric(root) {
  if (root === null) {
    return true;
  }
  return help(root.left, root.right)
}

function help(left, right) {
  if (left === null && right === null) {
    return true
  }
  if (left === null || right === null) {
    return false
  }
  return left.val === right.val && help(left.left, right.right) && help(left.right, right.left)
}
// 2 -- 动态规划
function isSymmetric(root) {
  const queue = [root, root]
  while (queue.length) {
    const l = queue.shift()
    const r = queue.shift()
    if (l === null && r === null) return true
    if (l === null || r === null) return false
    if (l.val !== r.val) return false
    queue.push(l.lfet)
    queue.push(r.right)
    queue.push(l.right)
    queue.push(r.left)
  }
  return true
}
```

## 7.二叉树的层次遍历
```js
function test(root) {
  if (root === null) return
  const list = [root]
  const result = []
  while(list.length) {
    const temp = list.shift()
    result.push(temp.value)
    const left = temp.left
    const right = temp.right
    if (left) { 
      list.push(left)
    }
    if (right) {
      list.push(right)
    }
  }
  return result
}
```

## 8.二叉树的查找和为n的路径
```js
// 思路，找到所有的路径，存下来，计算路径的值是否等于指定的值，so easy
function find(root, sum) {
  const lists = [] // 二维数组，记录所有路径数组
  function findPaths(node, path) {
    if (node.left) {
      findPaths(node.left, [...path, node.left.value])
    }
    if (node.right) {
      findPaths(node.right, [...path, node.right.value])
    }
    if (!node.left && !node.right) {
      lists.push(path)
    }
  }
  findPaths(root, [root.value])
  
  const result = lists.find(item => item.reduce((a, b) => a + b) === sum)

  return result
}
// 递归实现
function find2(root, sum, path = []) {
  const left = root.left
  const right = root.right
  const value = root.value
  if (sum - value === 0 && !left && !right) {
    return [...path, value]
  }
  if (sum - value > 0) {
    if (left || right) {
      return (left && find2(left, sum - value, [...path, value])) || (right && find2(right, sum - value, [...path, value]))
    }
  }
}
```

## 9.36进制加法
```js
const chartSet = ['0', '1', '2', ...., 'x', 'y', 'z']
function add(string1, string2) {
  // TODO: xxx
  return ''
}
```

## 10.大数相乘
```js
```

## 11.扑克牌中的顺子

从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

输入: [1,2,3,4,5]

输出: true

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
// 可以优化
var isStraight = function(nums) {
  if (!Array.isArray(nums)) return false
  if (nums.length !== 5) return false
  const zeroList = nums.filter(item => item === 0) // 0 的数组
  const listWithoutZero = nums.filter(item => item !== 0).sort((a, b) => a - b) // 非 0 的数组
  const resultList = [listWithoutZero[0]] // 初始化 resultList,
  listWithoutZero.shift() // 取出非 0 数组的第一个数
  while (zeroList.length || listWithoutZero.length) { // 循环，只有两个数组都为空 才结束
    if (listWithoutZero.length) { // 主要判断非 0 数组是否为空
      if (listWithoutZero[0] === resultList[resultList.length - 1] + 1) { // 非 0 数组不为空，判断大小，取数
        resultList.push(listWithoutZero[0])
        listWithoutZero.shift()
      } else if (zeroList.length) { // 非 0 数组取数失败，0 来补位
        resultList.push(resultList[resultList.length - 1] + 1)
        zeroList.shift()
      } else { // 没有补位且大小不对，直接 false
        return false
      }
    } else { // 只要非 0 数字为空了 ，一定是顺子
      return true
    }
  }
  return true // 走出循环了 也一定是顺子
};
```

## 12. 请用算法实现，从给定的无序、不重复的数组data中，取出n个数，使其相加和为sum。并给出算法的时间/空间复杂度 。(不需要找到所有的解，找到一个解即可) -- 递归
```js
function getResult(data, n, sum) {
  if (n === 0 && sum === 0) {
    return true
  }
  if (n < 0) {
    return false
  }
  if (n > 0 && sum > 0) {
    for(var i = 0; i < data.length; i++){
      var temp = data.slice(i + 1, data.length); // shift
      return getRsult(temp, n-1, sum-data[i]) || getRsult(temp, n, sum); // 
    }
  }
}
```

## 13. 组合 从 1...n 中,选出 k 个数两两组合 

```js
// combine(4, 2) -> [[1,2], [1,3], [1,4], [2, 3], [2, 4], [3,4]]
var combine = function(n, k) {
  var result = [];
  var subresult = [];
  function combineSub(start,subresult){
    //terminator
    if (subresult.length == k) {
      result.push(subresult.slice(0));
      return;
    }
    for (var i= start; i<=n; i++){
      subresult.push(i);
      combineSub(i+1,subresult);
      subresult.pop();
    }
  }
  combineSub(1,subresult);
  return result;
};
```