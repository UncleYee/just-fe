# React

## 0.浏览器的事件机制

### 0.1 拓展 - 浏览器事件
> 浏览器事件的执行需要经过三个阶段，捕获阶段-目标元素阶段-冒泡阶段。
* 捕获：从外层到内层
* 冒泡：从内层到外层

### 0.2 拓展 addEventListener 的三个参数
> element.addEventListener(event, function, useCapture);
> 第三个参数解释：事件是否在捕获或冒泡阶段执行。true: 捕获阶段即执行；false: 冒泡阶段再执行。

### 0.3 事件代理
> 如果要对一个列表下面的各个子元素增加事件监听，则会增加过多的事件监听。
> 因此可以将事件绑定在父节点上，在冒泡的时候就可以监听到。
> 再根据 target 找到实际被点击的元素，就可以达到预期的效果。

### 0.4 阻止冒泡
* 给子级加 event.stopPropagation() - 只阻止冒泡，不阻止本身的事件
* 在事件处理函数中返回 false - 阻止冒泡，同时也阻止事件本身(默认事件)
* event.target==event.currentTarget，让触发事件的元素等于绑定事件的元素，也可以阻止事件冒泡；

### 0.5 阻止默认事件
* event.preventDefault()
* 监听的回调函数返回 false

## 1.React 事件机制？
解释：React 所有的事件都不是绑定在 DOM 的，而是绑定在 Document 上，然后由统一的事件处理程序(dispatchEvent)来处理的。
同时也是基于浏览器的事件机制（冒泡），所有节点的事件都会在 document 上触发。

> 原生事件阻止冒泡肯定会阻止合成事件的触发,合成事件的阻止冒泡不会影响原生事件。

### 1.1 为什么把事件都绑定在 Document 上 (React 为什么要使用合成事件？)
* 减少内存消耗，提升性能，不需要注册那么多的事件了，一种事件类型只在 document 上注册一次
* 统一规范，解决 ie 事件兼容问题，简化事件逻辑
* 对开发者友好

### 1.2 合成事件
* 对原生事件的封装
* 对某些原生事件的升级和改造 (比如：input 输入框的 onChange 事件，原生只有当输入框失去焦点才会触发，但 React 会注册很多其他事件，如：focus、blur、change、click、keydown、keyup)
* 不同浏览器事件兼容的处理（比如：在非 IE 浏览器 addEventListener/removeEventListener，在 IE 浏览器 attachEvent/detachEvent）
* 合成事件回调函数中的参数 e, 不是原生事件对象，而是 React 包装过的事件对象，原生事件对象被放在了 e.nativeEvent 内

### 1.3 事件注册机制

事件注册的过程做两件事：事件注册、事件存储。
* 事件注册 - 组件挂载阶段，根据组件内的声明的事件类型-onClick，onChange 等，给 document 上添加事件 -addEventListener，并指定统一的事件处理程序 dispatchEvent。
* 事件存储 - 把 react 组件内的所有事件统一的存放到一个对象(Map)里，缓存起来，为了在触发事件的时候可以查找到对应的方法去执行。

事件存储详解：react 把所有的事件和事件类型及组件进行关联，关系保存在一个 Map 里面，在事件触发的时候根据 组件id 和 事件类型 查找对应的事件回调函数。

## 2.React 虚拟 DOM
Virtual DOM 其实就是 DOM 的抽象，本质上就是一个 JavaScript 对象，这个对象就是更加轻量级的对 DOM 的描述。（树状结构）

### 2.1 为什么要有虚拟 dom
> 直接操作 DOM 有性能问题，操作复杂，容易引起 bug，代码也难以维护。

## 3.React 的 diff 算法
事件复杂度：
* 老算法(递归)：O(n3) 将两个树的所有节点两两相比较，就是 n2 了，比较完之后还要进行一轮遍历，对树进行增删改的操作。
* 新算法：O(n) 所有节点按照层级只比较一次，比较完之后直接进行添加、移动、删除的操作。

新算法的实现：
* tree diff: 树的 diff，只比较同层级的节点，如果不同则直接替换节点
* component diff: 组件 diff, 如果是相同类型的组件 DOM 是相似的，则比较 element，如果不同类型的组件，直接替换
* element diff: 根据 key 的唯一标识，进行移动、删除、修改操作

## 4.React Fiber
> Fiber：一种将 recocilation (调和算法，递归 diff)，拆分成无数个小任务的算法；它随时能够停止，恢复。停止恢复的时机取决于当前的一帧（16ms）内，还有没有足够的时间允许计算。

React Fiber 是一种基于浏览器的单线程调度算法。React 16之前 ，reconcilation 算法实际上是递归，想要中断递归是很困难的，React 16 开始使用了循环来代替之前的递归。

fiber 改变了之前react的组件渲染机制，新的架构使原来同步渲染的组件现在可以异步化，可中途中断渲染，执行更高优先级的任务。释放浏览器主线程.

## 5.Time Slice (时间分片) - 就是 fiber
* React 在渲染（render）的时候，不会阻塞现在的线程
* 如果你的设备足够快，你会感觉渲染是同步的
* 如果你设备非常慢，你会感觉还算是灵敏的
* 虽然是异步渲染，但是你将会看到完整的渲染，而不是一个组件一行行的渲染出来
* 同样书写组件的方式

## 6.React setState
* state 不能直接修改
* setState 的表现可能是同步的，也可能是异步的
* setState 会合并更改

### 拓展：setState 到底是同步还是异步的？
* 在合成事件和勾子函数中是异步的，在 setTimeout 和原生事件中是同步的。
* setState 本身执行和代码都是同步的，只是钩子函数和合成事件的执行顺序是在更新之前，导致没办法立刻拿到更新后的值，因此表现出来异步。
* setState 批量更新优化也是建立在异步之上的，在原生事件和 setTimeout 中不会批量跟新，但是在钩子函数和合成事件中会合并，取最后一次执行。如果对不同的值进行 setState，则会在最后进行合并处理。

## 7.React 性能优化
* 减少重新 render 的次数。因为在 React 里最重(花时间最长)的一块就是 reconction(简单的可以理解为 diff)，如果不 render，就不会 reconction。
* 减少计算的量。主要是减少重复计算，对于函数式组件来说，每次 render 都会重新从头开始执行函数调用。
* 代码分割(code spliting)：
  * import('./xxx').then(() => {})
  * React.lazy + Suspense
```
const Home = React.lazy(() => import('./../home'))

render() {
  return (
    <Suspense fallback={<div>加载中</div>}>
      <Home />
    </Suspense>
  )
}
```

### 类组件优化方向
* shouldComponentUpdate
* pureComponent - 在 props 不变的情况下，不会再次渲染组件。class MyComponent extends React.PureComponent {}; 不能够在内部使用 shouldComponentUpdate

### 函数式组件优化方向
* React.memo - 在 props 不变的情况下，不会再次渲染组件。函数式组件专用
```js
// 基础用法
function Component(props) {
  // props 渲染
}
const MyComponent = React.memo(Component); // or export default React.memo(Component)
```
* React.memo 默认只会对 props 的复杂对象做浅层对比(引用)，不会对深层做比较。因此需要第二个参数来辅助对比
```js
function areEqual(preProps, nextProps) {
  // 判断，返回 boolean
}
export default React.memo(Component, areEqual);
```
* useCallback - 函数式组件每次重新渲染，传递给子组件的回调函数会被重新创建，导致 props 的改变，引起重新渲染。实际上函数的内容并没有变化。因此需要用 useCallback 来避免重新渲染。
```js
const callback = () => {
  dosomething(a)
}
// useCallback
const memoizedCallback = useCallback(callback, [a]) // 在 a 变化的时候，会重新生成 callback，触发子组件的渲染，否则不会渲染
```
* useMemo - 上面两种都是减少 Render，useMemo 是为了减少计算的量。
```js
const { a } = props;
function bigSum() {
  let count = a;
  for (let i = 0; i < 100000; i++) {
    count += i;
  }
  return count;
}
// useMemo
const memoizedValue = useMemo(bigSum, [a]) // 当 a 变化的时候，会重新计算，否则使用缓存的数据
```

## 8.React 生命周期

### React 16 之前
* componentWillMount()
* componentWillReceiveProps(nextProps)
* shouldComponentUpdate(nextProps, nextState)
* componentWillUpdate(nextProps, nextState)
* render()
* componentDidUpdate(prevProps, prevState)
* componentDidMount()
* componentWillUnmount()

### React 16 之后

废弃了：componentWillMount、componentWillReceiveProps、componentWillUpdate

* 挂载阶段：constructor、static getDerivedStateFromProps(nextProps, prevState)、render、componentDidMount
* 更新阶段：getDerivedStateFromProps、shouldComponentUpdate(nextProps, nextState)、render、getSnapshotBeforeUpdate(prevProps, prevState)、componentDidUpdate(prevProps, prevState, snapshot)
* 销毁阶段：componentWillUnmount

```js
class A {
  // 生命周期详解
  constructor(props) {
    super(props)
  }
  static getDerivedStateFromProps(nextProps, prevState) {
    // 静态方法，没有 this
    // 可以返回一个对象，用于更新当前的 state，如果不需要更新则返回 null
  }
  UNSAFE_componentWillReceiveProps(nextProps, prevState) {} // 即将删除
  UNSAFE_componentWillMount() {} // 即将删除，在 render 之前调用，setState 不会触发重新渲染
  shouldComponentUpdate(nextProps, nextState) {
    // 默认返回 true
    // 这里是优化的重点 需要判断 this.props vs nextProps， this.state vs nextState，已决定是否重新渲染
  }
  UNSAFE_componentWillUpdate(nextProps, nextState) {} // 即将删除，在上面返回 true 后，渲染前，会在这个生命周期
  render() {
     // 作为渲染用，纯函数，不能有副作用
  }
  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 触发时间: update发生的时候，在render之后，在组件dom渲染之前。
    // 这个方法必须和 componentDidUpdate 一起用，用来代替 componentWillUpdate
    // 使用场景：给一个列表新增了一个元素，需要计算在添加元素之后页面需要滚动到什么位置
  }
  componentDidUpdate(prevProps, prevState, snapshot) {
    // 渲染之后，根据 snapshot 进行一系列操作，比如页面滚动等
  }
  componentDidMount() {
    // 在渲染后可以获取到 DOM节点，进行操作，如果 setState 会触发 re-render
  }
}
```

## 9.React Hooks

### 9.1 hooks 怎么处理生命周期

`useEffect`给函数组件提供了副作用操作的能力，跟 class 组件中的 `componentDidMount`，`componentDidUpdate`，`componentWillUnmount`具有相同的用途，只不过被合并成一个 API `useEffect`。

### 9.2 特点
* 每一次渲染都有它自己的 Props and State。(每次 state 的变化都会引起函数组件被重新调用，state 和 props 都是和那一次渲染有关)
* 每一次渲染都有它自己的事件处理函数。(每次渲染后都会重新定义事件处理函数)
* 每次渲染都有它自己的 useEffects。(每一个版本的 useEffect 都只能看到那一个版本的 state，props)
* 如果需要在 useEffect 捕获未来的值，一种方法是 useRef。
* useEffect 的回调函数，可以返回一个函数，用于清理旧的 Effect 时执行。
  * 1. 渲染下一次的 UI （如： id: 20）
  * 2. 浏览器绘制 
  * 3. 清除上一次的effect {id : 10}。effect的清除并不会读取“最新”的props。它只能读取到定义它的那次渲染中的props值
  * 4. 执行新的 effect {id: 20}
* useEffect 不能欺骗，必须明确的告知依赖。
```js
// 告知依赖
useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, [count]);
// 移除依赖的方式，通过函数 setCount
useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
// 但是如果需要依赖其他参数，则要写成
useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]);

// 但是上面这种看起来还是比较怪，这里我们就需要用到 useReducer
// 当你想更新一个状态，并且这个状态更新依赖于另一个状态的值时，你可能需要用useReducer去替换它们。
function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
const initialState = { count: 0, step: 1 }
const [state, dispatch] = useReducer(reducer, initialState);
// useReducer(reducer, initialState)
// 另一种方法：useReducer(reducer, initialState, init) 
// 第二个参数作为第三个参数（函数）的入参，返回值作为 state

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' }); // Instead of setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```
* 把函数移到 useEffect 里面，就不用考虑太多的隐式依赖
```js
// 问题复现：
function SearchResults() {
  const [query, setQuery] = useState('react');
  // Imagine this function is also long
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }
  // Imagine this function is also long
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }
  useEffect(() => {
    fetchData();
  }, []); // 其实这里有隐式依赖 query 的值，但是因为函数被写到外民容易忽略
}
// 解决问题：
useEffect(() => {
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }
  fetchData();
}, [query]); // ✅ Deps are OK
```
* 如果考虑到复用，有些函数需要放到 useEffect 外面，则可以这么做
```js
// 1 把函数放到函数式组件外面
// ✅ Not affected by the data flow
function getFetchUrl(query) {
  return 'https://hn.algolia.com/api/v1/search?query=' + query;
}
function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK
  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK
}
// 2 useCallback
function SearchResults() {
  // ✅ Preserves identity when its own deps are the same
  const getFetchUrl = useCallback((query) => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, [query]);  // ✅ Callback deps are OK
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK
  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK
}
```

## 10. redux 概念
* store: 保存数据的容器，整个应用只有一个 store，保存了 state 数据
* action: state 的变化会导致 UI 的变化，用户通过 View 发出 action 通知，表示 state 应该要发生变化了。
* action creator: view 要发送多少种消息，就会有多少种 Action, 手写会很麻烦，所以需要一个函数来生成 Action，这个函数就是 action creator。
* reducer: reducer 是一个函数，接受 action 和当前 state 作为参数，返回一个新的 state。
* dispatch: 是 view 发出 action 的唯一方法

### 10.1 手写 redux

redux 三大设计原则：
* 单一数据源(所有 state 都只保存在一个 store)
* 状态是只读的(通过 store.getState 获取 state, 通过 dispatch 派发 action 更改 state)
* 使用纯函数写 reducer,(同样输入同样输出)

```js
/**
 * 
 * @param {*} reducer   //reducer
 * @param {*} initState    //初始状态
 * @param {*} middleware   //中间件
 */
const createStore = (reducer, initState, enhancer) => {
  let initialState;       //用于保存状态
  let currentReducer = reducer;        //reducer
  let listenerQueue = []; //存放所有的监听函数
  let isDispatch = false;

  if(initState){
    initialState = initState;
  }

  if(enhancer){
    return enhancer(createStore)(reducer,initState);
  }
  /**
   * 获取Store
   */
  const getState = () => {
    //判断是否正在派发
    if(isDispatch){
      throw new Error('dispatching...')
    }
    return initialState;
  }

  /**
   * 派发action 并触发所有的listeners
   * @param {*} action 
   */
  const dispatch = (action) => {
    //判断是否正在派发
    if(isDispatch){
      throw new Error('dispatching...')
    }
    try{
      isDispatch = true;
      initialState = currentReducer(initialState,action);
    }finally{
      isDispatch = false;
    }
    //执行所有的监听函数
    for(let listener of listenerQueue){
      listener.apply(null);
    }
  }
  /**
   * 订阅监听
   * @param {*} listener 
   */
  const subscribe = (listener) => {
    listenerQueue.push(listener);
    //移除监听
    return function unscribe(){
      let index = listenerQueue.indexOf(listener);
      let unListener = listenerQueue.splice(index,1);
      return unListener;
    }
  }

  /**
   * 替换reducer
   * @param {*} reducer 
   */
  const replaceReducer = (reducer) => {
    if(reducer){
      currentReducer = reducer;
    }
    dispatch({type:'REPLACE'});

  }
  dispatch({type:'INIT'});
  return {
    getState,
    dispatch,
    subscribe,
    replaceReducer
  }
}

export default createStore;
```

### 10.2 react-redux 是如何工作的？
* Provider: 从应用的最外部包装整个应用，并向 connect 模块传递 store
* connect: 负责连接 React 和 redux（connect 是一个高阶组件）
  * 获取 state: 通过 context 获取 provider 中的 store，通过 store.getState() 获取整个 store tree 上的所有 state
  * 包装原有组件: 将 state 和 action 通过 props 传入到组件内部，通过 mapStateToProps，mapDispatchToProps 与组件上原有的 props 合并。
  * 监听 store tree 变化: connect 中缓存了 store tree 中的 state, 通过比较当前 state 和变更前的 state 来判断是否需要调用 this.setState 来触发 Connect 及其子组件的重新渲染。
```js
export default class Provider extends Component {
  getChildContext() { // 自上而下传递 context
    return {store: this.props.store}
  }

  constructor() {
    super()

    this.state = {}
  }

  render() {
    return this.props.children
  }
}

// 一个高阶组件 装饰器写法
const connect = (mapStateToProps, mapDispatchToProps) => (WrappedComponent => {
  class Connect extends Component {
    constructor() {
      super()
      this.state = {}
    }
    componentWillMount() {
      this.unSubscribe = this.context.store.subscribe(() => {
        this.setState(mapStateToProps(this.context.store.getState())) // 从 provider 取 context
      })
    }
    componentWillUnmount() {
      this.unSubscribe()
    }
    render() {
      return <WrappedComponent 
                {...this.state}
                {...mapDispatchToProps(this.context.store.dispatch)}
              />
    }
  }
  return Connect
})
```

### 10.3 redux 异步中间件
* redux-thunk
* redux-saga

### 10.4 redux 做状态管理和发布订阅有什么区别？
> redux 也是一个发布订阅，但是 redux 可以做到数据的可预测（只有 action 发出通知）和可回溯（reducer 对 action 做出响应，返回新的 state）
> action: { type: '', payload: {text: 111, b: 222}}

## 11. mobx

### 概念
* 定义状态并使其可观察 @observable (被观察者)
* 创建视图以响应状态的变化 @observer (观察者)
* 更改状态
* 观察者模式

### 原则
action -> state -> view

mobx 原理：
* mobx4: Object.defineProperty
* mobx5: Proxy

### mobx 和 redux 的区别
* redux 将数据存在单一 store, mobx 将数据分散在多个 store
* redux 使用 plain object 保存数据，需要手动处理变化后的操作，mobx 使用 observable 保存数据，数据变化后自动响应操作。
* redux 状态只读，不能直接修改，需要通过 reducer，mobx 可以直接修改。
* redux 使用起来比较复杂，需要一系列的中间件处理异步和副作用，mobx 本身有很多抽象，使用起来比较方便。
* redux 可以追溯数据的变化，mobx 的抽象封装导致调试比较困难，难以预测。
* redux 面向函数式编程，Mobx 面向对象

## 11. setState 到底是同步还是异步？
* setState只在合成事件和钩子函数中是“异步”的，在原生事件和setTimeout 中都是同步的。
* 合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值。
* setState 的批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和setTimeout 中不会批量更新。

## 12. React 如何实现跨组件通信？
* 父 -> 子：props
* 子 -> 父：props + callback
* 兄弟：通过父组件
* 跨层级：Context
* Event，发布订阅模式
* 全局 Store：redux、mobx

## 13.组件复用

### 13.1 mixin - 已被官方抛弃
* ES6 class 不支持 mixin
* 存在隐式依赖，(在 mixin 中使用 componentDidMount 等)
* 多个 mixin 之间可能存在冲突：定义相同的 state
* 隐式依赖导致依赖关系不透明，维护成本上升

### 13.2 HOC (高阶组件)
优点：
* 通过外层 props 影响内部 state，不直接修改 state，降低耦合度
* 天然的层级结构，相比 mixin 降低复杂度
* 使用 @ 装饰器，使用方便

缺点：
* HOC 外无法访问子组件的 state，无法通过 shouldComponentUpdate 优化（ pureComponent 可以解决此问题）
* 命名冲突：如果高阶组件多次嵌套,没有使用命名空间的话会产生冲突,然后覆盖老属性
* 不可见性: HOC相当于在原有组件外层再包装一个组件,你压根不知道外层的包装是啥,对于你是黑盒
* Ref 传递问题，Ref 传递被隔断（React.forwardRef 可以解决此问题)
```js
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;

      // 将自定义的 prop 属性 “forwardedRef” 定义为 ref
      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  // 注意 React.forwardRef 回调的第二个参数 “ref”。
  // 我们可以将其作为常规 prop 属性传递给 LogProps，例如 “forwardedRef”
  // 然后它就可以被挂载到被 LogProps 包裹的子组件上。
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```

### 13.3 render props
优点
* 可以解决上面所有问题

缺点：
* 使用繁琐，HOC 只需要一行代码，装饰器写法
* 嵌套过深，虽然摆脱了组件嵌套，但是转化为回调函数嵌套

### 13.4 Hooks
优点：
* 简洁：解决了嵌套的问题
* 解耦：UI 和状态分离
* 组合：Hooks 可以组合，形成新的 hooks，千变万化
* 函数友好：解决了 this 问题；降低代码复用成本；分割在不同声明周期中的代码

缺点：
* 新的学习成本
* 写法的限制：不能出现在循环和判断中
* 破坏了 purecomponent 和 React.memo 浅比较的性能优化效果。（为了取最新的props和state，每次render()都要重新创建事件处函数）
* 在闭包场景下可能取到旧的 props 和 state
* 内部实现不直观，依赖一份可变的全局状态
* React.memo 并不能完全替代 shouldComponentUpdate (因为拿不到 state change，只针对 props change）