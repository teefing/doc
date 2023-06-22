---
category: 专业知识
tags:
  - Vue
  - 设计模式
date: 2020-12-19
title: 自己动手实现Vue响应式
vssue-title: 自己动手实现Vue响应式
---

> Vue 响应式 = 发布订阅模式 + 代理模式
## Vue中使用方式
先看一段 vue 代码

```html
<div id="app">
  <div>Price: ${{ price }}</div>
  <div>Total: ${{ price * quantity }}</div>
  <div>Taxes: ${{ totalPriceWithTax }}</div>
<div>
```

```html
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<script>
	var vm = new Vue({
	  el: '#app',
	  data: {
	    price: 5.00,
	    quantity: 2
	  },
	  computed: {
	    totalPriceWithTax() {
	      return this.price * this.quantity * 1.03
	    }
	  }
	})
</script>
```

每当price改变时，发生了三件事

1. 更新页面中的price
2. 页面中重新计算price * quantity 并更新
3. 重新计算totalPriceWithTax 并更新页面中的totalPriceWithTax

那么vue是如何自动做上面三件事的呢

接下来我们一步一步实现它

## 引入副作用概念
首先我们告诉程序，这里有一个副作用/计算方法，存起来我会在数据更新的时候调用它

```jsx
let target = null;
let price = 10;
let quantity = 2;
let total = 0;

// 这是计算方法，也可以理解为price的副作用函数之一
target = () => {
  total = price * quantity;
};

// 存计算方法的地方，可以理解为副作用函数列表
let storage = [];
const record = () => {
  storage.push(target);
};

// 之后在数据修改后调用
const replay = () => {
  storage.forEach((run) => run());
};

// 存计算方法
record();
// 先执行一遍计算方法获得total
target();

console.log(total); // 20
// 修改price
price = 20;
console.log(total); // 20
// price被修改，产生副作用，执行一遍副作用列表内的函数
replay();

// 得到price修改副作用生效后的新的total
console.log(total); // 40

```

## 发布订阅模式，引入依赖概念

接下来将与副作用相关的函数整合为Dep对象内，Dep是dependency的缩写，依赖的意思，一个数据的副作用本质上就是数据修改导致副作用被执行，那么该副作用函数也就相当于是该数据的依赖项，副作用函数的执行依赖于数据的修改。Dep也是vue内部的命名定义，这里准确理解Dep的含义是很重要的，框架中大部分变量的命名都是有意义的，包含了作者编写框架时的思路。

```jsx
let target = null;
class Dep {
  constructor() {
    // 存计算方法的地方，可以理解为副作用函数列表
		// subscribers的缩写
    this.subs = [];
  }

  // 添加依赖
  depend() {
    if (target && !this.subs.includes(target)) {
      this.subs.push(target);
    }
  }
  // 之后在数据修改后调用
  notify() {
    this.subs.forEach((sub) => sub());
  }
}

let dep = new Dep();

let price = 10;
let quantity = 2;
let total = 0;

// 这是计算方法，也可以理解为price的副作用函数之一
target = () => {
  total = price * quantity;
};

// 存计算方法
dep.depend();
// 先执行一遍计算方法获得total
target();

console.log(total); // 20
// 修改price
price = 20;
console.log(total); // 20
// price被修改，产生副作用，执行一遍副作用列表内的函数
dep.notify();

// 得到price修改副作用生效后的新的total
console.log(total); // 40
```

代码整理过变得清爽多了，与依赖/副作用相关的方法被聚在了Dep类中，这里与第一部分的代码相比方法名和变量更改了，为了方便之后的理解。可以看到Dep类其实就是一个典型的发布订阅者模式，通过depend方法依赖们对主体进行订阅，通过notify方法主体对订阅者（也就是依赖）进行消息通知

## 整理代码，引入观察者概念
接下来在Dep类外部进行依赖添加的部分也可以抽象，因为从依赖的视角，每个依赖项其实也是一个个观察者，观察price（依赖的值，这里以price举例）的变化，因此定义一个watch方法，进行依赖的订阅操作

```jsx
let target = null;
class Dep {
  constructor() {
    // 存计算方法的地方，可以理解为副作用函数列表
    this.subs = [];
  }

  // 添加依赖
  depend() {
    if (target && !this.subs.includes(target)) {
      this.subs.push(target);
    }
  }
  // 之后在数据修改后调用
  notify() {
    this.subs.forEach((sub) => sub());
  }
}

let dep = new Dep();

let price = 10;
let quantity = 2;
let total = 0;

function watch(func) {
  target = func;
  // 存计算方法 依赖收集
  dep.depend();
  // 先执行一遍计算方法获得total
  target();
  // 将target设置为null，供其它响应式数据使用
  target = null;
}

// 这是计算方法，也可以理解为price的副作用函数之一
watch(() => {
  total = price * quantity;
});

console.log(total); // 20
// 修改price
price = 20;
console.log(total); // 20
// price被修改，产生副作用，执行一遍副作用列表内的函数
dep.notify();

// 得到price修改副作用生效后的新的total
console.log(total); // 40
```

这里进行了一点优化，在watch方法尾部将target置空了，也就是恢复原状了，防止出现未知bug

## 数据对象化
接下来将依赖存入对象data

```jsx
let target = null;
class Dep {
  constructor() {
    // 存计算方法的地方，可以理解为副作用函数列表
    this.subs = [];
  }

  // 添加依赖
  depend() {
    if (target && !this.subs.includes(target)) {
      this.subs.push(target);
    }
  }
  // 之后在数据修改后调用
  notify() {
    this.subs.forEach((sub) => sub());
  }
}

let dep = new Dep();

let data = { price: 10, quantity: 2 };
let total = 0;

function watch(func) {
  target = func;
  // 存计算方法 依赖收集
  dep.depend();
  // 先执行一遍计算方法获得total
  target();
  // 将target设置为null，供其它响应式数据使用
  target = null;
}

// 这是计算方法，也可以理解为price的副作用函数之一
watch(() => {
  total = data.price * data.quantity;
});

console.log(total); // 20
// 修改price
data.price = 20;
console.log(total); // 20
// price被修改，产生副作用，执行一遍副作用列表内的函数
dep.notify();

// 得到price修改副作用生效后的新的total
console.log(total); // 40
```

## 代理模式，defineProperty
在上面的代码中，副作用我们是通过`dep.notify`手动触发的，那么我们怎样实现修改了data.price后自动触发呢，这里我们引入代理模式，对data的赋值操作进行代理，js中实现代理模式的方法就莫属Object.defineProperty/Proxy了，这里先讲Object.defineProperty的实现方式，vue2就是采用这种方式的。

```jsx
let target = null;
class Dep {
  constructor() {
    // 存计算方法的地方，可以理解为副作用函数列表
    this.subs = [];
  }

  // 添加依赖
  depend() {
    if (target && !this.subs.includes(target)) {
      this.subs.push(target);
    }
  }
  // 之后在数据修改后调用
  notify() {
    this.subs.forEach((sub) => sub());
  }
}
function watch(func) {
  target = func;
  // 先执行一遍计算方法获得total
  target();
  // 将target设置为null，供其它响应式数据使用
  target = null;
}

let data = { price: 10, quantity: 2 };
let total = 0;

Object.keys(data).forEach((key) => {
  let dep = new Dep();
  let internalValue = data[key];
  Object.defineProperty(data, key, {
    get() {
			// 存计算方法 依赖收集
      dep.depend();
      return internalValue;
    },
    set(newVal) {
      internalValue = newVal;
      // price被修改，产生副作用，执行一遍副作用列表内的函数
      dep.notify();
    },
  });
});

// 这是计算方法，也可以理解为price的副作用函数之一
watch(() => {
  total = data.price * data.quantity;
});

console.log(total); // 20
// 修改price
data.price = 20;

// 得到price修改副作用生效后的新的total
console.log(total); // 40

setTimeout(() => {
  data.price = 30;
  console.log(total); // 60
}, 1000);
```

注意，上面我们把依赖收集的逻辑移入了get方法中，把通知执行副作用函数的逻辑移入了set方法中，这样在调用watch方法时，内部会触发属性的读操作，从而触发依赖收集，在设置值的时候，会触发属性的写操作，从而再次执行副作用，更新total的值

到此为止，一个纯数据层面的响应式系统已经完成了 😆，可以看到，我们用了发布订阅模式和代理模式搭配使用，很快便搭好了一个简单的响应式系统。


## 数据与视图绑定
但是，vue中我们修改数据是能够实现页面数据的刷新的，接下来我们来探讨如何将数据与视图层面进行绑定。

首先我们创建一个html，dom结构就照着vue中的template来

```jsx
<body>
  <div id="app">
    {{name}}
    <h2>{{age}}</h2>
    <input type="text" v-model="name" />
  </div>
</body>
```

### Reactive类
接下来创建一个类似vue的Reactive类

这个类中我们主要做两件事情

1. 将我们传入的data变为响应式的
2. 编译body中的模版，将页面与数据关联

```jsx
class Reactive {
  constructor(options) {
    this.options = options;
    // 使data内的数据变为响应式
    this.$data = observe(this.options.data);
    this.el = document.querySelector(this.options.el);
		// 将模板编译，数据和视图绑定
    this.compile(this.el);
  }
}
```

对Reactive类的使用与vue类似

```jsx
let vm = new Reactive({
  el: "#app",
  data: {
    name: "飞",
    age: 23
  }
});
```

### observe
接下来先实现observe方法，其实就是把上面硬编码的逻辑抽成函数，并且加上递归优化

```jsx
function observe (data) {
  if(typeof data !== 'object' || data === null) return
  Object.keys(data).forEach((key) => {
    let dep = new Dep();
    let internalValue = data[key];
		// 递归使整个对象都变成响应式
    observe(data[key])
    Object.defineProperty(data, key, {
      get () {
        // 依赖注入
        dep.depend()
        return internalValue;
      },
      set(newVal) {
        internalValue = newVal;
        // 数据被修改，产生副作用，执行一遍副作用列表内的函数
        dep.notify();
      },
    });
  });
}
```
### compile
下面实现compile方法

```jsx
compile(el) {
  // 取出子节点
  let child = el.childNodes;
  // 遍历子节点
  [...child].forEach((node) => {
    // 如果是文本节点
    if (node.nodeType === Node.TEXT_NODE) {
      let text = node.textContent;
      let reg = /{{\s*([^\s{}]+)\s*}}/;
      // 如果文本内容符合 {{xxx}} 的形式
      if (reg.test(text)) {
        let $1 = RegExp.$1;
        // 如果data中有xxx，则用data中的数据替换xxx
        // 监听xxx，如果xxx发生更改，修改dom的内容
        this.$data[$1] && watch(() => {
          node.textContent = text.replace(reg, this.$data[$1]);
        });
      }
    } else if (node.nodeType === Node.ELEMENT_NODE) {
      // 如果是普通元素节点
      let attr = node.attributes;

      // 如果属性中存在v-model
      if (attr.hasOwnProperty("v-model")) {
        // 得到v-model属性节点的值
        let keyName = attr["v-model"].nodeValue;
        // 将元素节点的值修改
        node.value = this.$data[keyName];
        // 监听元素节点的input事件，input后修改data中的数据
        node.addEventListener("input", (e) => {
          this.$data[keyName] = node.value;
        });
      }
    }
    // 递归对子节点处理
    this.compile(node);
  });
}
```

代码逻辑整体上还是比较简单的，遍历子节点，如果是文本节点，那么看是否符合`{{xxx}}`这种形式，如果符合，并且data当中存在xxx属性，那么就将`{{xxx}}`替换为data中的数据，并且将这一操作作为副作用加入依赖中（watch中这些操作都做了）；如果是元素节点，那么判断其属性中是否存在v-model，如果存在v-model，就监听元素的input事件，当input时就将data中对应的数据进行修改，因为data是响应式的，所以修改了对于数据后，页面上与之关联的文本节点也会更新数据

### 完整代码
下面是完整代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
  </head>
  <body>
    <div id="app">
      {{name}}
      <h2>{{age}}</h2>
      <input type="text" v-model="name" />
    </div>
  </body>
</html>
<script>
  let target = null;
  class Dep {
    constructor() {
      // 存计算方法的地方，可以理解为副作用函数列表
      this.subs = [];
    }

    // 添加依赖
    depend() {
      if (target && !this.subs.includes(target)) {
        this.subs.push(target);
      }
    }
    // 之后在数据修改后调用
    notify() {
      this.subs.forEach((sub) => sub());
    }
  }

  function watch(func) {
    target = func;
    // 先执行一遍计算方法获得total
    target();
    // 将target设置为null，供其它响应式数据使用
    target = null;
  }

  function observe(data) {
    if (typeof data !== "object" || data === null) return;
    Object.keys(data).forEach((key) => {
      let dep = new Dep();
      let internalValue = data[key];
      observe(data[key]);
      Object.defineProperty(data, key, {
        get() {
          // 依赖注入
          dep.depend();
          return internalValue;
        },
        set(newVal) {
          internalValue = newVal;
          // 数据被修改，产生副作用，执行一遍副作用列表内的函数
          dep.notify();
        },
      });
    });
  }
  class Reactive {
    constructor(options) {
      this.options = options;
      // 使data内的数据变为响应式
      this.$data = this.options.data;
      observe(this.$data);
      this.el = document.querySelector(this.options.el);
      this.compile(this.el);
    }

    compile(el) {
      // 取出子节点
      let child = el.childNodes;
      // 遍历子节点
      [...child].forEach((node) => {
        // 如果是文本节点
        if (node.nodeType === Node.TEXT_NODE) {
          let text = node.textContent;
          let reg = /{{\s*([^\s{}]+)\s*}}/;
          // 如果文本内容符合 {{xxx}} 的形式
          if (reg.test(text)) {
            let $1 = RegExp.$1;
            // 如果data中有xxx，则用data中的数据替换xxx
            // 监听xxx，如果xxx发生更改，修改dom的内容
            this.$data[$1] &&
              watch(() => {
                node.textContent = text.replace(reg, this.$data[$1]);
              });
          }
        } else if (node.nodeType === Node.ELEMENT_NODE) {
          // 如果是普通元素节点
          let attr = node.attributes;

          // 如果属性中存在v-model
          if (attr.hasOwnProperty("v-model")) {
            // 得到v-model属性节点的值
            let keyName = attr["v-model"].nodeValue;
            // 将元素节点的值修改
            node.value = this.$data[keyName];
            // 监听元素节点的input事件，input后修改data中的数据
            node.addEventListener("input", (e) => {
              this.$data[keyName] = node.value;
            });
          }
        }
        // 递归对子节点处理
        this.compile(node);
      });
    }
  }
</script>
<script>
  let vm = new Reactive({
    // 挂载元素
    el: "#app",
    data: {
      name: "飞",
      age: 23,
    },
  });
</script>
```
## 使用Proxy优化
上面我们提到，在js中实现代理模式可以通过defineProperty，也可以通过Proxy，那么这两者有什么区别呢，vue3又为何将defineProperty替换为Proxy呢，我们继续探究

我们引入数组数据并且添加mounted生命周期钩子

```jsx
...
  <body>
    <div id="app">
      {{name}}
      <h2>{{age}}</h2>
      <input type="text" v-model="name" />
      {{arr}}
    </div>
  </body>
...
  class Reactive {
    constructor(options) {
      this.options = options;
      // 使data内的数据变为响应式
      this.$data = this.options.data;
      observe(this.$data);
      this.el = document.querySelector(this.options.el);
      this.compile(this.el);

      this.$mounted = this.options.mounted
      this.$mounted.call(this)
    }
...
  let vm = new Reactive({
    // 挂载元素
    el: "#app",
    data: {
      name: "飞",
      age: 23,
      arr: [0,1,2]
    },
		mounted(){
      setTimeout(() => {
        this.$data.arr[3] = 3
        console.log('this.$data.arr[3] = 3: ', this.$data.arr[3] = 3);
      }, 1000);
    }
  });
</script>
```

结果发现在一秒后，data中的arr[3]数据的确被修改了，但是页面上的数据还是没变化，这说明我们的数组并非响应式的。这个问题在vue中是老生常谈的问题了，虽然vue通过拦截push，pop等操作一定程度上实现了数组的响应式，可是对于`this.$data.arr[3] = 3`这种通过下标索引直接赋值的操作是做不到可响应的。以上现象的原因是受限于defineProperty，无法对数组内元素的直接操作进行监听。其实很多依赖于defineProperty的响应式库都有这个问题，mobx中的解决方式就是对于数组创建0-999项，将这1000项全变成响应式的，因此在使用mobx时，明明在需求层面，我们的列表中只有若干项，可是我们在打印数组时会打印出1000个数据。

Proxy相比于defineProperty，一个显著的优点就是可以通过下标监听数组内元素的变化了，接下来我们使用Proxy进行优化

```jsx
...
  // 使data变为响应式
  function observe(data) {
    if (typeof data !== "object" || data === null) {
      return data;
    }

    // 将data中的子对象也变为响应式
    let val;
    Object.keys(data).forEach((key) => {
      val = data[key];
      data[key] = observe(val);
    });
    const dep = new Dep();
    return new Proxy(data, {
      get(target, key, receiver) {
        dep.depend(); // 依赖注入
        return Reflect.get(target, key, receiver);
      },
      set(target, key, val, receiver) {
        Reflect.set(target, key, val, receiver);
        dep.notify(); // 执行
        return true;
      },
    });
  }
  class Reactive {
    constructor(options) {
      this.options = options;
      // 使data内的数据变为响应式
      this.$data = observe(this.options.data);
      this.el = document.querySelector(this.options.el);
      this.compile(this.el);

      this.$mounted = this.options.mounted;
      this.$mounted.call(this);
    }
...
```

刷新页面我们可以看到在一秒后页面中的`0,1,2`变成了`0,1,2,3` 这说明对于数组我们监听成功

下面是Proxy版本的完整代码

```jsx
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
  </head>
  <body>
    <div id="app">
      {{name}}
      <h2>{{age}}</h2>
      <input type="text" v-model="name" />
      {{arr}}
    </div>
  </body>
</html>
<script>
  let target = null;
  class Dep {
    constructor() {
      // 存计算方法的地方，可以理解为副作用函数列表
      this.subs = [];
    }

    // 添加依赖
    depend() {
      if (target && !this.subs.includes(target)) {
        this.subs.push(target);
      }
    }
    // 之后在数据修改后调用
    notify() {
      this.subs.forEach((sub) => sub());
    }
  }

  function watch(func) {
    target = func;
    // 先执行一遍计算方法获得total
    target();
    // 将target设置为null，供其它响应式数据使用
    target = null;
  }

  // 使data变为响应式
  function observe(data) {
    if (typeof data !== "object" || data === null) {
      return data;
    }

    // 将data中的子对象也变为响应式
    let val;
    Object.keys(data).forEach((key) => {
      val = data[key];
      data[key] = observe(val);
    });
    const dep = new Dep();
    return new Proxy(data, {
      get(target, key, receiver) {
        dep.depend(); // 依赖注入
        return Reflect.get(target, key, receiver);
      },
      set(target, key, val, receiver) {
        Reflect.set(target, key, val, receiver);
        dep.notify(); // 执行
        return true;
      },
    });
  }
  class Reactive {
    constructor(options) {
      this.options = options;
      // 使data内的数据变为响应式
      this.$data = observe(this.options.data);
      this.el = document.querySelector(this.options.el);
      this.compile(this.el);

      this.$mounted = this.options.mounted;
      this.$mounted.call(this);
    }

    compile(el) {
      // 取出子节点
      let child = el.childNodes;
      // 遍历子节点
      [...child].forEach((node) => {
        // 如果是文本节点
        if (node.nodeType === Node.TEXT_NODE) {
          let text = node.textContent;
          let reg = /{{\s*([^\s{}]+)\s*}}/;
          // 如果文本内容符合 {{xxx}} 的形式
          if (reg.test(text)) {
            let $1 = RegExp.$1;
            // 如果data中有xxx，则用data中的数据替换xxx
            // 监听xxx，如果xxx发生更改，修改dom的内容
            this.$data[$1] &&
              watch(() => {
                node.textContent = text.replace(reg, this.$data[$1]);
              });
          }
        } else if (node.nodeType === Node.ELEMENT_NODE) {
          // 如果是普通元素节点
          let attr = node.attributes;

          // 如果属性中存在v-model
          if (attr.hasOwnProperty("v-model")) {
            // 得到v-model属性节点的值
            let keyName = attr["v-model"].nodeValue;
            // 将元素节点的值修改
            node.value = this.$data[keyName];
            // 监听元素节点的input事件，input后修改data中的数据
            node.addEventListener("input", (e) => {
              this.$data[keyName] = node.value;
            });
          }
        }
        // 递归对子节点处理
        this.compile(node);
      });
    }
  }
</script>
<script>
  let vm = new Reactive({
    // 挂载元素
    el: "#app",
    data: {
      name: "飞",
      age: 23,
      arr: [0, 1, 2],
    },
    mounted() {
      setTimeout(() => {
        this.$data.arr[3] = 3;
        console.log("this.$data.arr[3] = 3: ", (this.$data.arr[3] = 3));
      }, 1000);
    },
  });
</script>
```

终于我们用了一百行左右的代码实现了一个类似vue的响应式框架 😜