---
layout: page
title: Animation
permalink: /animation/
---
## 动画实现
### CSS动画过度的实现
我们在使用css动画时，一般会设置一个开始值，一个结束值，一个时间段和一个缓动的[贝塞尔曲线](https://baike.baidu.com/item/%E8%B4%9D%E5%A1%9E%E5%B0%94%E6%9B%B2%E7%BA%BF/1091769?fr=aladdin)

+ ease: cubic-bezier(0.25, 0.1, 0.25, 1.0)
+ linear: cubic-bezier(0.0, 0.0, 1.0, 1.0)
+ ease-in: cubic-bezier(0.42, 0, 1.0, 1.0)
+ ease-out: cubic-bezier(0, 0, 0.58, 1.0)
+ ease-in-out: cubic-bezier(0.42, 0, 0.58, 1.0)

那么在css中贝塞尔曲线代表的含义是什么呢？

![bezier](http://static.chryseis.cn/bezier.jpg)

如图在css中bezier是一个关于时间和进度的函数
横坐标范围(0,1)，纵坐标范围(0,1)

即 ```p=kt```,由此可知，若已知贝塞尔曲线，便可知函数中每个时刻所对应的进度

此时我们借助bezier曲线函数便可以通过代码来表示这个值变化，即为

```javascript
const BezierEasing = require('bezier-easing')

const easing = BezierEasing(0, 0, 1, 0.5)

let duration = 2000, deltaTime = 20, start = 100, end = 1000

let current
let startTime = Date.now()
let timer = setInterval(() => {
    let deltaTime = Math.min(Date.now() - startTime, duration)
    current = (end - start) * easing(deltaTime / duration) + start
    deltaTime === duration && clearInterval(timer)
    console.log('current', current)
}, deltaTime)
```

结果为
![result](http://static.chryseis.cn/result.jpg)

即为一个时间轴动画

### Animation中keyFrames
如果我们定义一个keyFrames
``` css
@keyframes scale {
  0% {
    transform: scale(.5);
  }
  20% {
    transform: scale(1);
  }
  40% {
    transform: scale(1.5);
  }
  60% {
    transform: scale(2);
  }
  80% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```
然后在css中使用这个样式
```
animation: scale 5s cubic-bezier(0.060, 0.895, 0.060, 1.650) infinite;
```
会发现每个阶段都是重新执行一次bezier，也就是说0-20%，20%-40%，40%-60%，60%-80%，80%-100%改变率都是先快后慢
![animation](http://static.chryseis.cn/animation.gif)

## 未完待续