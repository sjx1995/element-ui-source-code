<template>
	<div class="el-badge">
		<slot></slot>
		<transition name="el-zoom-in-center">
			<sup
				v-show="!hidden && (content || content === 0 || isDot)"
				v-text="content"
				class="el-badge__content"
				:class="[
					'el-badge__content--' + type,
					{
						'is-fixed': $slots.default,
						'is-dot': isDot
					}
				]"
			>
			</sup>
		</transition>
	</div>
</template>

<script>
export default {
	name: "ElBadge",

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
	},

	computed: {
		// 计算数字
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
	}
};
</script>
