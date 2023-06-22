---
title: css之background
date: 2017-5-2
tags:
  - CSS
category: 专业知识
---

# background

## background-color

设置区块背景的颜色

## background-position

设置背景图像的起始位置  
这个属性设置背景原图像（由 background-image 定义）的位置，背景图像如果要重复，将从这一点开始  
需要把 background-attachment 属性设置为 "fixed"，才能保证该属性在 Firefox 和 Opera 中正常工作  
语法：`background-position: 水平位置 垂直位置；`  
可以使用关键词 center top left bottom right **如果您仅规定了一个关键词，那么第二个值将是"center"。**  
水平和垂直位置的搭配通式为
`关键词x x%|xpos 关键词y y%|ypos`  
**如果您仅规定了一个值，另一个值将是 50%。**  
比如  
`left 20% bottom 10%`  
`left 10px bottom 10px`  
`left 20% bottom 10px`  
若不加`%`或`pos`默认`0% 0%`

## background-size

规定背景图像的尺寸  
**语法**  
`background-size: length|percentage|cover|contain;`  
**length:**设置背景图像的高度和宽度。
第一个值设置宽度，第二个值设置高度。
如果只设置一个值，则第二个值会被设置为 "auto"。  
**percentage:**以父元素的百分比来设置背景图像的宽度和高度。
第一个值设置宽度，第二个值设置高度。
如果只设置一个值，则第二个值会被设置为 "auto"。  
**cover:**把背景图像扩展至足够大，以使背景图像完全覆盖背景区域。
背景图像的某些部分也许无法显示在背景定位区域中。  
**contain:**把背景图像扩展至最大尺寸，以使其宽度和高度完全适应内容区域。  
`cover`**VS**`contain`  
相同点：图片都是等比例缩放  
不同点：cover 是铺满整个显示区域。如果显示比例和显示区域的比例相差很大某些部分会不显示；
contain 正好相反，他是按照某一边来覆盖显示区域的，会有白边

## background-repeat

设置是否及如何重复背景图像  
默认地，背景图像在水平和垂直方向上重复。  
可能的值：  
`repeat：`默认。背景图像将在垂直方向和水平方向重复。  
`repeat-x：`背景图像将在水平方向重复。  
`repeat-y ：`背景图像将在垂直方向重复。  
`no-repeat：`背景图像将仅显示一次。  
`inherit：`规定应该从父元素继承 background-repeat 属性的设置。

## background-origin

规定 background-position 属性相对于什么位置来定位  
如果背景图像的 background-attachment 属性为 "fixed"，则该属性没有效果  
**语法**  
`background-origin: padding-box|border-box|content-box;`
`padding-box:`背景图像相对于内边距框来定位。  
`border-box:`背景图像相对于边框盒来定位。  
`content-box:`背景图像相对于内容框来定位。

## background-clip

规定背景的绘制区域
**语法**
`background-clip: border-box|padding-box|content-box;`  
`border-box:`背景被裁剪到边框盒。  
`padding-box:`背景被裁剪到内边距框。   
`content-box:`背景被裁剪到内容框。

## background-attachment

设置背景图像是否固定或者随着页面的其余部分滚动  
`scroll:`默认值。背景图像会随着页面其余部分的滚动而移动。  
`local:`背景随元素内容滚动。  
`fixed:`当页面的其余部分滚动时，背景图像不会移动。  
`inherit:`规定应该从父元素继承 background-attachment 属性的设置。

## background-image

为元素设置背景图像  
`url('URL')`:指向图像的路径。  
`none:`默认值。不显示背景图像。  
`inherit:`规定应该从父元素继承 background-image 属性的设置。

##简写顺序
background：color img_url repeat attachment position / size
origin clip

##复合属性。
检索或设置对象的背景特性（背景色 <' background-color '> 不能设置多组）。  
一个元素可以设置多重背景图像。  
每组属性间使用逗号分隔。  
如果设置的多重背景图之间存在着交集（即存在着重叠关系），前面的背景图会覆盖在后面的背景图之上。  
示例：假设要在同一个元素上定义 3 个背景图像  
缩写方式：

    background:url(test1.jpg) no-repeat scroll 10px 20px/50px 60px content-box padding-box,
    	   url(test1.jpg) no-repeat scroll 10px 20px/70px 90px content-box padding-box,
    	   url(test1.jpg) no-repeat scroll 10px 20px/110px 130px content-box padding-box #aaa;

注意， <' background-color '> 只能设置一次，且由于写在前面的背景会叠在之后的背景之上，所以背景色通常都定义在最后一组上，避免背景色将图像盖住。  
拆分方式：

    background-image:url(test1.jpg),url(test2.jpg),url(test3.jpg);
    background-repeat:no-repeat,no-repeat,no-repeat;
    background-attachment:scroll,scroll,scroll;
    background-position:10px 20px,10px 20px,10px 20px;
    background-size:50px 60px,70px 90px,110px 130px;
    background-origin:content-box,content-box,content-box;
    background-clip:padding-box,padding-box,padding-box;
    background-color:#aaa;

如果定义了多个背景图片，而其他属性只有一个参数值，则表明所有背景图片的该属性都应用同一个参数值。据此可以对上面的例子进行缩写：  
缩写方式：

    background-image:url(test1.jpg),url(test2.jpg),url(test3.jpg);
    background-repeat:no-repeat;
    background-attachment:scroll;
    background-position:10px 20px;
    background-size:50px 60px,70px 90px,110px 130px;
    background-origin:content-box;
    background-clip:padding-box;
    background-color:#aaa;

如果定义了多个背景图片，而 <' background-origin '> 和 <' background-clip '> 设置了相同的值。可这样缩写：  
缩写方式：

    background:url(test1.jpg) no-repeat scroll 10px 20px/50px 60px padding-box,
    	   url(test1.jpg) no-repeat scroll 10px 20px/70px 90px padding-box,
    	   url(test1.jpg) no-repeat scroll 10px 20px/110px 130px padding-box #aaa;

这表示 <' background-origin '> 和 <' background-clip '> 都使用了 padding-box 参数值。
