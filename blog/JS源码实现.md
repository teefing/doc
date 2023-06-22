---
category: 专业知识
tags:
  - JavaScript
date: 2021-02-25
title: JS源码实现
vssue-title: JS源码实现
top: true
---
[[toc]]
# JS 源码实现

## 原型链

### instanceof

```js
const instanceOf = (instance, constructor) => {
  let p = instance.__proto__
  while(p) {
    if(p === constructor.prototype) return true
    p = p.__proto__
  }
  return false
}

console.log(instanceOf(1, Number));
```

### new

```js
function NEW(className, ...args) {
  const obj = {};
  obj.__proto__ = className.prototype;
  const res = className.call(obj, ...args);
  return res instanceof Object
    ? res
    : obj;
}

function classA(age) {
  this.name = "start";
  this.age = age;
}
classA.prototype.sayName = function() {
  console.log(this.name);
};
classA.prototype.sayAge = function() {
  console.log(this.age);
};

const a = NEW(classA, 19);
a.sayName();
a.sayAge();
a.name = "a";
a.sayName();

```

### Object.create

```js
/**
 *
 * @param {*} prototype 传入的prototype将作为新对象的__proto__
 * @param {*} propertyDescriptors 自定义的属性描述符
 */
function create(prototype, propertyDescriptors) {
  const F = function() {};
  F.prototype = prototype;
  let res = new F();
  if (propertyDescriptors) {
    Object.defineProperties(res, propertyDescriptors);
  }
  return res
}

/* Object.create 和 new的本质上都是创建一个新对象，将__proto__指向prototype
两者的区别是前者直接接收prototype， 而后者接收constructor，通过constructor.prototype来间接获得prototype
*/
function create1(prototype, propertyDescriptors) {
  let res = {};
  res.__proto__ = prototype
  if (propertyDescriptors) {
    Object.defineProperties(res, propertyDescriptors);
  }
  return res
}

let obj = {a: 11}
let copy = Object.create(obj, {mm: {value: 10, enumerable: true}})
console.log(copy);
console.log(obj)

let obj1 = {a: 11}
let copy1 = create(obj, {mm: {value: 10, enumerable: true}})
console.log(copy1);
console.log(obj1)

let obj2 = {a: 11}
let copy2 = create1(obj, {mm: {value: 10, enumerable: true}})
console.log(copy2);
console.log(obj2)
```

### 继承

```js
// es3
function __extends(child, parent) {
  // 继承静态方法和属性
  child.__proto__ = parent;
  // 常规继承
  function __() {
    this.constructor = child;
  }
  __.prototype = parent.prototype;
  child.prototype = new __();
}

// es5
function Parent(val) {
  this.val = val;
}
Parent.StaticFunc = function() {
  console.log("static");
};

Parent.prototype.getValue = function() {
  console.log(this.val);
};

// 常规继承
function Child(value) {
  Parent.call(this, value);
}
Child.prototype = Object.create(Parent.prototype, {
  constructor: {
    value: Child,
    enumerable: false,
    writable: true,
    configurable: true,
  },
});
// 继承静态属性方法
Child.__proto__ = Parent;

const child = new Child("bob");
Child.StaticFunc();
child.getValue();

// es6就用class继承，太过简单就不写了
```

## this

### call

```js
Function.prototype.myCall = function(context = window, ...args) {
  const unique = Symbol("fn");
  context[unique] = this;
  const res = context[unique](...args);
  delete context[unique];
  return res;
};
```

### apply

```js
Function.prototype.myApply = function(context = window, args = []) {
  const unique = Symbol("fn");
  context[unique] = this;
  const res = context[unique](...args);
  delete context[unique];
  return res;
};
```

### bind

```js
// 第一版
Function.prototype.myBind = function(context, ...bindArgs) {
  const _this = this;
  context = context || window;

  return function(...args) {
    _this.apply(context, [...bindArgs, ...args]);
  };
};

// 精简版
Function.prototype.myBind = function(context = window, ...bindArgs) {
  return (...args) => this.apply(context, [...bindArgs, ...args]);
};

// 无依赖版
Function.prototype.myBind = function(context = window, ...bindArgs) {
  const uniq = Symbol("fn");
  context[uniq] = this;
  return function(...args) {
    const res = context[uniq](...bindArgs, ...args);
    delete context[uniq];
    return res;
  };
};
```

## 函数式

### curry

```js
function curry(fn, ...fixs) {
  return fn.length <= fixs.length
    ? fn(...fixs)
    : (...args) => curry(fn, ...fixs, ...args);
}

// 函数柯里化核心：fn.length
// 当传入的参数列表长度大于等于 原函数所需的参数长度(fn.length)时，执行原函数
// 否则返回一个能接收参数继续进行柯里化的函数

const add = (a, b, c) => a + b + c;
const curried = curry(add);
console.log(curried(1, 2)(3));
console.log(curried(1)(2, 3));
console.log(curried(1)(2)(3));
console.log(curried(1, 2, 3, 4));
```

```js
function add(...args) {
  let arr = args;
  function fn(...rest) {
    arr = [...arr, ...rest];
    return fn;
  }
  fn[Symbol.toPrimitive] = function() {
    return arr.reduce((acc, cur) => acc + parseInt(cur));
  };
  // fn.toString = fn.valueOf = function () {
  //   return arr.reduce((acc, cur) => acc + parseInt(cur));
  // };

  return fn;
}

const res = add(1)(2);
console.log(res + 10); // 13
console.log(add(1)(2)(3));
console.log(add(1, 2, 3));
/**
 * 函数柯里化另一种实现思路，可以实现对不定参数的函数实现柯里化，原理是每次调用只存储传入的参数，并且把存储参数的函数返回出去，重写函数的toString和valueOf，在外部对该函数进行使用时，就会调用重写后的toString和valueOf
 */
```

### compose

```js
function compose(...func) {
  return func.reduce((a, b) => {
    return (...args) => a(b(...args));
  });
}
```

## 防抖节流

### throttle

```js
function throttle(fn, timeout = 200) {
  let lastTime = 0;
  let cur;
  let result;
  return function cb(...args) {
    cur = Date.now();
    if (cur - lastTime >= timeout) {
      result = fn.call(this, ...args);
      lastTime = cur;
    }
    return result;
  };
}

function throttleSetTimeOutVersion(fn, timeout = 1000, immediate = false) {
  let timer = null;
  let __immediate = immediate;
  return function (...args) {
    if (__immediate) {
      fn.call(this, ...args);
      __immediate = false;
    }
    if (timer) return;
    timer = setTimeout(() => {
      fn.call(this, ...args);
      timer = null;
    }, timeout);
  };
}

const obj = {
  value: 1,
};

function print(val) {
  console.log(this.value + val);
}

const printThrottled = throttle(print, 5000);

// 16毫秒执行一次printThrottled方法
setInterval(() => {
  printThrottled.call(obj, 1);
}, 16);

```

### debounce

```js
function debounce(fn, timeout = 1000) {
  let t;
  let result;
  return function cb(...args) {
    // 在每次调用的时候都清除上一次的定时器，不管定时器内函数是否已经执行
    clearTimeout(t);
    t = setTimeout(() => {
      result = fn.call(this, ...args);
    }, timeout);
    return result;
  };
}

const obj = {
  value: 1,
};

function print(val) {
  console.log(this.value + val);
}

const printDebounced = debounce(print, 1000);

// 16毫秒执行一次printThrottled方法, 那么print永远不会被执行到
setInterval(() => {
  printDebounced.call(obj, 1);
}, 16);
```

## 异步

### promise 实现

```js
/** 三种状态 */
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
class MyPromise {
  /** 默认为等待态 */
  status = PENDING;
  /** 当状态为FULFILLED时存储的值 */
  value = undefined;
  /** 当状态为REJECTED时存储的拒因 */
  reason = undefined;
  /** 通过时的回调队列 */
  successCallbacks = [];
  /** 拒绝时的回调队列 */
  failCallbacks = [];

  constructor(executor) {
    /** promise同步代码执行时的错误处理 */
    try {
      /** 在创建promise时执行执行器函数 */
      executor(this.resolve, this.reject);
    } catch (e) {
      this.reject(e);
    }
  }

  /** 以value为值通过该promise */
  resolve = (value) => {
    /** 只有当状态为等待态时才能改变状态 */
    if (this.status === PENDING) {
      /** 将状态改为完成 */
      this.status = FULFILLED;
      this.value = value;
      /** 执行成功回调队列中的函数 */
      this.successCallbacks.forEach((fn) => fn());
    }
  };

  /** 以reason为拒因拒绝改promise */
  reject = (reason) => {
    /** 只有当状态为等待态时才能改变状态 */
    if (this.status === PENDING) {
      /** 将状态改为拒绝 */
      this.status = REJECTED;
      this.reason = reason;
      /** 执行成功回调队列中的函数 */
      this.failCallbacks.forEach((fn) => fn());
    }
  };

  /** 向promise中执行或注册promise在不同状态时的回调事件 */
  then = (onSuccess, onFail) => {
    let promise2;
    const resolvePromise = (promise2, x, resolve, reject) => {
      /**处理promise循环调用自身的情况 */
      if (promise2 === x) {
        return reject(new TypeError("不允许promise循环调用自身"));
      }
      if (x instanceof MyPromise) {
        x.then((v) => {
          resolvePromise(promise2, v, resolve, reject);
        }, reject);
      } else {
        resolve(x);
      }
    };
    const successCallback = (resolve, reject) => {
      /** promise2只有在MyPromise的逻辑结束后才能生成，如果因此如果同步执行下面的代码，获取到的promise2是undefined, 因此需要使用settimeout使下面代码执行时间推迟到promise2生成后 */
      setTimeout(() => {
        /** then回调函数执行时的错误处理 */
        try {
          /** 处理传入的callback非法的情况，当callback不是函数时，忽略这个then */
          if (typeof onSuccess === "function") {
            /** 不仅要执行回调函数，还要处理then中return数据作为下一个then的value的情况 */
            let x = onSuccess(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } else {
            /** 以上当前promise的value作为promise2 resolve时的value */
            resolve(this.value);
          }
        } catch (e) {
          reject(e);
        }
      }, 0);
    };

    const failCallback = (resolve, reject) => {
      /** promise2只有在MyPromise的逻辑结束后才能生成，如果因此如果同步执行下面的代码，获取到的promise2是undefined, 因此需要使用settimeout使下面代码执行时间推迟到promise2生成后 */
      setTimeout(() => {
        /** then回调函数执行时的错误处理 */
        try {
          /** 处理传入的callback非法的情况，当callback不是函数时，忽略这个then */
          if (typeof onFail === "function") {
            /** 不仅要执行回调函数，还要处理then中return数据作为下一个then的value的情况 */
            let x = onFail(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } else {
            /** 以上当前promise的reason作为promise2 reject时的reason */
            reject(this.reason);
          }
        } catch (e) {
          reject(e);
        }
      }, 0);
    };

    /** 为了支持then的链式调用，需要每次都返回一个新的promise */
    promise2 = new MyPromise((resolve, reject) => {
      /** 如果是PENDING状态，存储回调事件，否则直接执行 */
      switch (this.status) {
        case FULFILLED:
          successCallback(resolve, reject);
          break;
        case REJECTED:
          failCallback(resolve, reject);
          break;
        case PENDING:
          this.successCallbacks.push(() => successCallback(resolve, reject));
          this.failCallbacks.push(() => failCallback(resolve, reject));
          break;
        default:
      }
    });

    return promise2;
  };

  /** this.then(null, fn)的语法糖 */
  catch = (fn) => {
    /** 需要链式调用，所以需要return */
    return this.then(null, fn);
  };

  /**
   * finally传入的回调函数不管promise被reject还是resolve都会被执行
   * finally支持链式调用
   * 如果finally返回了普通值，将无视该返回值，下一个then接收的值仍然是finally上游的返回值
   * 如果返回了promise，下一个then将等待该promise的执行
   * 如果回调函数在执行过程中throw了一个错误，则会作为新的拒因传递给下一个then
   * 注意，finally的回调函数不接受value或者reason
   */
  finally = (fn) => {
    /** 需要链式调用，所以需要return */
    return this.then(
      (value) => {
        /** 在如果fn在执行过程中抛出错误，不会执行then中的回调函数，而是继续向下传递 */
        return MyPromise.resolve(fn()).then(() => value);
      },
      (e) => {
        /** 在如果fn在执行过程中抛出错误，不会执行then中的回调函数，而是继续向下传递 */
        return MyPromise.resolve(fn()).then(() => {
          throw e;
        });
      }
    );
  };

  /** 如果Promise.resolve的参数是一个promise，直接返回该promise，否则创建一个新的promise，该promise状态为fulfilled */
  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise((resolve, reject) => {
      resolve(value);
    });
  }

  /** 不管Promise.reject的参数是什么，都将它作为拒因，创建一个新的promise，该promise的状态为rejected */
  static reject(reason) {
    return new MyPromise((resolve, reject) => {
      reject(reason);
    });
  }

  /** 下面的race，allSettled，any都是以all为蓝本进行修改的 */
  static all(array) {
    /** 已经resolve的promise数量 */
    let count = 0;
    let length = array.length;
    /** 返回的结果数组 */
    let result = [];
    /** 当数组中有一个promise的结果为rejected，直接整个promise reject，并且以该rejected状态的promise的拒因为promise.all的拒因 */
    return new MyPromise((resolve, reject) => {
      array.forEach((promise, index) => {
        /** 通过Promise.resolve将非promise的值转为promise，来统一处理 */
        MyPromise.resolve(promise).then((v) => {
          result[index] = v;
          count++;
          /** 只有当已经resolve的promise的数量和传入的数组长度一致，才resolve结果数组 */
          if (count === length) {
            resolve(result);
          }
        }, reject);
      });
    });
  }

  static race(array) {
    /** 当数组中有一个promise被resolve或者reject了，就作为race的value或reason被返回 */
    return new MyPromise((resolve, reject) => {
      array.forEach((promise) => {
        /** 通过Promise.resolve将非promise的值转为promise，来统一处理 */
        MyPromise.resolve(promise).then(resolve, reject);
      });
    });
  }

  /** 当数组中的promise被resolve或者reject时，都是被settle了，当数组中所有的promise都被settle，返回结果数组 */
  static allSettled(array) {
    /** 已经settle的promise数量 */
    let count = 0;
    let length = array.length;
    /** 返回的结果数组 */
    let result = [];

    return new MyPromise((resolve, reject) => {
      array.forEach((promise, index) => {
        /** 通过Promise.resolve将非promise的值转为promise，来统一处理 */
        MyPromise.resolve(promise).then(
          (v) => {
            result[index] = {
              value: v,
              status: "fulfilled",
            };
            count++;
            /** 只有当已经settle的promise的数量和传入的数组长度一致，才resolve结果数组 */
            if (count === length) {
              resolve(result);
            }
          },
          (e) => {
            result[index] = {
              reason: e,
              status: "rejected",
            };
            count++;
            /** 只有当已经settle的promise的数量和传入的数组长度一致，才resolve结果数组 */
            if (count === length) {
              resolve(result);
            }
          }
        );
      });
    });
  }

  /** 借用Promise.all的Promise.allSettle的简化版 */
  static allSettled2(array) {
    return MyPromise.all(
      array.map((promise) => {
        return MyPromise.resolve(promise).then(
          (v) => ({ status: "fulfilled", value: v }),
          (e) => ({ status: "rejected", reason: e })
        );
      })
    );
  }

  /** https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/any */
  static any(array) {
    /** 已经reject的promise数量 */
    let count = 0;
    let length = array.length;
    /** 当数组中有一个promise的结果为fulfilled，直接整个promise resolve，并且以该ulfilled状态的promise的value为promise.any的value */
    return new MyPromise((resolve, reject) => {
      array.forEach((promise) => {
        /** 通过Promise.resolve将非promise的值转为promise，来统一处理 */
        MyPromise.resolve(promise).then(resolve, (e) => {
          count++;
          /** 当所有的promise都失败时，reject一个 AggregateError*/
          if (count === length) {
            reject(
              new Error(
                "AggregateError: No Promise in Promise.any was resolved"
              )
            );
          }
        });
      });
    });
  }
  static retry(promiseCreator, times, delay) {
    return new MyPromise((resolve, reject) => {
      function attempt() {
        promiseCreator()
          .then((data) => {
            resolve(data);
          })
          .catch((err) => {
            if (times === 0) {
              reject(err);
            } else {
              times--;
              setTimeout(attempt, delay);
            }
          });
      }
      attempt();
    });
  }
}

module.exports = MyPromise;

```

### promisify

```js
function func(a, b, cb) {
  const res = a + b;
  cb(null, res);
}

function promisify(fn) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      args.push((err, data) => {
        if (err) reject(err);
        else resolve(data);
      });
      fn.apply(null, args);
    });
  };
}

const funcPromisify = promisify(func);

funcPromisify(1, 2).then((val) => {
  console.log(val);
});

```

### callbackify

```js
const callbackify = (promiseCreator) => {
  return (...args) => {
    const arg = args.slice(0, -1);
    const cb = args.slice(-1)[0];
    promiseCreator(...arg)
      .then((val) => {
        cb(null, val);
      })
      .catch((err) => {
        cb(err, null);
      });
  };
};
```

### co

```js
function ajax(url) {
  return Promise.resolve(url);
}

function* main() {
  let res1 = yield ajax("https://www.baidu.com");
  console.log(res1);

  let res2 = yield ajax("https://www.google.com");
  console.log(res2);

  let res3 = yield ajax("https://www.taobao.com");
  console.log(res3);
}

function co(generator) {
  const g = generator();
  function handleResult(result) {
    if (result.done) return;
    result.value.then(
      (res) => {
        handleResult(g.next(res));
      },
      (e) => {
        g.throw(e);
      }
    );
  }
  handleResult(g.next());
}

co(main);
```

### await

```js
const getData = () =>
  new Promise((resolve) => setTimeout(() => resolve("data"), 1000));

const test = asyncToGenerator(function* testG() {
  const data = yield getData();
  console.log("data: ", data);
  const data2 = yield Promise.reject(222);
  console.log("data2: ", data2);
  return "success";
});

// 这样的一个函数 应该再1秒后打印data 再过一秒打印data2 最后打印success
test().then((res) => {
  console.log(res);
});

function asyncToGenerator(generatorFunc) {
  return function() {
    const gen = generatorFunc.apply(this, arguments);
    return new Promise((resolve, reject) => {
      function step(operate, val) {
        let genRes;
        try {
          genRes = gen[operate](val);
        } catch (err) {
          return reject(err);
        }
        const { value, done } = genRes;
        if (done) {
          resolve(value);
        } else {
          Promise.resolve(value).then(
            (val) => step("next", val),
            (err) => step("throw", err)
          );
        }
      }
      step("next");
    });
  };
}
```

## 数组

### filter

```js
Array.prototype.filter = function(filterCb, thisArg) {
  const res = [];
  const arr = this;
  for (let i = 0; i < arr.length; i++) {
    if (filterCb.call(thisArg, arr[i], i, arr)) {
      res.push(arr[i]);
    }
  }
  return res;
};

console.log([1, 2, 3].filter((v) => v > 1));
```

### flat

```js
function flat(arr, depth = Infinity) {
  if (depth === 0) return arr;
  return arr.reduce(
    (acc, cur) => acc.concat(Array.isArray(cur) ? flat(cur, depth - 1) : cur),
    []
  );
}

console.log(flat([1, 2, [3, 4, 5, [6, 7, 8]]], 1));

let arr = [1, [2, [3, 4], 5], 6];
let str = JSON.stringify(arr);
console.log(str.replace(/\[|\]/g, "").split(","));
console.log(JSON.parse("[" + str.replace(/\[|\]/g, "") + "]"));
```

## 其他

### Object.is

```js
function is(x, y) {
  if (x === y) {
    if (x !== 0 && y !== 0) return true;
    // x和y相等的情况下，处理+0和-0的情况
    else return 1 / x === 1 / y;
  } else {
    // x和y不相等的情况下，处理NaN的情况
    return x !== x && y !== y;
  }
}

console.log(is(+0, -0));
console.log(is(NaN, NaN));

```

### deepClone

```js
function isReferenceType(o) {
  return o instanceof Object;
}

/**
 * 1. 判断是否引用类型，如果不是直接返回
 * 2. 针对正则、函数和Date做异常处理
 * 3. 获取到原对象的constructor，创建新对象
 * 4. 引入WeakMap解决循环引用问题
 * 5. 遍历原对象中的数据，将数据通过深拷贝的方式赋值给新对象
 */
function deepClone(obj, hash = new WeakMap(), parent) {
  if (!isReferenceType(obj)) {
    return obj;
  }

  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  // 传入父级节点解决负责的函数this指向丢失的情况
  if (obj instanceof Function) return obj.bind(parent);

  const newObj = new obj.constructor();

  // 解决循环引用问题
  if (hash.has(obj)) {
    return hash.get(obj);
  }
  hash.set(obj, newObj);

  // Reflect.ownKeys可将对象上的可枚举和不可枚举以及symbol都访问到
  Reflect.ownKeys(obj).forEach((key) => {
    newObj[key] = deepClone(obj[key], hash, newObj);
  });

  return newObj;
}

function add(a, b, c) {
  return a + b + c;
}

const obj1 = {
  a: 1,
  b: 2,
  date: new Date(),
  arr: [1, 2, 3],
  func: add.bind({}, 1),
};
obj1.obj = obj1;

console.log(deepClone(obj1).func(2, 3));
```

### JSONP

```js
function JSONP(url) {
  return new Promise((resolve, reject) => {
    const callbackName = `callback_${Date.now()}`;
    const script = document.createElement('script');

    script.src = `${url}${url.indexOf('?') > -1 ? '&' : '?'}callback=${callbackName}`;
    document.body.appendChild(script);
    window[callbackName] = function (res) {
      delete window[callbackName];
      script.remove();
      resolve(res);
    };
    script.onerror = function (e) {
      delete window[callbackName];
      script.remove();
      reject(e);
    };
  });
}
```

### ajax

```js
const addUrl = (_url, param) => {
  let url = _url;
  if (param && Object.keys(param).length) {
    url += url.indexOf("?") === -1 ? "?" : "&";
    Object.keys(param).forEach((key) => {
      url += `${encodeURIComponent(key)}=${encodeURIComponent(param[key])}`;
    });
  }
  return url;
};

function ajax({
  url = "",
  method = "GET",
  data = {},
  header = {},
  async = false,
  timeout = 5 * 1000,
  onSuccess,
  onError,
  onTimeout,
}) {
  const requestURL = method === "GET" ? addUrl(url, data) : url;
  const sendData = method === "GET" ? null : data;

  const xhr = new XMLHttpRequest();
  xhr.open(method, requestURL, async);

  if (header && Object.keys(header).length) {
    Object.keys(header).forEach((key) => {
      xhr.setRequestHeader(key, header[key]);
    });
  }

  xhr.onload = () => {
    try {
      if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
        onSuccess(xhr.responseText);
      } else {
        onError(xhr.status + xhr.statusText);
      }
    } catch (err) {
      onError(err);
    }
  };
  xhr.timeout = timeout;
  xhr.ontimeout =
    onTimeout ||
    function() {
      console.error("request timeout");
    };

  xhr.send(sendData);
}
```

### node require

```js
const vm = require("vm");
const path = require("path");
const fs = require("fs");

function customRequire(filePath) {
  const pathToFile = path.resolve(__dirname, filePath);
  const content = fs.readFileSync(pathToFile, "utf-8");

  const wrapper = ["(function(require, module, exports) {", "})"];
  const wrappedContent = wrapper[0] + content + wrapper[1];
  console.log("wrappedContent: ", wrappedContent);

  const script = new vm.Script(wrappedContent, {
    filename: "index.js",
  });
  const module = {
    exports: {},
  };
  const func = script.runInThisContext();
  func(customRequire, module, module.exports);
  return module.exports;
}

const { add } = customRequire("./module.js");
console.log(add(1, 2));

```

### eventEmitter

```js
class EventEmitter {
  constructor() {
    this.events = {};
    this.count = 0;
  }

  subscribe(type, listener) {
    this.events[type] = this.events[type] || [];
    this.events[type].push(listener);

    return this.unsubscribe.bind(this, type, listener);
  }

  unsubscribe(type, listener) {
    if (this.events[type]) {
      this.events[type] = this.events[type].filter((cb) => cb !== listener);
    }
  }

  publish(type, ...args) {
    if (this.events[type]) {
      [...this.events[type]].forEach((cb) => {
        cb.call(this, ...args);
      });
    }
  }
}

const event = new EventEmitter();
event.subscribe("daily", function() {
  // 校验call的绑定
  console.log(`there has already ${this.count} subscribers`);
  console.log("bob");
});

event.subscribe("evening", () => {
  console.log("alice");
});

event.subscribe("noon", () => {
  console.log("Jim");
});

const JimEveningFn = function() {
  console.log("Jim");
};
event.subscribe("evening", JimEveningFn);

event.publish("daily");
console.log("****");
event.publish("evening");
console.log("****");
event.publish("noon");
```

### eventEmitterProxy

```js
class Observer {
  constructor(target) {
    this.observers = [];
    this.target = new Proxy(target, {
      set: (target, key, value, receiver) => {
        const result = Reflect.set(target, key, value, receiver);
        [...this.observers].forEach((cb) => cb(target, key, value, receiver));
        return result;
      },
    });

    return [this.target, this.observe];
  }

  observe = (fn) => {
    this.observers.push(fn);
    return this.unobserve.bind(null, fn);
  };

  unobserve = (fn) => {
    this.observers = this.observers.filter((o) => o !== fn);
  };
}

let [person, observe] = new Observer({ name: 1, age: 2 });
let unobserve = observe((target, key, value) => {
  console.log("target, key, value: ", target, key, value);
});

person.name = "bob"; // target, key, value:  { name: 'bob', age: 2 } name bob

unsubscribe();
person.name = "aaa"; // 无输出
```

### shuffle

```js
function shuffle(arr) {
  for (let i = 0; i < arr.length; i++) {
    let changeIndex = Math.floor(Math.random() * i);
    [arr[i], arr[changeIndex]] = [arr[changeIndex], arr[i]];
  }
  return arr;
}

let arr2 = [1, 2, 3, 4, 5, 6];
console.log(shuffle(arr2));
```

### sleep

```js
function sleep (time) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve()
    }, time);
  })
}

async function main(){
  console.log(1)
  await sleep(2000)
  console.log(2)
}

main()
```
