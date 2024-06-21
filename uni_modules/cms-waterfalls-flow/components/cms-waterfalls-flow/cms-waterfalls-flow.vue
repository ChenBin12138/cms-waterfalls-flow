<template>
	<view id="waterfall" class="waterfall-parent" :style="{
		height: innerH + 'px'
	}">
		<block v-if="list && list.length">
			<slot />
		</block>
	</view>
</template>
<script>

// 防抖
function debounce(func, wait) {
	let timeout;
	return function () {
		const context = this;
		const args = arguments;
		clearTimeout(timeout);
		timeout = setTimeout(() => {
			func.apply(context, args);
		}, wait);
	};
}

export default {
	props: {
		list: {
			type: Array,
			default: () => []
		},
		// 图片的标识
		imageKey: {
			type: String,
			default: "image"
		},
		// 列数
		column: {
			type: Number,
			default: 2
		},
		// rpx
		vGap: {
			type: [Number, String],
			default: 30
		},
		// rpx
		hGap: {
			type: [Number, String],
			default: 30
		}

	},
	data() {
		return {
			itemQueue: [],
			heights: [],
			colWidth: 0,
			nowIndex: 0,
			innerH: 0,
			childRefs: [],
			resetStartItemIndex: 99999
		}
	},
	mounted() {
		this.resetData();
	},
	methods: {
		dataChange(index) {
			if (this.resetStartItemIndex > index) {
				this.resetStartItemIndex = index;
			}
			this.resetColumnByIndex();
		},
		resetColumnByIndex: debounce(function () {
			// const index = this.resetStartItemIndex;
			// this.resetStartItemIndex = 99999;
			// this.nowIndex = index;
			// for (let i = index; i < this.childRefs.length; i++) {
			// 	this.childRefs[i].reLoad();
			// }

			// 此处暂时刷新全部，后续仅根据索引更新部分
			this.resetData();
		}, 300),
		addChild(ref) {
			if (!this.childRefs.find(item => item == ref)) {
				this.childRefs.push(ref);
				this.childRefs.sort((a, b) => a.$props.index - b.$props.index);
			}
		},
		removeChild(ref) {
			const removeIndex = this.childRefs.findIndex(item => item == ref);
			if (removeIndex != -1) {
				this.childRefs.splice(removeIndex, 1);
				this.childRefs.sort((a, b) => a.$props.index - b.$props.index);
			}
		},
		resetData() {
			this.colWidth = 0;
			this.nowIndex = 0;
			this.heights = new Array(this.column).fill(0);
			this.itemQueue = [];

			if (this.childRefs && this.childRefs.length) {
				this.childRefs.forEach(ref => {
					ref && ref.reLoad();
				})
			}
		},
		// 
		getItemStyle(itemH, index, callback) {

			// 如果顺序正确处理否则插入队列等待处理
			if (this.nowIndex == index) {

				// 计算高度
				let findLowIndex = 0;
				this.heights.forEach((h, i) => {
					if (this.heights[findLowIndex] > h) {
						findLowIndex = i;
					}
				});

				let left = findLowIndex * this.colWidth + findLowIndex * uni.upx2px(this.vGap);
				let top = this.heights[findLowIndex];
				const hGapPx = uni.upx2px(this.hGap);

				if (top) {
					top += hGapPx
				}
				this.heights[findLowIndex] = this.heights[findLowIndex] + itemH + hGapPx;
				this.innerH = this.heights.reduce((max, num) => max < num ? num : max, 0);

				callback && callback({
					left,
					top
				})

				this.nowIndex++;
				const findNextIndex = this.itemQueue.findIndex(item => item.index == this.nowIndex);
				if (findNextIndex != -1) {
					const nextItem = this.itemQueue.splice(findNextIndex, 1)[0];
					this.getItemStyle(nextItem.itemH, nextItem.index, nextItem.callback);
				}
			}
			else {
				this.itemQueue.push({
					itemH, index, callback
				})
			}

		},
		getColumnWidth() {
			if (this.colWidth) {
				return this.colWidth
			}

			return new Promise((resolve, reject) => {
				const query = uni.createSelectorQuery().in(this);
				query.select("#waterfall").boundingClientRect((data) => {
					this.colWidth = (data.width - (this.column - 1) * uni.upx2px(this.vGap)) / this.column;
					resolve(this.colWidth);
				}).exec();
			})
		}
	},
	watch: {
		column() {
			this.resetData();
		},
		list(n, o) {
			if (!n || !n.length) {
				this.resetData();
			}
		}
	}
}
</script>
<style scoped lang="scss">
.waterfall-parent {
	position: relative;
	display: flex;
	justify-content: space-between;
	flex-wrap: wrap;
}
</style>