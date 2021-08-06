<template>
	<button
		class="el-button"
		@click="handleClick"
		:disabled="buttonDisabled || loading"
		:autofocus="autofocus"
		:type="nativeType"
		:class="[
			type ? 'el-button--' + type : '',
			buttonSize ? 'el-button--' + buttonSize : '',
			{
				'is-disabled': buttonDisabled,
				'is-loading': loading,
				'is-plain': plain,
				'is-round': round,
				'is-circle': circle
			}
		]"
	>
		<i class="el-icon-loading" v-if="loading"></i>
		<i :class="icon" v-if="icon && !loading"></i>
		<span v-if="$slots.default"><slot></slot></span>
	</button>
</template>
<script>
export default {
	name: "ElButton",

	inject: {
		elForm: {
			default: ""
		},
		elFormItem: {
			default: ""
		}
	},

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
	},

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
	},

	methods: {
		handleClick(evt) {
			this.$emit("click", evt);
		}
	}
};
</script>
