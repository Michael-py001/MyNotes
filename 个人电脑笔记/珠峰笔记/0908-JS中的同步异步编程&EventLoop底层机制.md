## JS中的同步异步编程&EventLoop底层机制

### 「进程/线程」

- 进程代表的是一个程序（浏览器开一个页卡就是一个进程）
- 线程是用来处理处理进程中的具体事物的，如果一个程序中需要同时做好多事情，就需要开辟好多线程
- 一个线程同时只能做一件事情

### 浏览器是多线程的

- GUI渲染线程
- HTTP网络线程
- JS渲染线程

### JS是单线程的

- 浏览器只分配一个线程用来渲染JS代码
- JS中的代码大部分都是“同步编程”：上面的任务没有处理完成，下面的任务是无法处理的
- 但是JS中利用浏览器的多线程机制，可以规划出“异步编程”效果

具有异步效果的方法如下：

- 定时器
- ajax/Fetch/跨域 (HTTP网络请求)
- 事件绑定
- Promise中有也有异步编程
- Generator / yield 
- async / await
- ...

### 计算程序执行的时间(预估)

- 运行监控 console.time/timeEnd（受当前电脑运行环境的影响）
- 大O表示法(提前预估)(空间复杂度)

```js
console.time('AAA');
for (let i = 0; i < 99999999; i++) {}
console.timeEnd('AAA');//可以在浏览器输出时间
```

### 定时器的异步编程 

- 设置定时器任务是同步的
- “间隔interval这么长时间，执行定时器绑定的函数” 这个任务是异步的
- 遇到异步任务，浏览器不会等待它执行完，则继续渲染下面的代码；当等到下面代码运行完，时间也到达了执行的条件，才会把异步任务执行；

```js
setTimeout(() => {
    console.log("OK"); //2
}, 1000);
console.log('NO'); //1
```

- interval设置为零也不是立即执行，而是浏览器都有“最快反应时间（谷歌:5~6ms IE:13~17ms）”，设置为零，最快也需要等到5～6ms左右

```js
setTimeout(() => {
  console.log('OK'); //2
}, 0);
console.log('NO'); //1
```

### 例题

```js
setTimeout(() => {
    console.log(1);
}, 20);
console.log(2);
setTimeout(() => {
    console.log(3);
}, 10);
console.log(4);
console.time('AA');
for (let i = 0; i < 90000000; i++) {
    // do soming
}
console.timeEnd('AA'); //=>AA: 79ms 左右
console.log(5);
setTimeout(() => {
    console.log(6);
}, 8);
console.log(7);
setTimeout(() => {
    console.log(8);
}, 15);
console.log(9);
```

![1](https://i.loli.net/2020/09/08/83uK5NGCxIZWFfz.png)

先执行完同步任务，再执行异步任务，即使异步任务到了时间也不会先执行。

#### 微任务：Promise / async await

#### 宏任务：定时器

### Event Queue任务队列(优先级队列)

包含宏任务 macrotask和微任务 microtask的任务队列

### EventLoop事件循环

异步任务队列中：

- 有微任务，先执行微任务
- 没有微任务，再执行宏任务
- 执行顺序：谁先到时间的先执行谁
- 并且都是异步任务拿到执行上下文中，让“JS渲染线程”去渲染

![image-20200908234903124](https://i.loli.net/2020/09/08/lHn54AFoT2QNvhL.png)