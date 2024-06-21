<template>
	<view id="waterfallItem" class="waterfall-item" :class="{
		hidden: hiddenState
	}" :style="{
		width,
		top,
		left
	}">
		<slot />

		<image v-if="imageSrc" :src="imageSrc" @load="getElementHeight" />
	</view>
</template>
<script>
export default {
	props: {
		item: {
			type: Object,
			default: () => ({})
		},
		index: {
			type: [Number, String],
			default: 0
		}
	},
	data() {
		return {
			hiddenState: true,
			width: 0,
			top: 0,
			left: 0,
			imageSrc: "",
			componentParent: null
		}
	},
	mounted() {

		// #ifdef H5
		this.componentParent = this.$parent.$parent;
		// #endif	
		// #ifndef H5
		this.componentParent = this.$parent;
		// #endif

		this.componentParent.addChild(this);
		this.reLoad();
	},
	beforeDestroy() {
		this.componentParent.removeChild(this);
	},
	methods: {
		async reLoad() {
			const colWidth = await this.componentParent.getColumnWidth();
			this.width = colWidth + 'px';

			const imageKey = this.componentParent.$props.imageKey;
			if (imageKey && this.item[imageKey]) {
				this.imageSrc = this.item[imageKey];
			}
			else {
				this.getElementHeight();
			}
		},
		getElementHeight() {
			this.imageSrc = "";
			setTimeout(() => {
				const query = uni.createSelectorQuery().in(this);
				query.select("#waterfallItem").boundingClientRect((data) => {
					this.componentParent.getItemStyle(data.height, this.index, style => {
						this.top = style.top + 'px';
						this.left = style.left + 'px';
						this.hiddenState = false;
					})
				}).exec();
			}, 100);

		}
	},
	watch: {
		item: {
			deep: true,
			handler(n) {
				this.componentParent.dataChange(this.index);
			}
		}
	}

}
</script>
<style scoped lang="scss">
.waterfall-item {
	position: absolute;
	height: auto;
	overflow: hidden;
}

.waterfall-item.hidden {
	opacity: 0;
}
</style>