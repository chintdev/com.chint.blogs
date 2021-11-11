---
title: Promise # 文章标题
date: 2021-11-11 09:12:55  # 发布时间
author: djl # 文章作者
img: # 文章特征图，推荐使用图床(腾讯云、七牛云、又拍云等)来做图片的路径.如: http://xxx.com/xxx.jpg
top: true # 推荐文章（文章是否置顶）
hide: false # 隐藏文章，如果hide值为true，则文章不会在首页显示
cover: true # 文章是否需要加入到首页轮播封面中
coverImg: # 文章在首页轮播封面需要显示的图片路径，如果没有，则默认使用文章的特色图片
password:  # 文章阅读密码
toc: false # 是否开启 TOC，可以针对某篇文章单独关闭 TOC 的功能 `暂未开放`
mathjax: false # 是否开启数学公式支持 
summary: # 文章摘要
categories: # 文章分类
tags: Promise # 文章标签
keywords: Promise # 文章关键字，SEO 时需要
reprintPolicy: # 文章转载规则 `暂时不支持`
---

`过程综述：`

1. Promise 出现的原因
2. Promise 含义
3. Promise 基本用法
4. Promise 的相关面试题 

## 一、Promise 出现的原因
    Promise 出现以前，我们处理一个异步网络请求，大概是这样：
```javascript
// 请求 代表 一个异步网络调用。
// 请求结果 代表网络请求的响应。
请求1(function(请求结果1){
    处理请求结果1
})
```
     如遇到极端情况，就会出现如下代码：
```javascript
请求1(function(请求结果1){
    请求2(function(请求结果2){
        请求3(function(请求结果3){
            请求4(function(请求结果4){
                请求5(function(请求结果5){
                    请求6(function(请求结果6){
                        ...
                    })
                })
            })
        })
    })
})
```
    就会出现一个回调地狱，缺点：代码臃肿；可读性差；耦合度过高，可维护性差；代码复用性差；容易滋生 只能在回调里处理异常......于是 Promise 规范诞生了。
```javascript
new Promise(请求1)
    .then(请求2(请求结果1))
    .then(请求3(请求结果2))
    .then(请求4(请求结果3))
    .then(请求5(请求结果4))
    .catch(处理异常(异常信息))
```

## 二、Promise 含义
    Promise 是异步编程的一种解决方案。所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。

1. **Promise对象特点：**

  > **(1) 对象的状态不受外界影响。**
     在Promise的内部，有一个状态管理器的存在，有三种状态：pending(进行中)、resolved/fulfilled(已成功)和rejected(已失败)。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

  > **(2) 一旦状态改变，就不会再变，任何时候都可以得到这个结果。**
    Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。如果改变已经发生了，再对Promise对象添加回调函数，也会立即得到这个结果。

2. **Promise缺点：**

> **(1)无法取消Promise，一旦新建它就会立即执行，无法中途取消。**

> **(2)如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。**

> **(3)当处于pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。**

## 三、Promise 基本用法
    ES6 规定，Promise对象是一个构造函数，用来生成Promise实例。

```javascript
const promise = new Promise(function(resolve, reject) {
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
})
```

    Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});

```

### Promise.then()
    Promise 实例具有then方法，也就是说，then方法是定义在原型对象Promise.prototype上的。它的作用是为 Promise 实例添加状态改变时的回调函数。then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。
```javascript
promise1.then(onFulfilled1, onRejected1).then(onFulfilled2, onRejected2);
```

> **Promise 的执行规则，包括“值的传递”和“错误捕获”机制：**

> 1. 如果 onFulfilled 或者 onRejected 返回一个值 x ，则运行下面的 Promise 解决过程：

> - 若 x 不为 Promise ，则使 x 直接作为新返回的 Promise 对象的值， 即新的onFulfilled 或者 onRejected 函数的参数.
> - 若 x 为 Promise ，这时后一个回调函数，就会等待该 Promise 对象(即 x )的状态发生变化，才会被调用，并且新的 Promise 状态和 x 的状态相同。
```javascript
let promise1 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve()
    }, 1000)
  })
  promise2 = promise1.then(res => {
    // 返回一个普通值
    return '这里返回一个普通值'
  })
  promise2.then(res => {
    console.log(res) //1秒后打印出：这里返回一个普通值
  })

  let promise1 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve()
    }, 1000)
  })
  promise2 = promise1.then(res => {
    // 返回一个Promise对象
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve('这里返回一个Promise')
      }, 2000)
    })
  })
  promise2.then(res => {
    console.log(res) //3秒后打印出：这里返回一个Promise
  })
```
2. 如果 onFulfilled 或者onRejected 抛出一个异常 e ，则 promise2 必须变为失败（Rejected），并返回失败的值 e
```javascript
let promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
  }, 1000)
})
promise2 = promise1.then(res => {
  throw new Error('这里抛出一个异常e')
})
promise2.then(res => {
  console.log(res)
}, err => {
  console.log(err) //1秒后打印出：这里抛出一个异常e
})
```
3. 如果onFulfilled 不是函数且 promise1 状态为成功（Fulfilled）， promise2 必须变为成功（Fulfilled）并返回 promise1 成功的值
```javascript
let promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
  }, 1000)
})
promise2 = promise1.then('这里的onFulfilled本来是一个函数，但现在不是')
promise2.then(res => {
  console.log(res) // 1秒后打印出：success
}, err => {
  console.log(err)
})
```
4. 如果 onRejected 不是函数且 promise1 状态为失败（Rejected），promise2必须变为失败（Rejected） 并返回 promise1 失败的值
```javascript
let promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('fail')
  }, 1000)
})
promise2 = promise1.then(res => res, '这里的onRejected本来是一个函数，但现在不是')
promise2.then(res => {
  console.log(res)
}, err => {
  console.log(err)  // 1秒后打印出：fail
})
```
****
### Promise.catch()
    相当于调用 then 方法, 但只传入 Rejected 状态的回调函数。

### Promise.finally()
    Promise.finally()用于指定不管 Promise 对象最后状态如何，都会执行的操作。finally方法的回调函数不接受任何参数，这意味着没有办法知道，前面的 Promise 状态到底是fulfilled还是rejected。这表明，finally方法里面的操作，应该是与状态无关的，不依赖于 Promise 的执行结果。
```javascript
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```
### Promise.all()
    Promise.all()用于将多个 Promise 实例，包装成一个新的 Promise 实例。
```javascript
const p = Promise.all([p1, p2, p3]);
```
    Promise.all()方法接受一个数组作为参数，p1、p2、p3都是 Promise 实例，如果不是，就会先调用Promise.resolve方法，将参数转为 Promise 实例，再进一步处理。另外，Promise.all()方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。
    p的状态由p1、p2、p3决定，分成两种情况。
    （1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。
    （2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

### Promise.race()
    Promise.race()同样是将多个 Promise 实例，包装成一个新的 Promise 实例。
```javascript
const p = Promise.race([p1, p2, p3]);
```
    只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。Promise.race()的参数如果不是 Promise 实例，就会先调用Promise.resolve()方法，将参数转为 Promise 实例，再进一步处理。

### Promise.allSettled()
    Promise.allSettled()，用来确定一组异步操作是否都结束了（不管成功或失败）

### Promise.any()
    Promise.any()不会因为某个 Promise 变成rejected状态而结束，必须等到所有参数 Promise 变成rejected状态才会结束。

### Promise.resolve()

### Promise.rejected()

### Promise.try()

## 四、Promise 的相关面试题 

1. [手写Promise原理](https://www.jianshu.com/p/43de678e918a)
2. 下面1、2、3代码有何不同？

```javascript
      // 1
  var promise = new Promise(function (resolve, reject) {
    setTimeout(function () {
      resolve(1);
    }, 3000)
  })
  promise.then(2).then((n) => {
    console.log(n)
  });
  // 2
  var promise = new Promise(function (resolve, reject) {
    setTimeout(function () {
      resolve(1);
    }, 3000)
  });
  promise.then(() => {
    return Promise.resolve(2);
  }).then((n) => {
    console.log(n)
  });
  // 3
  var promise = new Promise(function (resolve, reject) {
    setTimeout(function () {
      resolve(1);
    }, 3000)
  })
  promise.then(() => {
    return 2
  }).then((n) => {
    console.log(n)
  });
```
3. 
```javascript
let a;
const b = new Promise((resolve, reject) => {
  console.log('promise1');
  resolve();
}).then(() => {
  console.log('promise2');
}).then(() => {
  console.log('promise3');
}).then(() => {
  console.log('promise4');
});

a = new Promise(async (resolve, reject) => {
  console.log(a);
  await b;
  console.log(a);
  console.log('after1');
  await a
  resolve(true);
  console.log('after2');
});

console.log('end');
```