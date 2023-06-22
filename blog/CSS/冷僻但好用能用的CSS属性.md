---
  category: 专业知识
  tags:
    - CSS
  date: 2020-06-15
  title: 冷僻但好用能用的CSS属性
  vssue-title: 冷僻但好用能用的CSS属性
---

## fill-available

`width:fill-available`表示撑满可用空间

举例来说，页面中一个`<div>`元素，该`<div>`元素的width表现就是fill-available自动填满剩余的空间

该属性高度也可用

使用场景：
1. 可以让元素的100%自动填充特性不仅仅在block水平元素上，也可以应用在其他元素，如span
2. 可以在非flex等高级布局下，轻松实现元素占据剩余空间的效果
3. 待发掘

兼容性：
![20200630201217](https://image.teefing.top/20200630201217.png)

*值得注意的是绝大部分浏览器只支持-webkit前缀的fill-available*

## fit-content

width:fit-content表示将元素宽度收缩为内容宽度

高度也可用

使用场景：
1. 可以赋予block的元素收缩的效果
2. 配合margin:auto轻松实现向内自适应同时的居中效果

兼容性：
![20200630202006](https://image.teefing.top/20200630202006.png)

*使用时最好也加上-webkit前缀的fit-content*

## min-content

width:min-content表示采用内部元素最小宽度值最大的那个元素的宽度作为最终容器的宽度

最小宽度值：
替换元素，例如图片的最小宽度值就是图片呈现的宽度，对于文本元素，如果全部是中文，则最小宽度值就是一个中文的宽度值；如果包含英文，因为默认英文单词不换行，所以，最小宽度可能就是里面最长的英文单词的宽度

兼容性：
![20200630202556](https://image.teefing.top/20200630202556.png)

## max-content

width:max-content表示采用内部元素宽度值最大的那个元素的宽度作为最终容器的宽度。如果出现文本，则相当于文本不换行

兼容性：
![20200630202625](https://image.teefing.top/20200630202625.png)


## max

```css
/* property: max(expression [, expression]) */
width: max(10vw, 4em, 80px);
```
兼容性：
![20200630203754](https://image.teefing.top/20200630203754.png)

不容乐观，混个眼熟就算了

## min

```css
/* property: min(expression [, expression]) */
width: min(1vw, 4em, 80px);
```

兼容性：
![20200630203921](https://image.teefing.top/20200630203921.png)

同样不容乐观，看看算了

>>> 以上所有属性在IE中都无效