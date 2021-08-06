<template>
	<div class="el-card" :class="shadow ? 'is-' + shadow + '-shadow' : 'is-always-shadow'">
		<div class="el-card__header" v-if="$slots.header || header">
			<slot name="header">{{ header }}</slot>
		</div>
		<div class="el-card__body" :style="bodyStyle">
			<slot></slot>
		</div>
	</div>
</template>

<script>
export default {
	name: "ElCard",
	props: {
		// 标题
		header: {},
		// body样式
		bodyStyle: {},
		// 阴影显示时机
		shadow: { type: String }
	}
};
</script>
