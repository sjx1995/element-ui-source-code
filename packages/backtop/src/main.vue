<template>
	<transition name="el-fade-in">
		<div
			v-if="visible"
			@click.stop="handleClick"
			:style="{
				right: styleRight,
				bottom: styleBottom
			}"
			class="el-backtop"
		>
			<slot>
				<el-icon name="caret-top"></el-icon>
			</slot>
		</div>
	</transition>
</template>

<script>
import throttle from "throttle-debounce/throttle";

// 定义滚动动画函数
const cubic = value => Math.pow(value, 3);
const easeInOutCubic = value => (value < 0.5 ? cubic(value * 2) / 2 : 1 - cubic((1 - value) * 2) / 2);

export default {
	name: "ElBacktop",

	props: {
		// 出现返回顶部按钮高度
		visibilityHeight: { type: Number, default: 200 },
		// 需要滚动的元素：css选择器
		target: [String],
		// 距离页面右边距
		right: { type: Number, default: 40 },
		// 距离页面下边距
		bottom: { type: Number, default: 40 }
	},

	data() {
		return {
			el: null, // 需要滚动到顶的元素
			container: null, // 监听滚动事件的元素
			visible: false // 是否显示滚动到顶的按钮
		};
	},

	computed: {
		// 设置定位位置
		styleBottom() {
			return `${this.bottom}px`;
		},
		styleRight() {
			return `${this.right}px`;
		}
	},

	mounted() {
		// 初始化：确定需要滚动的元素
		this.init();
		// 节流检查是否显示滚动到顶按钮
		this.throttledScrollHandler = throttle(300, this.onScroll);
		// 监听滚动事件并触发onScroll()方法
		this.container.addEventListener("scroll", this.throttledScrollHandler);
	},

	methods: {
		init() {
			// document.documentElement === <html>元素
			// 需要滚动的元素默认为<html>元素
			this.container = document;
			this.el = document.documentElement;
			// 如果传入了target，则将需要滚动的元素保存到el
			if (this.target) {
				this.el = document.querySelector(this.target);
				if (!this.el) {
					throw new Error(`target is not existed: ${this.target}`);
				}
				this.container = this.el;
			}
		},
		onScroll() {
			// 获取需要滚动到顶的元素的垂直滚动的像素数
			const scrollTop = this.el.scrollTop;
			// 判断时候需要显示按钮
			this.visible = scrollTop >= this.visibilityHeight;
		},
		handleClick(e) {
			// 触发滚动到顶方法
			this.scrollToTop();
			// 抛出`click`事件供调用者监听，外部可以拿到mouseEvent
			this.$emit("click", e);
		},
		scrollToTop() {
			// 需要滚动的元素
			const el = this.el;
			// 开始滚动时间
			const beginTime = Date.now();
			// 开始滚动的位置
			const beginValue = el.scrollTop;
			// 检查是否支持requestAnimationFrame，不支持的话使用setTimeout
			const rAF = window.requestAnimationFrame || (func => setTimeout(func, 16));
			// 滚动动画
			const frameFunc = () => {
				const progress = (Date.now() - beginTime) / 500;
				// 如果滚动过程小于500ms，则继续执行滚动动画
				// 否则滚动到顶结束动画
				if (progress < 1) {
					el.scrollTop = beginValue * (1 - easeInOutCubic(progress));
					rAF(frameFunc);
				} else {
					el.scrollTop = 0;
				}
			};
			rAF(frameFunc);
		}
	},
	// 取消监听
	beforeDestroy() {
		this.container.removeEventListener("scroll", this.throttledScrollHandler);
	}
};
</script>
