# Card 卡片组件

> [Card 卡片 - Element文档](https://element.eleme.io/#/zh-CN/component/card)

## prop

```js
props: {
  // 标题
  header: {},
  // body样式
  bodyStyle: {},
  // 阴影显示时机
  shadow: { type: String }
}
```

## header

```vue
// card.vue
<template>
  <div class="el-card">
    <div class="el-card__header" v-if="$slots.header || header">
      <slot name="header">{{ header }}</slot>
    </div>
    <div class="el-card__body" :style="bodyStyle">
      <slot></slot>
    </div>
  </div>
</template>
```

因为默认插槽是card的内容，这里使用`具名插槽`来指定标题，所以可以传入字符串来指定标题，也可以使用`<div slot="header">标题</div>`来设置标题

`<slot name="header">{{ header }}</slot>`中的字符串模板提供后备内容，如果没有插槽（即用户可能通过字符串设定标题），则渲染模板中的内容

### 插槽

插槽可以理解为一个占位符，待组件渲染时会被替换成传入的内容

#### 内容插槽

```vue
// home.vue
<template>
  <child> 按钮名称 </child>
</template>

// child.vue
<template>
  <button> <slot></slot> </button>
</template>
```

会被渲染成：

```vue
<template>
  <button>按钮名称</button>
</template>
```

##### 作用域

`home`组件中的插槽可以访问这个整个`home`的作用域，但不能访问`child`的作用域

即父级模板中的内容都是在父级作用域中编译的，子级模板中的内容都是在子级作用域中编译的

#### 后备内容

子级模板中可以在<slot>标签内设置默认内容，当父级模板没有传递内容时可以做后备渲染

```vue
// child.vue
<template>
  <slot>我是默认的内容</slot>
</template>
```

#### 具名插槽

当我们需要多个插槽的时候，<slot>元素有一个`name`属性可以帮到我们

```vue
// child
<template>
  <div>
    <header>
      <slot name="header"></slot>
    </header>
    
    <main>
      <slot></slot>
    </main>

    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

`<slot>`不带`name`属性时，他的默认`name`就是`default`

此时我们调用`child`组件时，通过在`<template>`元素上添加`v-slot`指令，来告诉vue我们希望这段内容添加到哪个插槽中

```vue
// home.vue
<template>
  <child>
    <template v-slot="header">
      <h1>This is a title</h1>
    </template>

    <article>This is content</article>

    <template v-slot="footer">
      <p>This is a footer</p>
    </template>
  </child>
</template>
```

没有指明`v-slot`的内容就会被渲染进入默认插槽，当然也可以手动指明`v-slot="default"`

`v-slot`只能添加到`<template>`上，只有一种例外（独占默认插槽的缩写语法）

在vue@2.6.0+版本中，原先的`slot`和`slot-scope`被弃用，在vue@2.x的版本中还会被支持，但是在vue@3.x版本中将被废弃：

> [插槽#废弃了的语法 - vue文档](https://cn.vuejs.org/v2/guide/components-slots.html#%E5%BA%9F%E5%BC%83%E4%BA%86%E7%9A%84%E8%AF%AD%E6%B3%95)

具名插槽`<template name="header"></template>`可以缩写为`<template #header></template>`

#### 作用域插槽

如果我们希望插槽访问子级模板的作用域，就可以使用`作用域插槽`

父级模板要访问子级模板的属性，我们需要把要传递的内容绑定到子级模板的`<slot>`上，然后在父级模板上使用`v-slot`设置一个值来定义插槽名字

```vue
// child.vue
<template>
  <div>
    <slot v-bind:childVal="name">{{ name }}</slot>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: "child"
    }
  }
}
</script>
```

在父级模板中接受传递过来的值：

```vue
<template>
  <test v-slot:default="slotProp">
    {{ slotProp.name }}
  </test>
</template>
```

父级模板中的`slotProp`是可以自定义的

##### 独占默认插槽的缩写语法

上面的例子中，`child`组件只提供了默认插槽`<slot name="default"></slot`，我们在父级模板中就可以简写`<test v-slot="slotProp">{{ slotProp.name }}</test>`

当出现多个插槽的情况，`不能缩写`

这也是唯一一种能在`非<template>元素`上使用`v-slot`的情况

##### 解构插槽Prop

上面的例子中，父级模板中的`插槽Prop`可以使用解构语法：

```vue
// home.vue
<template>
  <test v-slot="{name}">
    {{ name }}
  </test>
</template>
```

也可以使用解构重命名语法`v-slot="{name: newName}"`

###### 插槽其实是个函数

作用域插槽的工作原理是将插槽内容包裹在一个单参数函数中：

```js
function (slotProp) { /* 插槽内容 */ }
```

#### 动态插槽名

```vue
const attributeName = "default"

<template v-slot:[attributeName]></template>
```

## 动态设置Style

传入一个对象作为body的样式：

```vue
// 父组件调用el-card
<template>
  <el-card :bodyStyle="{ padding: '16px' }"></el-card>
</template>
```

渲染出的组件就会拥有这个样式：

```vue
// 渲染出的组件
<template>
  <div class="el-card">
    <div class="el-card__body" style="padding:'16px';"></div>
  </div>
</template>
```

> [绑定内联样式 - vue文档](https://cn.vuejs.org/v2/guide/class-and-style.html#%E7%BB%91%E5%AE%9A%E5%86%85%E8%81%94%E6%A0%B7%E5%BC%8F)