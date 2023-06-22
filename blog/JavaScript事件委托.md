---
title: JavaScript事件委托
date: 2017-11-19
tags:
  - JavaScript
category: 专业知识
---

# JavaScript 事件委托

事件委托就是利用事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。

# 为什么要用事件委托

一般来说，dom 需要有事件处理程序，我们都会直接给它设事件处理程序就好了，那如果是很多的 dom 需要添加事件处理呢？比如我们有 100 个 li，每个 li 都有相同的 click 点击事件，可能我们会用 for 循环的方法，来遍历所有的 li，然后给它们添加事件，那这么做会存在什么影响呢？

在 JavaScript 中，添加到页面上的事件处理程序数量将直接关系到页面的整体运行性能，因为需要不断的与 dom 节点进行交互，访问 dom 的次数越多，引起浏览器重绘与重排的次数也就越多，就会延长整个页面的交互就绪时间，这就是为什么性能优化的主要思想之一就是减少 DOM 操作的原因；如果要用事件委托，就会将所有的操作放到 js 程序里面，与 dom 的操作就只需要交互一次，这样就能大大的减少与 dom 的交互次数，提高性能；

每个函数都是一个对象，是对象就会占用内存，对象越多，内存占用率就越大，自然性能就越差了（内存不够用，是硬伤，哈哈），比如上面的 100 个 li，就要占用 100 个内存空间，如果是 1000 个，10000 个呢，那只能说呵呵了，如果用事件委托，那么我们就可以只对它的父级（如果只有一个父级）这一个对象进行操作，这样我们就需要一个内存空间就够了，是不是省了很多，自然性能就会更好。

总而言之，就是可以提高性能

# 实现

```html
<ul id="ul1">
  <li>111</li>
  <li>222</li>
  <li>333</li>
  <li>444</li>
</ul>
```

```js
window.onload = function() {
  var oUl = document.getElementById("ul1");
  oUl.onclick = function(e) {
    alert(123);
  };
};
```

这样当我们点击 li 的时候也是会 alert 的，但是当我们点击 ul 的时候也会触发，如果想让他不触发，怎么办呢？

当我们点击 li 的时候，会产生一个点击事件，这个点击事件当被点下后就确定了，因为冒泡，就会先触发 li 的 click 事件（如果有的话），再触发 ul 的点击事件，但是传入这两个点击事假的 e 是不变的，从这个 e 中我们就可以判断到底是 ul 点击的还是 li 点击的，从而进行特定的操作

```js
window.onload = function() {
  var oUl = document.getElementById("ul1");
  oUl.onclick = function(ev) {
    var ev = ev || window.event;
    var target = ev.target || ev.srcElement;
    if (target.nodeName.toLowerCase() == "li") {
      alert(123);
      alert(target.innerHTML);
    }
  };
};
```

这样就只有点击 li 会触发 alert 了

# other

使用事件委托还有一个好处，正常情况下我们给 li 添加了点击事件后，如果又新增了 li，新的 li 是没有点击事件的，而使用事件委托因为是交给的父元素进行事件的触发，所以新增多少 li 都没有关系

给父容器绑定事件，判断 target 是哪个子元素，做相关操作 JQuery 的 delegate 和 on 就是这么干的
