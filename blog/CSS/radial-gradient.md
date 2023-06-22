---
title: radial-gradient
date: 2017-4-3
tags: 
 - CSS
category: 专业知识
---
# radial-gradient
语法：  
`radial-gradient(类型 大小 at 圆心位置，颜色1 边界位置,颜色2 边界位置,颜色3 边界位置,...)`  
**类型**：可选，ellipse（椭圆）|circle 默认ellipse  
**大小**：可选， extent-keyword|circle-size|ellipse-size  
extent-keyword：closest-side|closest-corner|farthest-side|farthest-corner  
closest-side：
指定径向渐变的半径长度为从圆心到离圆心最近的边  
closest-corner：
指定径向渐变的半径长度为从圆心到离圆心最近的角  
farthest-side：
指定径向渐变的半径长度为从圆心到离圆心最远的边  
farthest-corner：
指定径向渐变的半径长度为从圆心到离圆心最远的角  
circle-size: 只接受长度单位如px，em  
ellipse-size: 水平大小 垂直大小，可接受长度单位或百分比  
圆心位置： 水平位置 垂直位置，都接收长度，百分比，left，top，center等关键词，也可混合使用，若只设定了一个位置，则水平位置为该位置，垂直位置默认为center  
颜色：rgba，hsla，关键词皆可  
边界位置：接受长度和百分比

举例：  
`radial-gradient(ellipse 200px 100px at left 100px top 50px,blue 30%,green 60%,yellow 80%);`
表示以距上边界50px,下边界100px的点为圆心，作水平半径200px，垂直半径100px的椭圆，其中在圆心到椭圆边界的路径上，0%-30%为蓝色，30%-50%为由蓝到绿的渐变色，50%-80%为由绿到黄的渐变色，80%-100%为黄色