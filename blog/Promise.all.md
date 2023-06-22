---
  category: 专业知识
  tags:
    - JavaScript
  date: 2020-11-13
  title: 你不知道的Promise.all
  vssue-title: 你不知道的Promise.all
---
滴滴面试被面试官问到promise.all的细节--当Promise.all中一个promise失败时，其它promise的状态会是什么
当时没答上来，现在在此复盘一下
```js
const promise1 = new Promise((resolve) => {
  resolve('promise1')
})

const promise2 = new Promise((resolve, reject) => {
  reject(new Error('error2'))
})

const promise3 = new Promise((resolve, reject) => {
  setTimeout(() => {
  reject(new Error('error3'))
    
  }, 2000);
})

const res = Promise.all([promise3, promise1, promise2]).then(res => {
  console.log(res); // 不执行
}).catch(e => {
  console.log(e); // Error: error2
}).finally(() => {
  console.log(promise1); // Promise { 'promise1' }
  console.log(promise2); //  Promise {
                         //  <rejected> Error: error2 ...
  console.log(promise3); //  Promise { <pending> 
  setTimeout(() => {
      console.log(promise3); //  Promise {
                             //  <rejected> Error: error3 ...
  }, 3000);
})
```

结论： 
1. 当Promise.all的一个promise reject时，不会影响其它promise的状态
2. Promise.all 返回的是时间维度的第一个reject的error
3. 如果Promise.all中一个promise为reject时，其它promise如果还未resolve或者reject，那么仍然处于pending状态，当之后获得异步的结果后，会变成resolve或者reject状态

以下是Promise.all的实现
```js
MyPromise.all = function(promiseList) {
  var resPromise = new MyPromise(function(resolve, reject) {
    var count = 0;
    var result = [];
    var length = promiseList.length;

    if (length === 0) {
      return resolve(result);
    }

    promiseList.forEach(function(promise, index) {
      // 通过Promise.resolve包裹，允许传入的promise参数是个普通的值
      MyPromise.resolve(promise).then(function(value) {
        count++;
        // 使用索引进行存储而非数组的push方法，是为了保证resolve的结果与传入的promise一一对应，防止因为异步导致的错位
        result[index] = value;
        // 当所有promise都resolve时，promise.all才resolve
        if (count === length) {
          resolve(result);
        }
        // 当一个promise被reject时，promise.all立马reject
      }, reject);
    });
  });

  return resPromise;
};
```

