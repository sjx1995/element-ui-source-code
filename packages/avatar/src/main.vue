<script>
export default {
	name: "ElAvatar",

	props: {
		// 尺寸
		size: {
			type: [Number, String],
			validator(val) {
				if (typeof val === "string") {
					return ["large", "medium", "small"].includes(val);
				}
				return typeof val === "number";
			}
		},
		// 形状
		shape: {
			type: String,
			default: "circle",
			validator(val) {
				return ["circle", "square"].includes(val);
			}
		},
		// icon名
		icon: String,
		// 头像图片地址
		src: String,
		// 图片替换文字
		alt: String,
		// 一组用户代理使用的图像
		srcSet: String,
		// 头像加载失败的回调
		error: Function,
		// 图片填充容器的方式
		fit: { type: String, default: "cover" }
	},

	data() {
		return {
			isImageExist: true
		};
	},

	computed: {
		avatarClass() {
			// 从上下文中获取到size、icon、shape
			const { size, icon, shape } = this;
			// 初始化classList
			let classList = ["el-avatar"];
			// 当size为字符串时，设置size属性
			if (size && typeof size === "string") {
				classList.push(`el-avatar--${size}`);
			}
			// 设置icon及形状
			if (icon) {
				classList.push("el-avatar--icon");
			}
			if (shape) {
				classList.push(`el-avatar--${shape}`);
			}
			// 拼接字符串
			return classList.join(" ");
		}
	},

	methods: {
		handleError() {
			const { error } = this;
			const errorFlag = error ? error() : undefined;
			if (errorFlag !== false) {
				this.isImageExist = false;
			}
		},
		renderAvatar() {
			// 从上下文获取变量
			const { icon, src, alt, isImageExist, srcSet, fit } = this;
			// 如果图片存在，则返回图片
			if (isImageExist && src) {
				return <img src={src} onError={this.handleError} alt={alt} srcSet={srcSet} style={{ "object-fit": fit }} />;
			}
			// 如果icon存在则渲染icon
			if (icon) {
				return <i class={icon} />;
			}
			return this.$slots.default;
		}
	},

	render() {
		// 从上下文获取变量
		const { avatarClass, size } = this;
		// 如果size是数字，则使用数字确定avatar大小
		const sizeStyle =
			typeof size === "number"
				? {
						height: `${size}px`,
						width: `${size}px`,
						lineHeight: `${size}px`
				  }
				: {};
		// 设置class和style并渲染
		return (
			<span class={avatarClass} style={sizeStyle}>
				{this.renderAvatar()}
			</span>
		);
	}
};
</script>
