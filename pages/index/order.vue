<template>
	<view class="category-page">
		<div class="titlebox">
			<text style="color: black;font-size: 21px;font-weight:bold">订单</text>
		</div>
		<scroll-view scroll-y="true">
			<div class="orderbody" v-for="item in orderList" @click="goorderdetail(item)">
				<scroll-view scroll-x="true"
					style="height: 70px;width: 70%;margin: 5px 10px;justify-content: center;align-items: center;">
					<image v-for=" good in parseGoods(item.goods)" :src="'http://localhost:8080' + good.src"
						style="height: 65px;width: 65px;border-radius: 6px;margin-left: 5px;" mode="aspectFill"></image>
				</scroll-view>
				<div class="orderinfo">
					<text style="font-weight: bold;font-size: 16px;color: #007AFF;">￥{{item.price}}</text>
					<text style="font-size: 14px;">共计：{{item.number}}件</text>
				</div>
			</div>
		</scroll-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				openid: '',
				orderList: [],
			}
		},
		created() {
			this.getorder()
		},
		methods: {
			getorder() {
				var Storageuserinfo = uni.getStorageSync('Storageuser')
				this.openid = Storageuserinfo.id
				uni.request({
					url: "http://localhost:8080/getOrder",
					method: 'POST',
					header: {
						'content-type': 'application/json',
					},
					data: this.openid,
					success: (res) => {
						this.orderList = res.data
						console.log(res.data)
					}
				})
			},
			parseGoods(goodsStr) {
				try {
					if (!goodsStr) return [];
					const parsed = JSON.parse(goodsStr);
					return Array.isArray(parsed) ? parsed : [parsed];
				} catch (e) {
					console.error('解析商品数据失败:', e);
					return [];
				}
			},
			goorderdetail(item) {
				uni.navigateTo({
					url: `/pages/order/orderdetail?orderData=${encodeURIComponent(JSON.stringify(item))}`
				});
			},
		}
	}
</script>

<style>
	.titlebox {
		margin-top: 12.6%;
		top: 0;
		display: flex;
		flex-direction: row;
		align-items: center;
		margin-left: 15px;
		margin-bottom: 30px;
		letter-spacing: 1px;
	}

	.category-page {
		display: flex;
		flex-direction: column;
		height: 100%;
		background-color: #f5f5f5;
	}

	.orderbody {
		display: flex;
		flex-direction: row;
		width: 95%;
		height: 100px;
		margin: 10px 2.5%;
		background-color: white;
		border-radius: 16px;
		justify-content: space-between;
		align-items: center;
	}

	.orderinfo {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-right: 10px;
	}
</style>