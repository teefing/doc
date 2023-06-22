---
title: JavaScript事件冒泡与事件捕获
date: 2017-3-19
tags:
  - JavaScript
category: 专业知识
---

# 事件冒泡

事件冒泡是自下而上的去触发事件

```html
<div id="parent">
  <div id="child" class="child"></div>
</div>
```

```js
document.getElementById("parent").addEventListener("click", function(e) {
  alert("parent 事件被触发，" + this.id);
});
document.getElementById("child").addEventListener("click", function(e) {
  alert("child 事件被触发，" + this.id);
});
```

    child事件被触发，child
    parent事件被触发，parent

# 事件捕获

事件捕获指的是从 document 到触发事件的那个节点，即自上而下的去触发事件

现在改变第三个参数的值为 true

```js
document.getElementById("parent").addEventListener(
  "click",
  function(e) {
    alert("parent事件被触发，" + e.target.id);
  },
  true
);
document.getElementById("child").addEventListener(
  "click",
  function(e) {
    alert("child事件被触发，" + e.target.id);
  },
  true
);
```

parent 事件被触发，parent
child 事件被触发，child

# other

需要记忆的是 blur、focus、load、unload、media 相关事件是不会冒泡的

事件冒泡的应用： 事件委托
