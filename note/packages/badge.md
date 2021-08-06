# Badge 标记组件

> [Badge 标记 - Element文档](https://element.eleme.io/#/zh-CN/component/backtop)

## prop

```js
props: {
  // 显示的数字值
  value: [String, Number],
  // 最大值，超过会显示{max}+
  max: Number,
  // 显示小圆点而不是数字
  isDot: Boolean,
  // 隐藏badge
  hidden: Boolean,
  // 标识类型
  type: {
    type: String,
    validator(val) {
      return ["primary", "success", "warning", "info", "danger"].indexOf(val) > -1;
    }
  }
}
```

`value: [Number, String]`表示数字、字符串的联合类型

## 通过computed计算属性

通过computed计算badge显示的数字：

```js
content() {
  // 如果显示小圆点，直接return
  if (this.isDot) return;
  // value:数字，max:最大值
  const value = this.value;
  const max = this.max;
  // 是否超过最大值
  if (typeof value === "number" && typeof max === "number") {
    return max < value ? `${max}+` : value;
  }
  return value;
}
```

## 渲染文字

```vue
<template>
  <div class="el-badge">
    <slot></slot>
    <sup v-text="content"></sup>
    <!-- 其他逻辑 -->
  </div>
</template>
```

`content`就是上面计算属性计算出的值，使用`v-text`指令添加到元素上

### v-text 与 插值表达式{{}}

`<sup>{{ content }}</sup>`也可以更新元素的`textContent`

#### 闪烁

但是使用插值表达式，当网络不好时，页面可能会先显示`{{ content }}`，待数据加载后才显示对应的值，要解决这个问题可以配合指令`v-cloak`：`<span v-cloak>{{ content }}</span>`

使用`v-text="content"`指令就不会发生这个问题

#### 部分更新

> vue文档里说：
> 
> 更新元素的 textContent。如果要更新部分的 textContent，需要使用 {{ Mustache }} 插值。

所以当只需要部分替换内容时，只能使用插值表达式

#### 结论

`插值表达式`其实是一个占位符，要更新的值会替换掉占位符的位置

`v-text`是Vue对象传递过来的值，他会把节点的`textContent`全部替换掉