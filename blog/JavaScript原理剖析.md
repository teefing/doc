---
title: JavaScript原理剖析
category: 专业知识
date: 2019-06-26 21:22:05
tags:
  - JavaScript
---

## `new`

> new 关键字作为 JS 中通过正常途径实例化一个类的唯一方法，经常出现在我们的视线中，那么其原理又是如何呢？它到底做了什么事情？

先看一段代码

```javascript
// 定义类 类名字是 classA
function classA() {
  this.name = 1;
}
classA.prototype.show = function() {
  alert(this.name);
};
// 用new实例化
var b = new classA();
b.show();
```

这是一个常见的使用 new 创建实例的例子

new 其实干了如下几件事

1. 创建了一个对象 a(a 是随便取的)
2. 将该对象的**proto**属性设置为 classA 的 prototype
3. 将 classA 中的 this 指向对象 a，也就是之后的`this.name=1`等同于`a.name=1`
4. 执行 classA 函数
5. 将对象 a 返回出去

接下来通过 JS 模拟一下 new 的操作

```javascript
function classA() {
  this.name = "start";
}
classA.prototype.sayName = function() {
  console.log(this.name);
};

function NEW(className) {
  let res = {}; // 新建一个对象
  res.__proto__ = className.prototype; // 通过原型链继承classA的方法
  className.call(res); // 通过call修改classA的this指向，并且调用classA这个函数，在这个过程中内部的name属性会被赋值
  return res;
}
let a = NEW(classA);
a.sayName();
a.name = "a";
a.sayName();
```

以上就是`new`的作用，如果在实例化的时候忘记使用`new`了，其实就是简单地调用了一下那个函数，函数内部的 this 其实指向的是外部，

```javascript
function classA() {
  this.name = "start";
}
classA.prototype.sayName = function() {
  console.log(this.name);
};
let a = classA();
try {
  a.sayName();
} catch (e) {
  console.log(e);
}
console.log(name);
```

上述代码运行后会发生报错，打印的`name`是`start`，可见`this`默认被绑到了外部（浏览器端为 window，nodejs 端为 global）

> 注意以上是在函数体内，如果在全局环境中浏览器的依然为`window`，但是 nodejs 端为`{}`

由以上的所讲的`new`的原理，进一步，我们可以解释如下现象

```js
function classA() {
  this.name = "start";
  let obj = { a: 1 };
  return obj;
}
classA.prototype.sayName = function() {
  console.log(this.name);
};
let a = classA();
console.log(a.a); // 1
a.sayName(); // 报错
```

因为在`return`最后的`res`之前，使用`call`调用了`classA`这个函数，结果直接返回出来了，因此所获得的并不是我们想要的结果

> 构造函数中绝对不能 return 数据

## `浅拷贝`VS`深拷贝`

```js
function isReferenceType (o) {
  return o instanceof Object
}

function deepClone (obj) {
  if (!isReferenceType(obj)) {
    return obj
  }

  if (obj instanceof RegExp) return new RegExp(obj)
  if (obj instanceof Function) return obj.bind({})

  let newObj = new obj.constructor()
  Reflect.ownKeys(obj).forEach(key => {
    newObj[key] = deepClone(obj[key])
  })

  return newObj
}

let arr = [1, 2, 3];
arr.push(arr);
var obj = {
  a: 1,
  arr,
  func: () => {
    console.log("aaa");
  },
  date: new Date(),
  reg: new RegExp(/test/),
  none: undefined,
};
obj.obj = obj;
console.log(deepClone(obj));
//   return JSON.parse(JSON.stringify(obj))
// }
// console.log(jsonDeepClone({
//   a: 1,
//   func: ()=>{
//     console.log('aaa');
//   },
//   date: new Date(),
//   reg: new RegExp(/test/),
//   none: undefined
// }));
```

## `call`和`apply`

```js
Function.prototype.call = function() {
  let [context, ...args] = [...arguments];
  if (!context) {
    context = typeof window === "undefined" ? global : window;
  }
  // 注意，这里必须要context.func 不能是一个简单的变量，因为需要给函数一个运行的上下文，即context
  context.func = this;
  let res = context.func(...args);
  delete context.func;
  // context中实际并无func这个key，要删除恢复原样
  return res;
};
Function.prototype.apply = function() {
  let [context, args] = arguments;
  if (!context) {
    context = typeof window === "undefined" ? global : window;
  }
  context.func = this;
  let res;
  if (!args) res = context.func();
  else res = context.func(...args);
  delete context.func;
  return res;
};

function getName(age, address) {
  console.log(this.name);
  console.log({ age, address });
}
let obj = {
  name: "teefing",
};
getName.apply(obj, [22, "hangzhou"]);
getName.call(obj, 22, "hangzhou");
```

## `函数柯里化`

```js
const curry = (fn, ...args) =>
  args.length < fn.length
    ? (...arguments) => curry(fn, ...args, ...arguments)
    : fn(...args);
function sumFn(a, b, c) {
  return a + b + c;
}
var sum = curry(sumFn);
console.log(sum(1)(2)(3)); //6

let add1 = sum(1);
let add2 = add1(2);
let res = add2(3);
console.log(res); //6
```

> 注： `Function.length`返回一个函数的`参数个数`，但是`剩余参数`不会算上，还有`第一个默认参数及其之后的参数`不会被算上

> 函数柯里化主要作用

1. 参数复用
2. 提前返回 – 返回接受余下的参数且返回结果的新函数
3. 延迟执行 – 返回新函数，等待执行

## `flat`

```js
let flattenDeep1 = function(arr, deepLength) {
  return arr.flat(deepLength, Math.pow(2, 53) - 1);
};

let flattenDeep2 = (arr) =>
  arr.reduce(
    (acc, val) =>
      Array.isArray(val) ? acc.concat(flattenDeep2(val)) : acc.concat(val),
    []
  );

let flattenDeep3 = (arr) => {
  let stack = [...arr];
  let res = [];
  while (stack.length) {
    let next = stack.shift();
    if (Array.isArray(next)) {
      res = res.concat(flattenDeep4(next));
    } else {
      res.push(next);
    }
  }
  return res;
};

let flattenDeep4 = (arr) => {
  let stack = [...arr];
  let res = [];
  while (stack.length) {
    const next = stack.shift();
    if (Array.isArray(next)) {
      stack.unshift(...next);
    } else {
      res.push(next);
    }
  }
  return res;
};
[flattenDeep1, flattenDeep2, flattenDeep3, flattenDeep4].forEach(
  (flattenDeep, index) => {
    try {
      console.log(flattenDeep([1, 2, [3, 4, 5, [6, 7, 8]]]));
    } catch (e) {
      console.log(`method-${index} not supported`);
    }
  }
);
```

## `uniq`函数

```js
let uniq1 = (arr) => [...new Set(arr)];

let uniq2 = (arr) => {
  let res = [];
  for (let i = 0; i < arr.length; i++) {
    if (res.indexOf(arr[i]) === -1) res.push(arr[i]);
  }
  return res;
};

let uniq3 = (arr) => {
  let res = [];
  for (let i = 0; i < arr.length; i++) {
    if (!res.includes(arr[i])) {
      res.push(arr[i]);
    }
  }
  return res;
};

let uniq4 = (arr) =>
  arr.reduce((acc, val) => (acc.includes(val) ? acc : [...acc, val]), []);

let uniq5 = (arr) => {
  let m = new Map();
  let res = [];
  for (let i = 0; i < arr.length; i++) {
    if (m.has(arr[i])) m.set(arr[i], true);
    // true和false可以判断是否只存在一次
    else {
      m.set(arr[i], false);
      res.push(arr[i]);
    }
  }
  return res;
};
[uniq1, uniq2, uniq3, uniq4, uniq5].forEach((uniq, index) => {
  console.log(uniq([1, 1, 2, 2, 3, 3, 4]));
});
```
