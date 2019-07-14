---
layout: page
title: Animation
---
## 动画实现
### CSS动画过度的实现
我们在使用css动画时，一般会设置一个开始值，一个结束值，一个时间段和一个缓动的[贝塞尔曲线](https://baike.baidu.com/item/%E8%B4%9D%E5%A1%9E%E5%B0%94%E6%9B%B2%E7%BA%BF/1091769?fr=aladdin)

```css
	.box{
		width:100px;a
		&.animation{
			width:500px;
			transition: width 5s easein
		}
	}

```

+ ease: cubic-bezier(0.25, 0.1, 0.25, 1.0)
+ linear: cubic-bezier(0.0, 0.0, 1.0, 1.0)
+ ease-in: cubic-bezier(0.42, 0, 1.0, 1.0)
+ ease-out: cubic-bezier(0, 0, 0.58, 1.0)
+ ease-in-out: cubic-bezier(0.42, 0, 0.58, 1.0)

那么在css中贝塞尔曲线代表的含义是什么呢？

![bezier](http://static.chryseis.cn/bezier.jpg)

如图在css中bezier是一个关于时间和进度的函数
横坐标范围(0,1)，纵坐标范围(0,1)

即 ```f(p)=t```,由此可知，若已知贝塞尔曲线，便可知函数中每个时刻所对应的进度

此时我们借助bezier曲线函数便可以通过代码来表示这个值变化，即为

```javascript
const BezierEasing = require('bezier-easing')

const easing = BezierEasing(0, 0, 1, 0.5)

let duration = 2000, deltaTime = 1000 / 60, start = 100, end = 1000

let current
let startTime = Date.now()
let timer = setInterval(() => {
    let currDuration = Math.min(Date.now() - startTime, duration)
    current = (end - start) * easing(currDuration / duration) + start
    currDuration === duration && clearInterval(timer)
    console.log('current', current)
}, deltaTime)
```

结果为
![result](http://static.chryseis.cn/result.jpg)

即为一个时间轴动画,在动画开始前便已知动画过程

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

### 例子

```Jsx
import React from 'react'
import BezierEasing from 'bezier-easing'

const ease = BezierEasing(0.25, 0.1, 0.25, 1.0)
const easein = BezierEasing(0.42, 0, 1.0, 1.0)
const easeout = BezierEasing(0, 0, 0.58, 1.0)

let duration = 5000

const style = {
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    height: 100,
    backgroundColor: 'red',
    marginBottom: 10,
    fontSize: 20,
}

class App extends React.Component {
    state = {
        width1: 0,
        width2: 0,
        width3: 0
    }

    componentDidMount() {
        this.nextTick(Date.now(), 'width1', 0, 500, duration, ease)
        this.nextTick(Date.now(), 'width2', 0, 500, duration, easein)
        this.nextTick(Date.now(), 'width3', 0, 500, duration, easeout)
    }

    nextTick = (startTime, property, start, end, duration, fn) => {
        let currDuration = Math.min(Date.now() - startTime, duration)
        this.setState({
            [property]: (end - start) * fn(currDuration / duration) + start
        })
        currDuration !== duration && window.requestAnimationFrame(this.nextTick.bind(this, startTime, property, start, end, duration, fn))
    }

    render() {
        const { width1, width2, width3 } = this.state
        return <div>
            <div className="box" style={{ width: width1, ...style }}>Ease
            </div>
            <div className="box"
                 style={{ width: width2, ...style }}>Easein
            </div>
            <div className="box" style={{ width: width3, ...style }}>Easeout</div>
        </div>
    }
}
```

效果图
![example](http://static.chryseis.cn/example.gif)

[在线效果](https://codesandbox.io/s/wonderful-https-y2wmv)

由于采用javascript模拟css的动画实现，故会发现动画的时间可能并不是那么精确，但可以发现基本上是同时到达的


### Javascript 动画

javascript动画的实现分为两种

+ 基于时间
+ 基于变化率

#### 时间动画

时间动画便是上文中用js模拟css的动画的原理，
整个动画的在执行之初便已设定完成，我们只需改变bezier函数便可以获得各式各样的动画效果。在此不多做叙述。

#### 变化率的动画

关于变化率的动画，对比时间动画，我们只关心**开始量** ，**结束量**和每个时刻的**变化量**，不关心**时间**

例如 一辆汽车启驶速度为10，我们要把速度变为100，速度改变率为10，
那么上一刻速度和下一刻的速度关系即为 ```v1=at+v0```，t是我们做动画的间隔时间，由此，我们做动画只需要一直累加，直到速度到100时，动画停止便可以。

用代码的表示

```javascript
let start = 10, end = 100, a = 10, deltaTime = 1000 / 60
let current = start
let timer = setInterval(() => {
    current = current + a
    current === end && clearInterval(timer)
    console.log(current)
}, deltaTime)

```
结果为![example](http://static.chryseis.cn/js_example.jpg)

到这里js动画的原理已经完了，各式各样的动画无非是改变变化率，产生不同的效果。但是我们会发现一个问题，如果每次方法的**执行时间**大于动画的**间隔时间**怎么办？

**说明： 执行时间为浏览器执行下面代码
	() => {
	    current = current + a
	    current === end && clearInterval(timer)
	    console.log(current)
	   }的时间； 间隔时间=deltaTime;**

由于**执行时间**取决于浏览器本身，而**间隔时间**一般我们都会默认为1000/60=16.7ms,1秒60桢，当**执行时间**大于**间隔时间**时，实际间隔时间即为**执行时间**，动画不可避免的就会出现卡顿，也就是我们经常说的动画不流畅，外加javascript的单线程原因，我们执行动画时，不能避免别的逻辑的执行，因此对于这个问题的解决，我们需要做一些特殊处理