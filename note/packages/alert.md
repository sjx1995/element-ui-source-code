# Alert 警告组件

> [Alert 警告 - Element文档](https://element.eleme.io/#/zh-CN/component/alert)

## prop

prop是组件接受外部数据的重要途经，组件的`Attributes`便是通过props传递进来的，从文档和源码我们可以看到：

```js
props: {
  // 标题
  title: { type: String, default: "" },
  // 描数
  description: { type: String, default: "" },
  // 类型
  type: { type: String, default: "info" },
  //可关闭
  closable: { type: Boolean, default: true },
  // 关闭文字
  closeText: { type: String, default: "" },
  showIcon: Boolean, // 展示图标
  center: Boolean, // 文字居中
  // 颜色主题
  effect: {
    type: String,
    default: "light",
    // props验证
    validator: function (value) {
      return ["light", "dark"].indexOf(value) !== -1;
    }
  }
}
```

这里的`effect`使用了验证函数`validator`，参数是prop的值，返回一个布尔类型，这里要求`effect`的值必须是`light`或者`dark`中的一个，如果验证失败会产生一个控制台警告（开发环境）

需要注意的是，`prop`验证的时机是在组件实例创建**之前**，所以实例的`property`（比如`data`、`computed`）在`default`或`validator`中无效

## 显示与关闭

`alert`组件默认在`data`中声明一个值为`true`的变量`visible`，用来控制组建的显示与否，关闭时将`visible`设置为`false`，并抛出`close`时间供调用者监听

```vue
<template>
  <div class="el-alert" v-show="visible">
    <!-- 其他逻辑 --> 
    <i class="el-alert--closebtn" @click="close()">关闭</i>
  </div>
</template>

<script>
export default {
  data() {
    return {
      visible: true
    };
  },

  methods: {
    close() {
      this.visible = false;
      this.$emit("close");
    }
  }

  // 其他逻辑
}
</script>
```

## 通过computed计算属性

```js
computed: {
  // 计算样式类型
  typeClass() {
    return `el-alert--${this.type}`;
  },
  // 根据样式类型计算icon
  iconClass() {
    return TYPE_CLASSES_MAP[this.type] || "el-icon-info";
  },
  // 如果有描述文字或者描述文字插槽有内容，则返回`is-big`
  isBigIcon() {
    return this.description || this.$slots.default ? "is-big" : "";
  },
  // 如果有描述文字或者描述文字插槽有内容，则返回`is-bold`
  isBoldTitle() {
    return this.description || this.$slots.default ? "is-bold" : "";
  }
}
```

`逻辑或(||)`的运算优先级要**大于**`三元运算符(...?...:...)`

通过`this.$slots.default`来获取默认插槽内容

