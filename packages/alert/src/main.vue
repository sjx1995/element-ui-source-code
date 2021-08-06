<template>
	<transition name="el-alert-fade">
		<div class="el-alert" :class="[typeClass, center ? 'is-center' : '', 'is-' + effect]" v-show="visible" role="alert">
			<i class="el-alert__icon" :class="[iconClass, isBigIcon]" v-if="showIcon"></i>
			<div class="el-alert__content">
				<span class="el-alert__title" :class="[isBoldTitle]" v-if="title || $slots.title">
					<slot name="title">{{ title }}</slot>
				</span>
				<p class="el-alert__description" v-if="$slots.default && !description"><slot></slot></p>
				<p class="el-alert__description" v-if="description && !$slots.default">{{ description }}</p>
				<i
					class="el-alert__closebtn"
					:class="{ 'is-customed': closeText !== '', 'el-icon-close': closeText === '' }"
					v-show="closable"
					@click="close()"
				>
					{{ closeText }}
				</i>
			</div>
		</div>
	</transition>
</template>

<script type="text/babel">
const TYPE_CLASSES_MAP = {
	success: "el-icon-success",
	warning: "el-icon-warning",
	error: "el-icon-error"
};
export default {
	name: "ElAlert",

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
	},

	data() {
		return {
			visible: true
		};
	},

	methods: {
		// 关闭触发close事件
		close() {
			this.visible = false;
			this.$emit("close");
		}
	},

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
};
</script>
