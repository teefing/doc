---
title: css之border-image
date: 2017-4-3
tags: 
	- CSS
category: 专业知识
---

# border-image
## border-image-source 用在边框的图片的路径。
	border-image-source: url(border.png);
## border-image-slice	图片边框向内偏移。

设置切割线的位置，最多可设4个值，长度百分比均可，同margin，   
fill为可选属性值，假如指定，那么中间第九块不是透明块，假如不指定，那么为透明图片处理
## border-image-width	图片边框的宽度。
设置边框的宽度，最多可设4个值，长度百分比均可，同margin，还可设auto， 如果规定该属性，则宽度为对应的图像切片的固有宽度。  

## border-image-outset	边框图像区域超出边框的量。	
设置边框偏移量，最多可设4个值，长度百分比均可，同margin。   
## border-image-repeat	图像边框是否应平铺(repeated)、铺满(rounded)或拉伸(stretched)。