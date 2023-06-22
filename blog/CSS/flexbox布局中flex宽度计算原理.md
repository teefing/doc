---
  category: 专业知识
  draft: true
  tags:
    - CSS
  date: 2020-07-16
  title: flexbox布局中flex宽度计算原理
  vssue-title: flexbox布局中flex宽度计算原理
---

### 问题描述
最近遇到了一个小bug

> flex 项中子元素文本截断 text-overflow:ellipsis 失效

以下是代码
```html
<div class="parent">
  <div class="width-fixed"></div>
  <div class="text">
    <div class="text-inner">
      很长很长很长很长很长很长很长很长很长很长很长很长很长很长很长很长
    </div>
    </div>
</div>
```
```css
.parent {
  width: 200px;
  
  height: 100px;
  background-color: gray;
  display: flex;
  align-items: center;
}

.width-fixed {
  width: 30px;
  height: 30px;
  background-color: pink;
  flex: none;
}

.text {
  flex: 1;
}

.text-inner {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```
表现结果如下
![20200716123139](https://image.teefing.top/20200716123139.png)

### 解决方案
解决方案如下
```css
.text {
  flex: 1;
  min-width: 0;
}
```
在flex项中设置`min-width: 0`解决
![20200716123252](https://image.teefing.top/20200716123252.png)
ok 解决

ps: 也可以通过`overflow:hidden`来解决

### 原理剖析
#### flex项的宽度计算原理
根据 CSS 规范草案，一般情况下 `min-width` 属性默认值是 `0`，但是 Flexbox 容器中的 flex 项的 `min-width` 属性默认值是 `auto` ，这样能为 Flexbox 布局指明更合理的默认表现。
##### flex项宽度应用顺序
`flex-basis` 定义了在分配剩余空间之前 flex 项`默认的大小`。`flex-basis: auto` 意味着flex项会按照其`本来的大小`显示。

决定flex项宽度的因素有以下三点，优先级从高到低
1. flex-basis
   1. 受到max-width和min-width的控制，不会超过min-width和max-width的范围
2. width
3. 内容

> shrink-to-fit 的宽度 = min ( max (最小宽度, 可用宽度) , 首选宽度)

> https://css-tricks.com/flexbox-truncated-text/
> https://dev.w3.org/csswg/css-flexbox/#min-size-auto