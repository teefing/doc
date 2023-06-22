---
title: css之linear-gradient
date: 2017-4-4
tags: 
 - CSS
category: 专业知识
---
# linear-gradient
语法: -浏览器前缀-linear-gradient(方向(可选),颜色结点1,颜色结点2,颜色结点3,...)  
* 浏览器前缀 -webkit -o -moz 不加浏览器前缀则
需在方向前加to 如to top，注意to top等同与bottom表示从下往上,当然，可以使用角度  
* 方向可以为角度，或是to top,to right,to bottom,to left 中的一个，默认从上往下=to bottom=180deg
top即方向由上往下按顺序依次画出颜色结点的颜色，left由左往右方向，right和bottom同理
若方向为角度，0deg等同于to top，随着角度增加，方向向量顺时针旋转，90deg=to right，180deg=to bottom，270deg=to left，并且角度可为负值
使用角度的优点在于灵活，角度可任意  
* 颜色结点可设置支持俗名，#，rgba等等，并且可额外附加百分比（注意颜色与百分比之间要加一个空格），表示该颜色所在的位置，中间部分为渐变色  
比如`linear-gradient(red 20%，blue 80%);`
则从上往下前20%区域为红色，20%-80%区域为红色到蓝色的渐变色，80%-100%区域为蓝色  
当然`linear-gradient(red 20%，green 70%，blue 80%);`表示0-20%区域为红色，20%-70%区域为红色到绿色的渐变色，70%-80%区域为绿色到蓝色的渐变色，80%-100%区域为蓝色  
`linear-gradient(red 20%，green，blue 80%);`如果中间的green不标注百分比，则默认百分比为（20%+80%）/2=50%，表示0-20%区域为红色，20%-50%区域为红色到绿色的渐变色，50%-80%区域为绿色到蓝色的渐变色，80%-100%区域为蓝色 
小应用：  
产生条纹状背景  
非渐变：  
		background: -webkit-linear-gradient(top,#fb3 50%,#58a 50%);
		background-size: 100% 30%;  
渐变：  
		background: -webkit-linear-gradient(top,white 0%,#fb3 30%,#58a 60%,white 100%);
		background-size: 100% 30%;  
