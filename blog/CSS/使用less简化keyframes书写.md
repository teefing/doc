---
  category: 专业知识
  tags:
    - Less
  date: 2021-03-10
  title: 使用less简化keyframes计算
  vssue-title: 使用less简化keyframes计算
---

# 使用less简化keyframes计算

## 场景
今天需要做一个稍微复杂的动画，大致流程是 放大 -> 缩小 -> shaking一遍 -> 停顿一会 -> shaking一遍 -> 放大。 
上面的流程是从设计给的动效视频里看出来的，除此以外无任何数据。  
这个需求点的难点不在于实现，而在于调试动效。  
我们可以根据肉眼大致观察出每个动效的时间点，手动计算出百分比后进行代码的编写。  
如果动画的总时长确定倒也还容易处理，在每个动效的时间点发生变化时计算对应的百分比即可，可是如果我们在调试过程中发现总时长需要调整，那么所有动效的时间点就都要重新计算，浪费大量精力。   
因此就想能不能通过定义关键帧的时间点，使用less自动帮我们计算百分比。  
在google好一会后，也没找到相关的文章，因此只能自己造轮子了。  
平时在工作中对于less的使用并不是非常深入，最多的也就是父级选择器和变量的使用。这次为了实现这个功能，我们需要用到less的变量、变量计算、字符串插值、自定义函数、内置函数、mixin、传递规则集给mixin这些知识点  
下面就是实际代码

## 代码
```less
@boxOpenTotal: 2.5;
@scaleEndPoint: 0.5/@boxOpenTotal;
@shake1EndPoint: 1/ @boxOpenTotal;
@shake2StartEndPoint: 1.5/@boxOpenTotal;
@shake2EndPoint: 2/@boxOpenTotal;
@openEndPoint: 2.5/@boxOpenTotal;

.point(@input, @ruleset){
  @t: percentage(@input);
  @{t} {
    @ruleset()
  }
}

@keyframes box-open {
  0 {
    transform: scale(1);
  }

  .point(@scaleEndPoint/2, {
    transform: scale(0.8);
  });

  .point(@scaleEndPoint, {
    transform: scale(1) rotate(0);
  });

  .point(@scaleEndPoint + (@shake1EndPoint - @scaleEndPoint)/4, {
    transform: rotate(-10deg);
  });

  .point(@scaleEndPoint + (@shake1EndPoint - @scaleEndPoint)/4*3, {
    transform: rotate(20deg);
  });

  .point(@shake1EndPoint, {
    transform: rotate(0);
  });

  .point(@shake2StartEndPoint, {
    transform: rotate(0);
  });

  .point(@shake2StartEndPoint + (@shake2EndPoint - @shake2StartEndPoint)/4, {
    transform: rotate(-10deg);
  });

  .point(@shake2StartEndPoint + (@shake2EndPoint - @shake2StartEndPoint)/4*3, {
    transform: rotate(20deg);
  });

  .point(@shake2EndPoint, {
    transform: rotate(0);
  });

  .point(@openEndPoint, {
    transform: scale(1.3);
  })
}
```

## 编译结果
可以上 [lesstester.com](https://lesstester.com/) 查看编译结果
```css
@keyframes box-open {
  0 {
    transform: scale(1);
  }
  10% {
    transform: scale(0.8);
  }
  20% {
    transform: scale(1) rotate(0);
  }
  25% {
    transform: rotate(-10deg);
  }
  35% {
    transform: rotate(20deg);
  }
  40% {
    transform: rotate(0);
  }
  60% {
    transform: rotate(0);
  }
  65% {
    transform: rotate(-10deg);
  }
  75% {
    transform: rotate(20deg);
  }
  80% {
    transform: rotate(0);
  }
  100% {
    transform: scale(1.3);
  }
}

```