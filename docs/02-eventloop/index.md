# 底层概念

## 1.浏览器的 Eventloop

javascript 有一个主线程 main thread，和调用栈 call-stack 也称之为执行栈。所有的任务都会放到调用栈中等待主线程来执行。

同步任务 & 异步任务：同步任务说白了就是主线程来执行的时候立即就能执行的代码，异步任务就会被扔到任务队列中去等待被执行。

打比方：！！！！
* 我六点钟下班了，可以安排下自己的活动了!
* 然后收拾电脑(同步任务)、收拾书包(同步任务)、给女朋友打电话说出来吃饭吧(必然是异步任务)，然后女朋友说你等会，我先化个妆，等我画好了call你。
* 那我不能干等着呀，就接着做别的事情，比如那我就在改个 bug 吧，你好了通知我。结果等她一个小时后说我化好妆了，我们出去吃饭吧。不行！我 bug 还没有解决掉呢？你等会。。。。其实这个时候你的一小时化妆还是 5 分钟化妆都已经毫无意义了。。。因为哥哥这会没空~~
* 如果我 bug 在半个小时就解决完了，没别的任务需要执行了，那么就在这等着呀！必须等着！随时待命！。然后女朋友来电话了，我化完妆了，我们出去吃饭吧，那么刚好，我们在你的完成了请求或者 timeout 时间到了后我刚好闲着，那么我必须立即执行了。

event loop: 时间循环，为了协调事件，用户交互，脚本，渲染，网络等，用户代理必须使用本节所述的event loop。

* 每个线程都有自己的event loop。
* 浏览器可以有多个event loop，browsing contexts和web workers就是相互独立的。
* 所有同源的browsing contexts（浏览器上下文）可以共用event loop，这样它们之间就可以相互通信。

### 1.1 task
一个event loop有一个或者多个task队列。当用户代理安排一个任务，必须将该任务增加到相应的event loop的一个tsak队列中。task也被称为macrotask(宏任务)

task 任务源：setTimeout、setInterval、setImmediate、I/O、UI rendering

### 1.2 microtask
* 每一个event loop都有一个microtask队列，一个microtask会被排进microtask队列而不是task队列。
* microtask 队列和task 队列有些相似，都是先进先出的队列，由指定的任务源去提供任务，不同的是一个event loop里只有一个microtask 队列。

microtask 任务源：process.nextTick、promise、Object.observe、MutationObserver

### 1.3 浏览器事件循环的过程
* 1.在tasks队列中选择最老的一个task,用户代理可以选择任何task队列，如果没有可选的任务，则跳到下边的microtasks步骤。
* 2.将上边选择的task设置为正在运行的task。
* 3.Run: 运行被选择的task。
* 4.将event loop的currently running task变为null。
* 5.从task队列里移除前边运行的task。
* 6.Microtasks: 执行microtasks任务检查点。（也就是执行microtasks队列里的任务）
* 7.更新渲染（Update the rendering）...
* 8.如果这是一个worker event loop，但是没有任务在task队列中，并且WorkerGlobalScope对象的closing标识为true，则销毁event loop，中止这些步骤，然后进行定义在Web workers章节的run a worker。
* 9.返回到第一步。

简要概括：
* event loop会不断循环的去取tasks队列的中最老的一个任务推入栈中执行，并在当次循环里依次执行并清空microtask队列里的任务。
* 执行完microtask队列里的任务，有可能会渲染更新。（浏览器很聪明，在一帧以内的多次dom变动浏览器不会立即响应，而是会积攒变动以最高60HZ的频率更新视图）

microtasks 检查：
* 1.将microtask checkpoint的flag设为true。
* 2.Microtask queue handling: 如果event loop的microtask队列为空，直接跳到第八步（Done）。
* 3.在microtask队列中选择最老的一个任务。
* 4.将上一步选择的任务设为event loop的currently running task。
* 5.运行选择的任务。
* 6.将event loop的currently running task变为null。
* 7.将前面运行的microtask从microtask队列中删除，然后返回到第二步（Microtask queue handling）。
* 8.Done: 每一个environment settings object它们的 responsible event loop就是当前的event loop，会给environment settings object发一个 rejected promises 的通知。
* 9.清理IndexedDB的事务。
* 10.将microtask checkpoint的flag设为flase。

什么时候会调用microtask checkpoint呢?
* 当上下文执行栈为空时，执行一个microtask checkpoint。
* 在event loop的第六步（Microtasks: Perform a microtask checkpoint）执行checkpoint，也就是在运行task之后，更新渲染之前。

UI Rendering 的时机：
* 在一轮event loop中多次修改同一dom，只有最后一次会进行绘制。
* 渲染更新（Update the rendering）会在event loop中的tasks和microtasks完成后进行，但并不是每轮event loop都会更新渲染，这取决于是否修改了dom和浏览器觉得是否有必要在此时立即将新状态呈现给用户。如果在一帧的时间内（时间并不确定，因为浏览器每秒的帧数总在波动，16.7ms只是估算并不准确）修改了多处dom，浏览器可能将变动积攒起来，只进行一次绘制，这是合理的。
* 如果希望在每轮event loop都即时呈现变动，可以使用requestAnimationFrame。

回答：
* 执行同步代码，这属于宏任务
* 执行栈为空，查询是否有微任务需要执行
* 执行所有微任务
* 必要的话渲染 UI
* 然后开始下一轮 Event loop，执行宏任务中的异步代码

## 2.nodejs 的 Eventloop
* timers: 执行setTimeout和setInterval中到期的callback。
* pending callback: 上一轮循环中少数的callback会放在这一阶段执行。
* idle, prepare: 仅在内部使用。
* poll: 最重要的阶段，执行 callback，在适当的情况下会阻塞在这个阶段。
  * 执行I/O回调。(如果 timer 为空，且 pool 不为空，循环遍历执行所有 pool，直到执行完或系统限制)
  * 处理轮询队列中的事件。(如果 pool 为空，如果有 setImmediate 执行，进入 check，否则等待 callback 被添加到 pool)
* check: 执行setImmediate(setImmediate()是将事件插入到事件队列尾部，主线程和事件队列的函数执行完成之后立即执行setImmediate指定的回调函数)的callback。
* close callbacks: 执行close事件的callback，例如socket.on('close'[,fn])或者http.server.on('close, fn)

## 3.node event loop 与浏览器有什么区别？
> 一句话总结其中：浏览器环境下，microtask的任务队列是每个macrotask执行完之后执行。而在Node.js中，microtask会在事件循环的各个阶段之间执行，也就是一个阶段执行完毕，就会去执行microtask队列的任务。
