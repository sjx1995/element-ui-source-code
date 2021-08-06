# Button 按钮组件

> [Button 按钮 - Element文档](https://element.eleme.io/#/zh-CN/component/button)

## prop

```js
props: {
  // 按钮类型
  type: { type: String, default: "default" },
  // 按钮大小
  size: String,
  // 按钮图标
  icon: { type: String, default: "" },
  // 原生type属性
  nativeType: { type: String, default: "button" },
  // 正在加载
  loading: Boolean,
  // 禁用
  disabled: Boolean,
  // 幽灵按钮
  plain: Boolean,
  // 默认聚焦
  autofocus: Boolean,
  // 圆角按钮
  round: Boolean,
  circle: Boolean
}
```

## 通过computed计算属性

```js
computed: {
  // 计算得到表单中按钮大小，供下一步使用
  _elFormItemSize() {
    return (this.elFormItem || {}).elFormItemSize;
  },
  // 计算按钮大小
  buttonSize() {
    return this.size || this._elFormItemSize || (this.$ELEMENT || {}).size;
  },
  // 计算按钮是否禁用
  buttonDisabled() {
    return this.disabled || (this.elForm || {}).disabled;
  }
}
```

这里的`this.elFormItem`和`this.elForm`都是都过`inject`注入的`property`

### $ELEMENT

这里有一个在Vue原型链上的属性`$ELEMENT`，是一个在组件被引入时定义的全局样式配置：

```js
// 引入element-ui时设置全局大小
import Element from "element-ui";
Vue.use(Element, { size: "small", zIndex: 3000 });
```

传入的配置在组件install时被挂载到原型上：

```js
// /src/index.js
// ...引入各个组件
const install = function(Vue, opt = {}) {
  Vue.prototype.$ELEMENT = {
    size: opt.size || "",
    zIndex: opt.zIndex || 2000
  }
  // ...
}
```

## 逻辑

### 点击按钮抛出click事件

```js
handleClick(evt) {
  this.$emit("click", evt);
}
```

### 自动聚焦

直接调用<button>元素上原生的`autofocus`属性：`<button :autofoucs="autofocus"></button>`

> [button#autofoucs - MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/button#attr-autofocus)

### nativeType

组件attribute中`nativeType`可以有三个值：`submit`、`reset`、`button`，这里对应<button>元素的原生属性`type`：`<button :type="nativeType"></button>`

button的type属性可以拥有四个值：

- `submit`：按钮将表单数据提交给服务器，如果未指定属性则其为默认值
- `reset`：重置所有组件为初始值
- `button`：没有默认行为
- `menu`：打开一个由指定<menu>元素进行定义的弹出菜单

> [button#type - MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/button#attr-type)

### 按钮文字

通过`$slots.default`进行渲染

```vue
<template>
  <button class="el-button">
    <span v-if="$slots.default">
      <slot></slot>
    </span>
  <button>
</template>
```

### 按钮组

按钮组通过<slot>包裹一组<el-button>，其余通过样式实现：

```vue
// button-group.vue
<template>
  <div class="el-button-group">
    <slot></slot>
  </div>
</template>
```