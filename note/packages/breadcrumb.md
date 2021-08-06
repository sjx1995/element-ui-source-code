# Breadcrumb 面包屑组件

> [Breadcrumb 面包屑 - Element文档](https://element.eleme.io/#/zh-CN/component/breadcrumb)

## prop

```js
// Breadcrumb 组件
props: {
  // 分隔符
  separator: { type: String, default: "/" },
  // 自定义分隔icon
  separatorClass: { type: String, default: "" }
}

// Breadcrumb-item 组件
props: {
  // 路由跳转对象，等同于 vue-router 中的 to
  to: {},
  // 在使用to跳转路由时，启用replace将不会向history中添加新纪录
  replace: Boolean
}
```

### 依赖注入(provide / inject)

#### 什么是依赖注入

Vue可以通过`$parent`来访问父组件，但是嵌套很深的孙组件想要访问就变得十分困难，`provide/inject`可以实现这种跨级访问组件数据的功能

#### 使用方法

```vue
// 祖先组件
<script>
export default {
  data() {
    return {
      val: 123
    }
  },
  provide() {
    return {
      someData: this.val
    }
  }
}
</script>
```

```vue
// 后代组件
<script>
export default {
  inject: ["someData"],
  methods: {
    getData() {
      return `injected data: ${this.someData}`
    }
  }
}
</script>
```

#### 注意

你可以将`provide / inject`理解为**大范围的prop**，有别于prop的在于：

- 祖先组件不需要知道那些后代组件使用了他提供的property
- 后代组件不需要知道被注入的property来自于哪个祖先组件

依赖注入会让你的组件的组织方式耦合起来，使重构变得困难

`provide` 和 `inject` 绑定并**不是可响应**的。这是刻意为之的。然而，如果你传入了一个**可监听的对象**，那么其对象的 property 还是**可响应**的。不过当你在祖先组件中需要提供可更新数据时，应该考虑使用`vuex`这种状态管理库。

>[provide/inject - Vue文档](https://cn.vuejs.org/v2/api/#provide-inject)

#### Breadcrumb 组件中的依赖注入

父组件`Breadcrumb`通过`provide`向子组件`BreadcrumbItem`提供了父组件的上下文：

```vue
// Breadcrumb.vue
<script>
export default {
  props: ["separator", "separatorClass"],
  provide() {
    return {
      elBreadcrumb: this
    }
  }
  // 其他逻辑
}
</script>
```

子组件`BreadcrumbItem`通过注入父组件的上下文，获取到父组件的`prop`:

```vue
// breadcrumb-item.vue
<script>
export default {
  inject: ["elBreadcrumb"],
  mounted() {
    this.separator = this.elBreadcrumb.separator;
    this.separatorClass = this.elBreadcrumb.separatorClass;
    // 其他逻辑
  }
}
</script>
```

## 逻辑

### breadcrumb 组件标记当前路由

标记最后一个breadcrumb-item为当前路由：

```js
mounted() {
  const items = this.$el.querySelectorAll(".el-breadcrumb__item");
  if (items.length) {
    items[items.length - 1].setAttribute("aria-current", "page");
  }
}
```

### 渲染 breadcrumb-item 组件并设置逻辑

获取到分隔符供模板渲染，监听子组件点击事件并做路由跳转

```js
mounted() {
  // 从父组件获取分隔符
  this.separator = this.elBreadcrumb.separator;
  this.separatorClass = this.elBreadcrumb.separatorClass;
  // 引用link
  const link = this.$refs.link;
  // 增强语义，告诉辅助阅读设备这个元素扮演的角色
  link.setAttribute("role", "link");
  // 监听点击事件
  link.addEventListener("click", _ => {
    // 从上下文获取prop.to和this.$router
    const { to, $router } = this;
    // 如果不需要跳转直接return
    if (!to || !$router) return;
    // 根据replace选择router跳转方式
    this.replace ? $router.replace(to) : $router.push(to);
  });
}
```
每一个`breadcrumb-item`组件渲染一个可以跳转的路由和分隔符：

```vue
<template>
  <span class="el-breadcrumb__item">
    <span :class="to ? "is-link" : ''" ref="link">
      <slot></slot>
    </span>
    <i v-if="separatorClass" :class="separatorClass"></i>
    <span v-else>{{ separator }}</span>
  </span>
</template>
```

通过`:last-child`选择器让最后一个`breadcrumb-item`组件不显示分隔符：

```css
.el-breadcrumb__item:last-child .el-breadcrumb__separator {
  display: none;
}
```

#### Class 与 Style 的绑定


#### router.replace() 与 router.push()

##### router.push()

```js
router.push(location, onComplete?, onAbort?);
```

`router.push()`可以导航至指定的URL，并且向`history`栈添加一个新纪录，以便用户点击后退按钮可以回到原来页面

第二个参数`onComplete`是个回调函数，在导航完成时触发

第三个参数`onAbort`是个回调函数，在导航终止时（导航到相同页面，或者在前一个路由未完成时导航值新路由导致前一个路由终止）触发

点击`<router-link :to="">`后，内部会调用`router.push()`方法，所以两者作用是等同的

- 通过字符串导航：`router.push("home")`
- 通过对象导航：`router.push({ path: "home" })`
- 通过命名路由导航：`router.push({ name: "homePath" })`
- 携带查询参数：`router.push({ path: "home", query: { id: 123 }})`，等于`/home?id=123`

提供了path，`params`会失效，但是`query`不会：

```js
// 需要导航至 /user/123
const userId = "123";
// 提供path，userId不生效，下面的写法会导航至 /user
router.push({ path: "user", params: { userId }});
// 正确的写法：使用name；或者拼接字符串
router.push({ name: "userPath", params: { userId }});
router.push({ path: `/user/${userId}` });
```

##### router.replace()

```js
router.replace(location, onComplete?, onAbort?);
```

和`router.push()`类型，唯一的不同是它不会向`history`中添加记录，而是替换掉当前的`history`记录

点击`<router-link :to="" replace>`等同于`router.replace()`

##### router.go()

此外还有一个方法`router.go(n)`，可以向前或者向后多少部，正数向前

- `router.go(1)` 等同于 `history.forward()`
- `router.go(-1)` 等同于 `history.back()`