---
title: js中的map函数
date: 2017-4-14
tags: 
 - JavaScript
category: 专业知识
---

# js 自带的 map() 方法

## 1. 方法概述 ##

     map() 方法返回一个由原数组中的每个元素调用一个指定方法后的返回值组成的新数组。

## 2. 例子 ##

### 2.1 在字符串中使用map###

  在一个 String  上使用 map 方法获取字符串中每个字符所对应的 ASCII 码组成的数组：

    var map = Array.prototype.map
    var a = map.call("Hello World", function(x) { return x.charCodeAt(0); })
    // a的值为[72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100]
### 2.2 易犯错误###

   通常情况下，map 方法中的 callback 函数只需要接受一个参数（很多时候，自定义的函数形参只有一个），就是正在被遍历的数组元素本身。

   但这并不意味着 map 只给 callback 传了一个参数（会传递3个参数）。这个思维惯性可能会让我们犯一个很容易犯的错误。

  

    
    // 下面的语句返回什么呢:
    ["1", "2", "3"].map(parseInt);
    // 你可能觉的会是[1, 2, 3]
    // 但实际的结果是 [1, NaN, NaN]
    
    // 通常使用parseInt时,只需要传递一个参数.但实际上,parseInt可以有两个参数.第二个参数是进制数.可以通过语句"alert(parseInt.length)===2"来验证.
    // map方法在调用callback函数时,会给它传递三个参数:当前正在遍历的元素, 元素索引, 原数组本身.
    // 第三个参数parseInt会忽视, 但第二个参数不会,也就是说,parseInt把传过来的索引值当成进制数来使用.从而返回了NaN.
    
    /*
    //应该使用如下的用户函数returnInt
    
    function returnInt(element){
      return parseInt(element,10);
    }
    
    ["1", "2", "3"].map(returnInt);
    // 返回[1,2,3]
