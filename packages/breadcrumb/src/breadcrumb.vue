<template>
	<div class="el-breadcrumb" aria-label="Breadcrumb" role="navigation">
		<slot></slot>
	</div>
</template>
<script>
export default {
	name: "ElBreadcrumb",

	props: {
		// 分隔符
		separator: { type: String, default: "/" },
		// 自定义分隔icon
		separatorClass: { type: String, default: "" }
	},
	// 提供父组件上下文
	provide() {
		return {
			elBreadcrumb: this
		};
	},
	// 标记最后一个breadcrumbItem为当前路由
	mounted() {
		const items = this.$el.querySelectorAll(".el-breadcrumb__item");
		if (items.length) {
			items[items.length - 1].setAttribute("aria-current", "page");
		}
	}
};
</script>
