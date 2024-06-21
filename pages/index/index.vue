<template>
    <view>

        <view style="position: fixed;left: 0;top: 150px;width: 120px;z-index: 9">
            <button style="width: 120px;height: 40px;" @click="clearData">清空数据</button>
            <button style="width: 120px;height: 40px;" @click="changeItem">随机修改</button>
            <button style="width: 120px;height: 40px;" @click="loadList">加载10条</button>
            <button style="width: 120px;height: 40px;" @click="setColumn(2)">2列</button>
            <button style="width: 120px;height: 40px;" @click="setColumn(3)">3列</button>
            <button style="width: 120px;height: 40px;" @click="setColumn(4)">4列</button>
            <button style="width: 120px;height: 40px;" @click="setColumn(5)">5列</button>
        </view>

        <view style="padding: 0 30rpx;">

            <cms-waterfalls-flow ref="waterfall" imageKey="image" style="width:100%;" :list="goodsList"
                :column="column">
                <cms-waterfalls-item v-for="(item, index) of goodsList" :item="item" :index="index" :key="item.Id">
                    <view>
                        <image :src="item.image" style="width: 100%;" mode="widthFix" />
                        <view>id:{{ item.id }}</view>
                        <view>label:{{ item.label }}</view>
                    </view>
                </cms-waterfalls-item>
            </cms-waterfalls-flow>
        </view>
    </view>
</template>

<script>

export default {
    data() {
        return {
            index: 0,
            column: 2,
            goodsList: [],
        }
    },
    onLoad(params) {
        this.loadList();
    },
    onReachBottom() {
        this.loadList();
    },
    methods: {
        clearData() {
            this.goodsList = [];
        },
        setColumn(count) {
            this.column = count;
        },
        loadList() {
            for (let i = 0; i < 10; i++) {
                this.goodsList.push(this.giveNewItem())
            }
        },
        giveNewItem() {
            const urls = ['https://s21.ax1x.com/2024/06/21/pkD6ZFI.jpg', 'https://s21.ax1x.com/2024/06/21/pkD6eYt.jpg', 'https://s21.ax1x.com/2024/06/21/pkD6mfP.jpg', 'https://s21.ax1x.com/2024/06/21/pkD6ETA.jpg','https://s21.ax1x.com/2024/06/21/pkD6D0J.jpg','https://s21.ax1x.com/2024/06/21/pkD6Bm4.jpg','https://s21.ax1x.com/2024/06/21/pkD6wXF.jpg'];
            const random = Math.floor(Math.random() * urls.length);
            return {
                id: this.index++,
                label: '商品' + Math.random() * 1000000,
                image: urls[random]
            }
        },
        changeItem() {

            // 随机修改多少个元素
            const count = Math.floor(Math.random() * this.goodsList.length);
            const indexArray = new Array(this.goodsList.length).fill(0).map((item, index) => index);
            for (let i = 0; i < count; i++) {
                const newItem = this.giveNewItem();
                const editIndex = indexArray[Math.floor(Math.random() * indexArray.length)];
                // 修改整个元素
                if (Math.floor(Math.random() * 2) == 0) {
                    this.goodsList[editIndex] = newItem;
                }
                // 修改部分字段
                else {
                    this.goodsList[editIndex].image = newItem.image;
                    this.goodsList[editIndex].label = newItem.label;
                }
                indexArray.splice(editIndex, 1);
            }

        }

    }
}
</script>

<style scoped lang="scss"></style>