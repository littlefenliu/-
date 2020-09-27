## ES6 Promise

异步编程传统解决方案：回调函数&事件

Promise对象代表一个异步操作

### 1.用法

```js
const promise = new Promise(function(resolve, reject) {
        if ( /*异步操作成功*/ ) {
            resolve(value);
        } else {
            reject(error);
        }
  });
```

 `Promise`构造函数接受一个函数作为参数，该函数的两个参数分别是**resolve**和**reject**。它们是两个函数。 

 `resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从 **pending** 变为 **resolved**），在异步**操作成功**时调用，并将异步操作的结果，作为参数传递出去；

`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”（即从 **pending** 变为 **rejected**），在异步**操作失败**时调用，并将异步操作报出的错误，作为参数传递出去。 

```js
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```



###  2.例子

```js
function timeout(ms) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, ms, 'done');
    });
  }
timeout(100).then((value) => {
    console.log(value);
});
```

 上面代码中，`timeout`方法返回一个`Promise`实例，表示一段时间以后才会发生的结果。

过了指定的时间（`ms`参数）以后，`Promise`实例的状态变为`resolved`，就会触发`then`方法绑定的回调函数。 

### 3.promise新建后会立即执行

```js
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});
promise.then(function() {
    console.log('resolved');
});
console.log('hi');
//Promise
//hi
//resolved
```

上面代码中，Promise 新建后**立即执行**，所以首先输出的是Promise。
然后，then方法指定的回调函数，将在当前脚本**所有同步**任务执行完才会执行，所以resolved最后输出。

### 错题

```js
setTimeout(() => {			//setTimeout 在主线程中没有同步代码下才会执行
        console.log(1);
    }, 0);							
new Promise(function(resolve, reject) {		//promise创建时执行 内部代码同步执行
    console.log(2);
    for (var i = 0; i < 1000; i++) {
        if (i === 10) {
            console.log(10);
        }
        i === 999 && resolve();
    }
      console.log(3);
}).then(function() {		//then方法指向的回调在当前脚本的同步任务执行完成后执行
    console.log(4);
});
console.log(5);
//执行顺序：2 10 3 5 4 1
//同步执行的代码-> promise.then-> setTimeout
```

1、创建**Promise**实例是同步执行的。所以先输出2，10，3，5这四行代码都是**同步执行。**

2、promise.then和setTimeout都是异步执行，会先执行谁呢？

setTimeout异步会放到**异步队列**中等待执行。

promise.then异步会放到**microtask queue**中。microtask队列中的内容经常是为了需要直接在当前脚本执行完后立即发生的事，所以当同步脚本执行完之后，就调用**microtask**队列中的内容，然后把**异步队列**中的setTimeout放入执行栈中执行，所以最终结果是先执行promise.then异步，然后再执行setTimeout异步。

注意：**目前microtask队列中常用的就是promise.then。**

### 4.基于 Promise 处理多次 Ajax 请求

实际开发中，我们经常需要同时请求多个接口。比如说：在请求完`接口1`的数据`data1`之后，需要根据`data1`的数据，继续请求接口 2，获取`data2`；然后根据`data2`的数据，继续请求接口 3。

换而言之，现在有三个网络请求，请求 2 必须依赖请求 1 的结果，请求 3 必须依赖请求 2 的结果，如果按照往常的写法，会有三层回调，会陷入“回调地狱”。

这种场景其实就是接口的多层嵌套调用。有了 Promise 之后，我们可以把多层嵌套调用按照**线性**的方式进行书写，非常优雅。也就是说：Promise 可以把原本的**多层嵌套调用**改进为**链式调用**。

```js
const request = require('request');
    //promise封装接口1
    const request1 = function() {
            const promise = new Promise((resolve, reject) => {
                request('http://www.baidu.com', function(response) {
                    if (response.retCode == 200) {
                        // 这里的 response 是接口1的返回结果
                        resolve('request1 success' + response);
                    } else {
                        reject('接口请求失败');
                    }
                });
            });
            return promise;
        }
        //promise封装接口2
    const request2 = function() {
            const promise = new Promise((resolve, reject) => {
                request('http://www.baidu.com', function(response) {
                    if (response.retCode == 200) {
                        // 这里的 response 是接口1的返回结果
                        resolve('request2 success' + response);
                    } else {
                        reject('接口请求失败');
                    }
                });
            });
            return promise;
        }
        //promise封装接口3
    const request3 = function() {
            const promise = new Promise((resolve, reject) => {
                request('http://www.baidu.com', function(response) {
                    if (response.retCode == 200) {
                        // 这里的 response 是接口1的返回结果
                        resolve('request3 success' + response);
                    } else {
                        reject('接口请求失败');
                    }
                });
            });
            return promise;
        }
        // 先发起request1，等resolve后再发起request2；紧接着，等 request2有了 resolve之后，再发起 request3
    request1().then((rest1) => {
            //rest1是resolve（）中的参数
            console.log(rest1);
            return request2();
        })
        .then((rest2) => {
            console.log(rest2);
            return request3();
        })
        .then((rest3) => {
            console.log(rest3);
        });
```

### return的函数返回值

两种情况

1. 返回promise实例对象，对象会调用下一个then();
2. 返回普通值。同时产生一个新的默认promise实例来调用then，防止链式操作断掉



## 5.promise常用的API：实例方法

Promise 自带的 API 提供了如下实例方法：

- promise.then()：获取异步任务的正常结果。
- promise.catch()：获取异步任务的异常结果。
- promise.finaly()：异步任务无论成功与否，都会执行。

```js
function queryData() {
        return new Promise((resolve, reject) => {
            setTimeout(function() {
                //接口返回的数据
                var data = {
                    retCode: 0,
                    msg: 'shentihao'
                }
                if (data.retCode == 0) {
                    // 接口请求成功时调用
                    resolve(data);
                } else {
                    // 接口请求失败时调用
                    reject({
                        retCode: -1,
                        msg: 'network Error'
                    });
                }
            }, 100);
            queryData().then((data) => {
                // 从resolve获取正常结果
                console.log('接口请求成功时，走这里');
                console.log(data);
            })
              .catch((data) => {
                // 从reject获取异常结果
                console.log('接口请求失败时，走这里');
                console.log(data);
            })
              .finally(() => {
                console.log('无论接口请求是否成功，都会这里');
            })
        });
    }
```



## 6.Promise 的常用 API：对象方法

Promise 自带的 API 提供了如下对象方法：

- Promise.all()：并发处理多个异步任务，所有任务都执行成功，才能得到结果。&
- Promise.race(): 并发处理多个异步任务，只要有一个任务执行成功，就能得到结果。|| 

```js
function queryData(url) {
    return new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {
            if (xhr.readyState != 4) return;
            if (xhr.readyState == 4 && xhr.status == 200) {
                // 处理正常结果
                resolve(xhr.responseText);
            } else {
                // 处理异常结果
                reject('服务器错误');
            }
        };
        xhr.open('get', url);
        xhr.send(null);
    });
}
    var promise1 = queryData('http://localhost:3000/api1');
    var promise2 = queryData('http://localhost:3000/api2');
    var promise3 = queryData('http://localhost:3000/api3');
    //全都成功执行all
    Promise.all([promise1, promise2, promise3]).then((result) => {
        console.log(result);
    });
    //有一个成功执行race
    Promise.race([promise1, promise2, promise3]).then((result) => {
        console.log(result);
    });
```

