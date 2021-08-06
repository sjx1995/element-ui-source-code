<template>
	<span class="el-breadcrumb__item">
		<span :class="['el-breadcrumb__inner', to ? 'is-link' : '']" ref="link" role="link">
			<slot></slot>
		</span>
		<i v-if="separatorClass" class="el-breadcrumb__separator" :class="separatorClass"></i>
		<span v-else class="el-breadcrumb__separator" role="presentation">{{ separator }}</span>
	</span>
</template>
<script>
export default {
	name: "ElBreadcrumbItem",
	props: {
		// 路由跳转对象，等同于 vue-router 中的 to
		to: {},
		// 在使用to跳转路由时，启用replace将不会向history中添加新纪录
		replace: Boolean
	},
	data() {
		return {
			separator: "",
			separatorClass: ""
		};
	},

	inject: ["elBreadcrumb"],

	mounted() {
		// 从父组件获取分隔符
		this.separator = this.elBreadcrumb.separator;
		this.separatorClass = this.elBreadcrumb.separatorClass;
		// 引用link
		const link = this.$refs.link;
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
};
</script>
