---
title: Flex
category: 专业知识
date: 2019-05-19 15:51:03
tags: 
  - CSS
---
# Flex 布局

 Flex（Flexible Box）布局 称为 "弹性布局"，可以为网页的布局提供最大的灵活性，取代了往常的 浮动（float） 布局，并且任何一个容器都可以设置 Flex 布局。
 **注：设置 Flex 布局后，子元素的 Float 布局将失效**

## Flex 中的四大概念

> **容器：** 如果给一个标签添加`display:flex;`，那么这个标签就是一个容器
> **项目：** 在容器中的直接子元素叫项目（一定是 _**直接**_ 子元素）
> **主轴：** 项目的默认排序方向就是主轴（默认横向排列，一个容器可以有多根主轴）
> **交叉轴：** 和主轴垂直的那个轴，就是交叉轴
> 
> ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_1.png)

## 容器的属性

**Flex-direction | Flex-wrap | Flex-flow | justify-content | align-items | align-content**

1.  Flex-direction（属性决定主轴的方向）

> **`flex-direction：row | row-reverse | column | column-reverse;`**
> 
> > *   row（默认值）：主轴为水平方向，起点在左端
> > *   row-reverse：主轴为水平方向，起点在右端
> > *   column：主轴为垂直方向，起点在上端
> > *   column-reverse：主轴为垂直方向，起点在下端 ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_2.png)

1.  Flex-wrap(属性决定项目排不下时如何换行)

> **`flex-wrap：nowrap | wrap | wrap-reverse;`**
> 
> > *   norwrap（默认）：不换行
> > *   wrap：换行，第一行在上方
> > *   wrap-reverse：换行，第一行在下方
> >     ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_3.png)

3.Flex-flow（属性是 flex-direction 和 flex-wrap 的简写形式）

> *   **`flex-flow: flex-direction || flex-wrap;`**

4.justify-content(处理项目外的 富余空间)

> *   **`justify-content：flex-start | flex-end | center | space-between | space-around;`**
> 
> > *   flex-start；（默认值），项目左对齐
> > *   flex-end：项目右对齐
> > *   center： 项目居中
> > *   space-between：项目两端对齐，项目之间的间隔都相等
> > *   space-around：每个项目两侧的间隔相等
> >     ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_4.png)

5.align-items(属性决定项目在交叉轴上如何对齐)

> *   **`align-items：stretch | flex-start | flex-end | center | baseline;`**
> 
> > *   stretch（默认值）：如果项目未设置高度或设为 auto，将占满整个容器的高度
> > *   flex-start：交叉轴的起点对齐
> > *   flex-end：交叉轴的终点对齐
> > *   center：交叉轴的中点对齐
> > *   baseline：项目的第一行文字的基线对齐
> >     ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_5.png)

6.align-content（属性决定了多根主轴的对齐方式）

> *   **`align-content：stretch | flex-start | flex-end | center | space-between | space-around;`**
> 
> > *   stretch（默认值）：轴线占满整个交叉轴
> > *   flex-start：与交叉轴的起点对齐
> > *   flex-end：与交叉轴的终点对齐
> > *   center：与交叉轴的中点对齐
> > *   space-between：与交叉轴两端对齐，轴线之间的间隔平均分
> > *   space-around：每根轴线两侧的间隔都相等 ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_6.png)

## 项目的属性

**order | flex-grow |flex-shrink | flex-basis | flex | align-self**

> 1.  order（属性定义项目的排列顺序。数值越小，排列越靠前，默认为 0）
> 
> > `order: <integer>;`
> > 
> > ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_7.png)

> 1.  flex-grow（属性定义项目的放大比例，默认为 0，即如果存在剩余空间，也不放大）
> 
> > `flex-grow：<number>;`
> > 
> > ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_8.png)

> 1.  flex-shrink（属性定义了项目的缩小比例，默认为 1，即如果空间不足，该项目将缩小）
> 
> > `flex-shrink: <number>;`
> > 
> > ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_9.png)

> 1.  flex-basis(属性定义了在分配多余空间之前，项目占据的主轴空间。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为 auto，即项目的本来大小, 也可以设置 `xx px`, 项目将占据固定空间)
> 
> > `flex-basis: <length> | auto;`
> > 
> > ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_10.png)

> 1.  flex（属性是 flex-grow, flex-shrink 和 flex-basis 的简写，默认值为 0 1 auto。后两个属性可选）
> 
> > `flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ];`

> 1.  align-self（属性允许单个项目有与其他项目不一样的对齐方式，可覆盖 align-items 属性。默认值为 auto，表示继承父元素的 align-items 属性，如果没有父元素，则等同于 stretch）
> 
> > `align-self: auto | stretch | flex-start | flex-end | center | baseline;`
> > 
> > ![](https://gitee.com/git_fanjunyang/juejin/raw/master/images/Flex_%E5%B8%83%E5%B1%80%E6%95%99%E7%A8%8B_11.png)