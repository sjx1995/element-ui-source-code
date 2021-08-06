# Backtop 回到顶部组件

> [Backtop 回到顶部 - Element文档](https://element.eleme.io/#/zh-CN/component/backtop)

## prop

```js
props: {
  // 出现返回顶部按钮高度
  visibilityHeight: { type: Number, default: 200 },
  // 需要滚动的元素：css选择器
  target: [String],
  // 距离页面右边距
  right: { type: Number, default: 40 },
  // 距离页面下边距
  bottom: { type: Number, default: 40 }
}
```

## data

```js
data() {
  return {
    el: null, // 需要滚动到顶的元素
    container: null, // 监听滚动事件的元素
    visible: false // 是否显示滚动到顶的按钮
  };
}
```

## 使用computed计算属性

```js
computed: {
  // 设置定位位置
  styleBottom() {
    return `${this.bottom}px`;
  },
  styleRight() {
    return `${this.right}px`;
  }
}
```

## 逻辑

在mounted钩子中定义了以下方法：

```js
mounted() {
		// 初始化：确定需要滚动的元素
  this.init();
  // 节流检查是否显示滚动到顶按钮
  this.throttledScrollHandler = throttle(300, this.onScroll);
  // 监听滚动事件并触发onScroll()方法
  this.container.addEventListener("scroll", this.throttledScrollHandler);
}
```

这里使用了npm模块：[throttle-debounce](https://www.npmjs.com/package/throttle-debounce)

### 初始化：确定需要滚动的元素

`init()`方法确定需要滚动到顶的元素：

```js
init() {
  // document.documentElement === <html>元素
  // 需要滚动的元素默认为<html>元素
  this.container = document;
  this.el = document.documentElement;
  // 如果传入了target，则将需要滚动的元素保存到el
  if (this.target) {
    this.el = document.querySelector(this.target);
    if (!this.el) {
      throw new Error(`target is not existed: ${this.target}`);
    }
    this.container = this.el;
  }
}
```

#### document、document.documentElement 与 document.body 的区别

- document 是整个文档
- document.documentElement 是文档根节点，对于网页来说就是<html>标签
- document.body 是DOM结构中的<body>节点

```js
document.querySelector("html") === document.documentElement; // true
Object.prototype.toString.call(document); // "[object HTMLDocument]"
Object.prototype.toString.call(document.documentElement); // "[object HTMLHtmlElement]"
Object.prototype.toString.call(document.body); // "[object HTMLBodyElement]"
```

- 如果文档开始声明了DTD，那么使用`document.documentElement.scrollTop`来获取垂直滚动像素数
- 如果没有声明DTD，那么使用`document.body.scrollTop`来获取垂直滚动像素数

### 检查是否显示滚动到顶按钮

```js
onScroll() {
  // 获取需要滚动到顶的元素的垂直滚动的像素数
  const scrollTop = this.el.scrollTop;
  // 判断时候需要显示按钮
  this.visible = scrollTop >= this.visibilityHeight;
}
```

### 点击滚动到顶按钮

```js
handleClick(e) {
  // 触发滚动到顶方法
  this.scrollToTop();
  // 抛出`click`事件供调用者监听，外部可以拿到mouseEvent
  this.$emit("click", e);
}
```

### 滚动动画

在支持`window.requestAnimationFrame()`的浏览器中使用`requestAnimationFrame()`方法，否则使用`setTimeout()`每16ms（保证60fps）执行一次

```js
scrollToTop() {
  // 定义滚动动画函数
  const cubic = value => Math.pow(value, 3);
  const easeInOutCubic = value => (value < 0.5 ? cubic(value * 2) / 2 : 1 - cubic((1 - value) * 2) / 2);
  // 需要滚动的元素
  const el = this.el;
  // 开始滚动时间
  const beginTime = Date.now();
  // 开始滚动的位置
  const beginValue = el.scrollTop;
  // 检查是否支持requestAnimationFrame，不支持的话使用setTimeout
  const rAF = window.requestAnimationFrame || (func => setTimeout(func, 16));
  // 滚动动画
  const frameFunc = () => {
    const progress = (Date.now() - beginTime) / 500;
    // 如果滚动过程小于500ms，则继续执行滚动动画
    // 否则滚动到顶结束动画
    if (progress < 1) {
      el.scrollTop = beginValue * (1 - easeInOutCubic(progress));
      rAF(frameFunc);
    } else {
      el.scrollTop = 0;
    }
  };
  rAF(frameFunc);
}
```

#### window.requestAnimationRequest()

```js
window.requestAnimationFrame(callback);
```

`requestAnimationFrame()`告诉浏览器你希望执行的动画，浏览器会在下次重绘的时候调用绘制这个动画，方法接受一个参数作为执行动画的回调

##### setTimeout/setInterval的缺点

- 当浏览器最小化时，JavaScript计时器仍会运行，但用户看不到动画，造成资源浪费
- 如果回调执行时间大于计时器间隔时间，那么下一次的回调就会“排队”执行，最后浏览器也许会奔溃
- 为了让动画更平滑，有时我们会选择比刷新率略高的频率执行计时器，这让在屏幕刷新要绘制新画面时，一些帧已经画过而被抛弃，造成不必要的资源浪费

##### requestAnimationFrame()的解决办法

- `requestAnimationFrame()`知道浏览器的状态，他只绘制用户可见的动画
- 当浏览器空闲的时候，才绘制一帧，没有等待中的帧，所以不会造成阻塞、排队的情况，也没有多余的帧被浪费，动画也更平滑

##### requestAnimationFrame()需要注意的问题

- 如果你希望下一帧继续执行这个动画，需要手动再次调用`window.requestAnimationFrame(callback)`
- 如果需要取消动画，使用`cancelAnimationFrame(id)`
- 如果同时执行两个需要同步的动画，但是其中一个在前三秒内不可见，那么三秒后两个动画将不会同步

### 取消监听

别忘记在vue实例销毁的时候取消监听`scroll`，因为对于单页面应用来说，注册监听事件的`container`不一定会被销毁，但是离开组件之后却不需要继续监听`scroll`事件

```js
beforeDestroy() {
  this.container.removeEventListener("scroll", this.throttledScrollHandler);
}
```