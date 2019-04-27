---
title: Promise链式操作引出的EventLoop
slug:
date: 2019-04-08
author: summerpen
# cover: true
categories: JS
tags: ES6
---

阅读此文章前需要有一定的 Promise 基础 ,若无,请移步阮一峰老师的 ES6 入门[http://es6.ruanyifeng.com/#docs/promise](http://es6.ruanyifeng.com/#docs/promise)

#### 此篇文章主要说下 Promise 的链式操作

我们经常会遇到依次请求多个接口,或者其他操作的情况
简单写下 Promise 如何处理优雅进行链式操作,先看以下代码

```javascript
"use strict";
//Promise1
function Pro1(p) {
  return new Promise((resolve, reject) => {
    console.log("p1");
    let a = Math.random();
    if (a > 0.5) {
      resolve(a);
    } else {
      reject(a);
    }
  });
}
//模拟上传
function upLoad() {
  return new Promise((resolve, reject) => {
    console.log(`upLoad${Date.now()}`);
    //创建3种情况
    let key = Math.floor(Math.random() * 3 + 1);
    switch (key) {
      case 1: //上传成功
        resolve("success");
        break;
      case 2:
        return upLoad(); //上传失败可重新上传
        break;
      case 3:
        reject("upLoad err"); //代码层面错误,reject
        break;
      default:
        break;
    }
  });
}
function test() {
  Pro1()
    .then(data => {
      console.log(`Pro1进入upLoad`);
      ...
      ...
      return upLoad();
    })
    .then(data => {
      console.log("完成upLoad");
      console.log(data);
      ...
      ...
    })
    .catch(err => {
      throw new Error(err);
    });
}
test();
```

##### promise 的调用

可以看出,在 test 函数中,依次调用 Pro1,upLoad 两个函数,每个函数中返回的都是`Promise对象`,最后通过`catch`对两个函数进行错误处理

1. 在`Pro1`中,各有一半的几率返回`resolve`和`reject`函数.在 test 函数中,先调用 Pro1 函数,完成后返回 Promise 对象,
   因而可以继续使用.then 方法. 在`第一个`.then 中继续调用 upload 方法
2. 在`upload`中,三种情况分别返回:
   - `resolve`成功,
   - 上传失败,重新上传.(此处 return upLoad 函数,进行自调)
   - `reject`失败,
3. 完成`upLoad`后,在第二个.then 里进行书数据处理,其他想进行的操作.
4. `Pro1`和`upLoad`如有`reject()`时,会进行`.catch`中的错误抛出

##### 这样就完成的两个 Promise 对象的链式操作,多个层级的链式操作类似

#### 使用 Promise 对象,逻辑可以更清晰,函数自调也更方便.

#### 看到这里,想起了前几天看到的一道题目

```js
new Promise((resolve, reject) => {
  console.log("promise1");
  resolve();
})
  .then(() => {
    console.log("then11");
    new Promise((resolve, reject) => {
      console.log("promise2");
      resolve();
    })
      .then(() => {
        console.log("then21");
      })
      .then(() => {
        console.log("then23");
      });
  })
  .then(() => {
    console.log("then12");
  });
```

Promise 为了解决回调地狱,但是有时候也不乏 Promise 的嵌套

#### 题目解析

##### 主要配合 Promise 考察 EventLoop

1. 所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。

2. 主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。

3. 一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。

4. 主线程不断重复上面的第三步。

##### 即 js 分为:

- 同步任务和异步任务

- 宏任务(macrotask)和微任务(microtask)

- 主线程（同步任务） - 所有同步任务都在主线程上执行，形成一个执行栈。

- 任务队列（异步任务）：当异步任务有了结果，就在任务队列中放一个事件。

- JS 运行机制：当"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"

其中宏任务包括：script（主代码）, setTimeout, setInterval, setImmediate, I/O, UI rendering

微任务包括：process.nextTick（Nodejs）, Promises, Object.observe, MutationObserver

这里我们注意到，宏任务里有 script，也就是我们的正常执行的主代码

##### 事件循环 EventLoop

> 主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为 Event Loop（事件循环）。此机制具体如下:主线程会不断从任务队列中按顺序取任务执行，每执行完一个任务都会检查 microtask 队列是否为空（执行完一个任务的具体标志是函数执行栈为空），如果不为空则会一次性执行完所有 microtask。然后再进入下一个循环去任务队列中取下一个任务执行。

总结了下过程：script(宏任务) - 清空微任务队列 - 执行一个宏任务 - 清空微任务队列 - 执行一个宏任务， 如此往复。

- 先执行 script 里的同步代码（此时是宏任务）。碰到异步任务，放到任务队列。
- 查找任务队列有没有微任务，有就把此时的微任务全部按顺序执行 （这就是为什么 promise 会比 setTimeout 先执行，因为先执行的宏任务是同步代码，setTimeout 被放进任务队列了，setTimeout - 又是宏任务，在它之前先得执行微任务(就比如 promise)）。
- 执行一个宏任务（先进到队列中的那个宏任务），再把这次宏任务里的宏任务和微任务放到任务队列。
- ...重复 2、3 步骤

##### 要做到心中有队列，有先进先出的概念

<!-- 如图: -->
<!-- ![EventLoop图解](https://dev.tencent.com/u/summerpen/p/summerpen/git/blob/master/medias/JS/EventLoop.png) -->

##### 输出结果

> promise1
> then11
> promise2
> then21
> then12
> then23

> 第一轮
>
> - current task: promise1 是当之无愧的立即执行的一个函数，参考上一章节的 executor，立即执行输出`promise1`
> - micro task queue: [promise1 的第一个 then]
>   第二轮
> - current task: then1 执行中，立即输出了 `then11` 以及新 promise2 的 `promise2`
> - micro task queue: [新 promise2 的 then 函数,以及 promise1 的第二个 then 函数]
>   第三轮
> - current task: 新 promise2 的 then 函数输出 `then21` 和 promise1 的第二个 then 函数输出 `then12`。
> - micro task queue: [新 promise2 的第二 then 函数]
>   第四轮
> - current task: 新 promise2 的第二 then 函数输出 `then23`
> - micro task queue: []
> - END

### THE END
