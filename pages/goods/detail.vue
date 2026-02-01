<template>
	<div class="container">
		<view class="back-btn" @click="goBack()">
			<image src="/static/arrow-left.png" style="width: 30px;height: 30px;"></image>
		</view>
		<div class="goodsimage">
			<image style="width: 100%;height: 400px;" :src="'http://localhost:8080/' + goodData.src" mode="aspectFill">
			</image>
		</div>
		<div class="goods_main">
			<div class="price-bar">
				<text style="color: #007AFF;font-weight: bold;font-size: 22px;margin: 20px 0;">售价：￥{{goodData.price}}</text>
				<text>销量：{{goodData.sales}}</text>
			</div>
			<text style="font-weight: bold;font-size: 18px;margin: 0 10px;">{{goodData.title}}</text>
		</div>
		<div class="goods_info">
			<div class="express_bar">
				<image src="/static/express.png" style="width: 25px;height: 25px;margin: 0 10px;"></image>
				<text style="color: chartreuse;font-size: 16px;">预计明天发货 | 承诺48小时内发货</text>
			</div>
			<div class="infomation">
				<text style="font-weight: bold;font-size: 16px;margin: 5px 10px;">产品详细：</text>
				<text style="font-size: 16px;margin: 0 10px;">{{goodData.info}}</text>
			</div>
		</div>
		<div class="bottom-bar">
			<button class="cart-btn" @click="handleCart">加入购物车</button>
			<button class="buy-btn" @click="handleBuy()">立即购买</button>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				goodData: [],
				openid:"",
			}
		},
		onLoad(options) {
			const goodsDataStr = decodeURIComponent(options.goodsData);
			this.goodData = JSON.parse(goodsDataStr);
		},
		methods: {
			handleBuy(){
				this.getinfo()
				let order={
					"UID":this.openid,
					"goods":JSON.stringify([this.goodData]),
					"number":1,
					"price": this.goodData.price
				}
				console.log(order);
				uni.request({
					url:"http://localhost:8080/UploadOderForm",
					method: 'POST',
					header: {
						'content-type': 'application/json',
					},
					data:order,
					success: (res) => {
						if (res.data === 'New record created successfully') {
							uni.showToast({
								icon:"success",
								title:"购买成功",
								duration:2000
							})
						}
					}
				})
			},
			goBack() {
				uni.navigateBack({
					delta: 1 
				});
			},
			getinfo() {
				var Storageuserinfo = uni.getStorageSync('Storageuser')
				this.openid = Storageuserinfo.id
			},
		},
	}
</script>

<style>
	.back-btn {
		position: fixed;
		top: 50px;
		left: 15px;
		width: 32px;
		height: 32px;
		border-radius: 5px;
		background-color: rgba(50, 50, 50, 0.8);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 100;
	}

	.container {
		background-color: #F8F8F8;
	}

	.price-bar {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		margin: 0 10px;
	}

	.goods_main {
		background-color: white;
		border-radius: 16px;
	}

	.goods_info {
		margin: 8px 0;
		background-color: white;
		border-radius: 16px;
	}

	.express_bar {
		display: flex;
		flex-direction: row;
		align-items: center;
		border-bottom: 0.5px solid rgba(50, 50, 50, 0.2);
	}

	.infomation {
		display: flex;
		flex-direction: column;
	}

	.bottom-bar {
		position: fixed;
		left: 0;
		bottom: 0;
		width: 100%;
		height: 60px;
		display: flex;
		background-color: #fff;
		box-shadow: 0 -2px 8px rgba(0, 0, 0, 0.1);
	}

	.bottom-bar button {
		flex: 1;
		height: 100%;
		line-height: 60px;
		border: 0;
		font-size: 16px;
		border-radius: 0;
	}

	.cart-btn {
		background-color: #fff;
		color: #ff5722;
		border-right: 1px solid #f2f2f2;
	}

	.buy-btn {
		background-color: #ff5722;
		color: #fff;
	}
</style>