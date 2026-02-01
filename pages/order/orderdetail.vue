<template>
	<div class="order-detail">
		<div class="titlebox">
			<text style="color: black;font-size: 21px;font-weight:bold">订单详细</text>
		</div>
		<div class="order-item" v-if="order">
			<div class="order-info">
				<p><strong>订单号：</strong>{{ order.id }}</p>
				<p><strong>下单时间：</strong>{{ order.createTime }}</p>
				<p><strong>商品数量：</strong>{{ order.number }}</p>
				<p><strong>订单总价：</strong>¥{{ order.price }}</p>
			</div>
			<div class="goods-detail" v-if="parsedGoods" v-for="item in parsedGoods">
				<img :src="'http://localhost:8080'+item.src" alt="商品图" class="goods-img" />
				<div class="goods-info">
					<p><strong>商品名称：</strong>{{ item.title }}</p>
					<p><strong>商品单价：</strong>¥{{ item.price }}</p>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				order: null
			};
		},
		onLoad(options) {
			try {
				const orderDataStr = decodeURIComponent(options.orderData);
				this.order = JSON.parse(orderDataStr);
			} catch (error) {
				console.error("获取订单数据失败", error);
				uni.showToast({
					title: '订单数据加载失败',
					icon: 'none'
				});
			}
		},
		computed: {
			parsedGoods() {
				try {
					return this.order ? JSON.parse(this.order.goods) : null;
				} catch (error) {
					console.error("解析goods失败", error);
					return null;
				}
			}
		}
	};
</script>

<style scoped>
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
	.order-detail {
		padding: 20px;
		background: #fff;
		border-radius: 8px;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
	}

	.order-item {
		margin-bottom: 20px;
		padding: 15px;
		border: 1px solid #eee;
		border-radius: 16px;
	}

	.order-info p {
		margin: 6px 0;
		color: #333;
	}

	.goods-detail {
		display: flex;
		align-items: center;
		margin-top: 10px;
	}

	.goods-img {
		width: 80px;
		height: 80px;
		object-fit: cover;
		margin-right: 15px;
		border: 1px solid #eee;
		border-radius: 4px;
	}

	.goods-info p {
		margin: 4px 0;
		color: #666;
	}
</style>