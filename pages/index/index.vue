<template>
	<div class="body" @touchmove.stop.prevent="() => {}">
		<div class="layout_top">
			<text style="margin-top: auto;font-size: 22px;font-weight: bold;color: white;">文创商城</text>
			<div class="search_bar">
				<input type="text"></input>
				<div
					style="display: flex;flex-direction: column;;height: 35px; width: 60px;background-color: #007AFF;border-radius: 6px;text-align: center;justify-content: center;margin-right: 3px">
					<text style="color: white;font-size: 14px;margin: auto;letter-spacing: 2px;">搜索</text>
				</div>
			</div>
		</div>
		<scroll-view scroll-y="true" show-scrollbar="false" style="height: calc(100vh - 130px);">
			<div class="container">
				<div class="colum">
					<div class="item" v-for="item in data1" @click="godetail(item)">
						<image class="item-image" :src="'http://localhost:8080/' + item.src" @load="onImageLoad(item,$event)"
							:mode="item.newHeight == 240 ? 'aspectFill' : 'aspectFit'"
							:style="item.newHeight ? `height: ${item.newHeight}px;` : 'height: 180px;'"></image>
						<text
							style="color: black;font-size: 18px;font-weight: bold;margin: 0 0 5px 5px;white-space:nowarp;over-flow:hidden;">{{item.title}}</text>
						<text style="color: #007AFF;font-size: 20px;margin: 0 0 5px 5px;font-weight: bold;">￥{{item.price}}</text>
					</div>
				</div>
				<div class="colum" style="margin: 5px 5px 5px 0;">
					<div class="item" v-for="item in data2" @click="godetail(item)">
						<image class="item-image" :src="'http://localhost:8080/' + item.src" @load="onImageLoad(item,$event)" mode="aspectFit"
							:style="item.newHeight ? `height: ${item.newHeight}px;` : 'height: 180px;'"></image>
						<text
							style="color: black;font-size: 18px;font-weight: bold;margin: 0 0 5px 5px;">{{item.title}}</text>
						<text style="color: #007AFF;font-size: 20px;margin: 0 0 5px 5px;font-weight: bold;">￥{{item.price}}</text>
					</div>
				</div>
			</div>
		</scroll-view>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				Goodsdata: [],
				data1: [],
				data2: [],
			}
		}, 
		created() {
			this.getData();
			this.Goodsdata.forEach(item => {
				item.newHeight = null;
			});

		},
		methods: {
			getData() {
				uni.request({
					url: "http://localhost:8080/getgoodslist",
					success: (res) => {
						this.Goodsdata = res.data;
						uni.setStorageSync("goodsList", this.Goodsdata);
						let i = 0;
						this.data1 = [];
						this.data2 = [];
						while (i < this.Goodsdata.length) {
							this.data1.push(this.Goodsdata[i++]);
							if (i < this.Goodsdata.length) {
								this.data2.push(this.Goodsdata[i++]);
							}
						}
					}
				});
			},
			onImageLoad(item, event) {
				try {
					const width = event.detail.width;
					const height = event.detail.height;
					const ratio = width / height;
					this.$set(item, 'aspectRatio', ratio);
					if (ratio === null) {
						this.$set(item, 'newHeight', "180");
					}
					if (ratio < 1) {
						this.$set(item, 'newHeight', "240");
					} else {
						const newHeight = (height * 183.5 / width).toFixed(2);
						console.log(newHeight)
						this.$set(item, 'newHeight', newHeight);
					}
				} catch (error) {
					console.error('图片加载错误:', error);
				}
			},
			godetail(item){
				uni.navigateTo({
					url: `/pages/goods/detail?goodsData=${encodeURIComponent(JSON.stringify(item))}`
				});
			},
		}
	}
</script>

<style>
	.body {
		width: 100%;
		height: 100%;
		background-color: #007AFF;
	}

	.layout_top {
		width: 100%;
		height: 145px;
		display: flex;
		flex-direction: column;
		margin: 0 0 0 10px;
	}

	.container {
		margin: auto;
		width: 100%;
		display: flex;
		flex-direction: row;
		background-color: #e7e7e7;
		border-radius: 15px 15px 0 0;
	}

	.colum {
		border-radius: 14px;
		margin: 5px;
		display: flex;
		flex-direction: column;
		flex: 1;
		padding: 0 2px;
	}

	.item {
		background-color: white;
		border-radius: 14px;
		margin-bottom: 8px;
		display: flex;
		flex-direction: column;
		width: 100%;
	}

	.item-image {
		width: 100%;
		object-fit: cover;
		border-radius: 14px 14px 0 0;
		margin-bottom:5px ;
	}

	.search_bar {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		background-color: white;
		height: 40px;
		width: 96%;
		border-radius: 7px;
		margin: 10px 0;
		align-items: center;
	}
</style>