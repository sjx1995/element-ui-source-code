# Avatar 头像组件

> [Avatar 头像 - Element文档](https://element.eleme.io/#/zh-CN/component/avatar)

## prop

```js
props: {
  // 尺寸
  size: {
    type: [Number, String],
    validator(val) {
      if (typeof val === "string") {
        return ["large", "medium", "small"].includes(val);
      }
      return typeof val === "number";
    }
  },
  // 形状
  shape: {
    type: String,
    default: "circle",
    validator(val) {
      return ["circle", "square"].includes(val);
    }
  },
  // icon名
  icon: String,
  // 头像图片地址
  src: String,
  // 图片替换文字
  alt: String,
  // 一组用户代理使用的图像
  srcSet: String,
  // 头像加载失败的回调
  error: Function,
  // 图片填充容器的方式
  fit: { type: String, default: "cover" }
}
```

`size`和`shape`的验证函数在`Alert`中已经说过，这里使用`includes()`方法来判断时候包含给定文字

`fit`作为css样式`object-fit`的值来控制图片如何适应到其使用的高度和宽度确定的框：

```
object-fit: fill | contain | cover | none | scale-down
fill: 拉伸填满容器
contain: 保持比例，内容缩放
cover: 保持比例，内容被裁切
none: 保持原有长宽
scale-down: 保持比例，具体样式为contain或者cover，取决于谁更小
```

> [object-fit - MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/object-fit)

## 通过computed计算属性

`Avatar`组件渲染使用了jsx，这里使用computed计算得到组件的`class`

```js
computed: {
  avatarClass() {
    // 从上下文中获取到size、icon、shape
    const { size, icon, shape } = this;
    // 初始化classList
    let classList = ["el-avatar"];
    // 当size为字符串时，设置size属性
    if (size && typeof size === "string") {
      classList.push(`el-avatar--${size}`);
    }
    // 设置icon及形状
    if (icon) {
      classList.push("el-avatar--icon");
    }
    if (shape) {
      classList.push(`el-avatar--${shape}`);
    }
    // 拼接字符串
    return classList.join(" ");
  }
}
```

## 渲染

### 渲染组件

```js
render() {
  // 从上下文获取变量
  const { avatarClass, size } = this;
  // 如果size是数字，则使用数字确定avatar大小
  const sizeStyle =
    typeof size === "number"
      ? {
          height: `${size}px`,
          width: `${size}px`,
          lineHeight: `${size}px`
        }
      : {};
  // 设置class和style并渲染
  return (
    <span class={avatarClass} style={sizeStyle}>
      {this.renderAvatar()}
    </span>
  );
}
```

### 渲染avatar函数

```js
renderAvatar() {
  // 从上下文获取变量
  const { icon, src, alt, isImageExist, srcSet, fit } = this;
  // 如果图片存在，则返回图片
  if (isImageExist && src) {
    return <img src={src} onError={this.handleError} alt={alt} srcSet={srcSet} style={{ "object-fit": fit }} />;
  }
  // 如果icon存在则渲染icon
  if (icon) {
    return <i class={icon} />;
  }
  return this.$slots.default;
}
// 错误函数
handleError() {
  const { error } = this;
  const errorFlag = error ? error() : undefined;
  if (errorFlag !== false) {
    this.isImageExist = false;
  }
}
```

#### srcset

这里有一个srcset属性，是img标签的属性：用来逗号分隔一个或多个字符串列表，表明一系列用户代理使用的可能的图像

简单说就是根据用户的设备不同加载不同图片，比如：

```html
<img src="small.jpg " srcset="big.jpg 1440w, middle.jpg 800w, small.jpg 1x" />
```

`w`表示宽度、`x`表示像素密度：

- 如果宽度达到1440px，则加载big.jpg
- 如果宽度达到800w，则加载middle.jpg
- 否则加载small.jpg

> [img - MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/img#attr-srcset)