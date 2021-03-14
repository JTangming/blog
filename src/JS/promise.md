### 你能手写一个 Promise 吗？
想要手写一个 Promise 则需要了解 Promise/A+ 规范并遵循这个规范。

#### 简易版（入门）
我们是这样来使用 promise：
```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {resolve('success');}, 1000);
})

const p2 = p1.then(val => {
  console.log(val)
})
```

通过 Promise/A+ 规范和对其 API 的熟知，实现一个 Promise 的具体思路是：
- 实例化一个 Promise 后，实例对象有一个 then 原型方法；
- 构建 Promise 对象时，需要传入一个 executor 函数，主要的业务流程都在 executor 函数中执行；
- executor 接收两个参数（resolve and reject），执行成功，则调用 resolve 函数，否则调用 reject 函数；
- Promise 一旦执行，状态将不可逆，即只可能是 pending -> fulfilled or rejected。

一个简易版的 Promise 如下：
```js
// 声明一下三种状态值
const PENDING = 'PENDING';
const FULFILLED = 'FULFILLED';
const REJECTED = 'REJECTED';

class Promise {
  constructor(executor) {
    this.status = PENDING;
    this.value = undefined;
    this.reason = undefined;
    // 通过发布订阅来执行 then
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks= [];

    let resolve = (value) => {
      if(this.status ===  PENDING) {
        this.status = FULFILLED;
        this.value = value;
        this.onResolvedCallbacks.forEach(fn => fn());
      }
    } 

    let reject = (reason) => {
      if(this.status ===  PENDING) {
        this.status = REJECTED;
        this.reason = reason;
        this.onRejectedCallbacks.forEach(fn => fn());
      }
    }

    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }

  then(onFulfilled, onRejected) {
    // 兼容一下 executor 同步使用的情况
    this.status === FULFILLED && onFulfilled(this.value);
    this.status === REJECTED && onRejected(this.reason);

    if (this.status === PENDING) {
      // 这里通过发布订阅的方式，将 onFulfilled 和 onRejected 函数存放到具体状态的 callback 中
      this.onResolvedCallbacks.push(() => {
        onFulfilled(this.value)
      });
      this.onRejectedCallbacks.push(()=> {
        onRejected(this.reason);
      })
    }
  }
}
```

#### 链式调用
链式调用即当 Promise 的 then 函数中 return 了一个值，不管是什么值，我们都能在下一个 then 中获取到，这就是所谓的then 的链式调用。

回到第一个例子，我们改造一下使用链式写法：
```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {resolve('success');}, 1000);
})

const p2 = p1.then(val => {
  console.log('p2 success:', val);
  // 或者 return 一个 promise/非 promise 的值
  throw new Error('失败了')
}).then(
    data => {
        console.log('success again', data);
    }, 
    ex => {
        console.log('fail again', data);
    }
);
```

结合 Promise/A+ 规范，咱们需要做到：
- promise 的 then 可以链式调用下去，这就需要每次执行完 promise.then 后返回一个新的 promise 实例；
- 如果 then 原型方法参数 fulfilled or rejected 函数返回的是一个非 promise 的值，则传递给下一个 then 的第一个回调函数（fulfilled）；
- 如果返回的是一个 promise，那么等待其执行完毕，成功或者失败的值通过 resolve or reject 分别传递给到下一个 then 的 fulfilled or rejected;

大致的伪代码如下：
```js
// status ...
const resolvePromise = (newPromise, ret, resolve, reject) => {
  // 如果是非基础类型的数据，另外判断处理
  if ((typeof ret === 'object' && ret != null) || typeof ret === 'function') { 
    try {
      // 如果上一个 then 的参数方法返回的是一个 promise，则等待 promise 执行完毕，再暴露执行结果
      let then = ret.then;
      if (typeof then === 'function') {
        // 返回的 promise 执行后，将成功或者失败的结果暴露给当前 then 的 fulfilled or rejected 
        then.call(ret, succ => {
          resolve(succ); 
        }, fail => {
          reject(fail);
        });
      } else {
        // 如果是引用类型（非 promise 的对象），则直接 resolve
        resolve(ret);
      }
    } catch (ex) {
      reject(ex);
    }
  } else {
    // 如果上一个 then 参数中的函数返回值 ret 是个基本类型的值则直接 resolve
    resolve(ret)
  }
}

class Promise {
  constructor(executor) {
    // this.xxxs ...

    let resolve = (value) => {
      if(this.status ===  PENDING) {
        this.status = FULFILLED;
        this.value = value;
        this.onResolvedCallbacks.forEach(fn=>fn());
      }
    } 

    let reject = (reason) => {
      if(this.status ===  PENDING) {
        this.status = REJECTED;
        this.reason = reason;
        this.onRejectedCallbacks.forEach(fn=>fn());
      }
    }

    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }

  then(onFulfilled, onRejected) {
    // 兼容 onFufilled，onRejected 不传值
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : () => {};
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
    // then 最终返回一个新的 promise 实例
    let newPromise = new Promise((resolve, reject) => {
      // ...

      if (this.status === PENDING) {
        this.onResolvedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let ret = onFulfilled(this.value);
              resolvePromise(newPromise, ret, resolve, reject);
            } catch (e) {
              reject(e)
            }
          }, 0);
        });

        this.onRejectedCallbacks.push(()=> {
          setTimeout(() => {
            try {
              let ret = onRejected(this.reason);
              resolvePromise(newPromise, ret, resolve, reject)
            } catch (e) {
              reject(e)
            }
          }, 0);
        });
      }
    });

    return newPromise;
  }
}
```

> 注，使用 setTimeout 模拟了微任务

#### 静态方法
常用的静态方法有 Promise.resolve()，Promise.reject()，可实现如下：
```js
static resolve/*or reject**/(data) {
  return new Promise((resolve, reject) => {
    resolve(data); // or reject(data);
  })
}
```

#### Promise.all
实现思路是：
- all 返回一个 promise；
- 如果每一个并发的 promise 都正常返回，则把值维护到一个数组中
- 否则直接 reject 即可

```js
Promise.all = function(promises) {
  return new Promise((resolve, reject) => {
    let resultArr = [];
    let orderIndex = 0;
    const processResultByKey = (value, index) => {
      resultArr[index] = value;
      if (++orderIndex === promises.length) {
          resolve(resultArr);
      }
    }
    for (let i = 0; i < promises.length; i++) {
      let value = promises[i];
      if (value && typeof value.then === 'function') {
        value.then((value) => {
          processResultByKey(value, i);
        }, reject);
      } else { // 如果并发的数组并不是一个 promise，则直接加到记录结果的数组中即可
        processResultByKey(value, i);
      }
    }
  });
}
```
#### Promise.race
```js
Promise.race = function(promises) {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promises.length; i++) {
      let val = promises[i];
      if (val && typeof val.then === 'function') {
        val.then(resolve, reject);
      } else {
        resolve(val)
      }
    }
  });
}
```
- 用来处理多个并发的值，遍历执行，如果是非 promise 实例，直接 resolve 即可；
- 否则的话，等待 promise 执行完毕，任何一个有结果直接 resolve or reject 即可；
- 注，promise 里边有控制仅 resolve 或 reject 一次了，这里不用再判断。