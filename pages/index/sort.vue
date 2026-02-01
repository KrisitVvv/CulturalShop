<template>
	<view class="category-page">
		<div class="titlebox">
			<text style="color: black;font-size: 21px;font-weight:bold">分类</text>
		</div>
		<view class="header">
			<div class="search-bar"></div>
			<view class="filter-group">
				<text class="filter-item" :class="{active: sortType === 'sales'}" @click="sortType = 'sales'">销量</text>
				<text class="filter-item" :class="{active: sortType === 'price'}" @click="sortType = 'price'">价格</text>
			</view>
		</view>
		<view class="main-container">
			<scroll-view class="category-scroll" scroll-y show-scrollbar="false">
				<view v-for="(item, index) in categories" :key="index" class="category-item"
					:class="{active: currentCategory === index}" @click="changeCategory(index)">
					<view class="category-text">{{ item.name }}</view>
					<view class="sort-tag" v-if="currentCategory === index && item.sort !== 'none'">
						<text>{{ item.sort === 'asc' ? '↑' : '↓' }}</text>
					</view>
				</view>
			</scroll-view>
			<scroll-view class="goods-scroll" scroll-y show-scrollbar="false">
				<view v-for="(goods, idx) in filteredGoods" :key="goods.id" class="goods-item" @click="godetail(goods)">
					<image class="goods-img" :src="'http://localhost:8080/' + goods.src" mode="aspectFill"></image>
					<view class="goods-info">
						<view class="goods-title">{{ goods.title }}</view>
						<view class="goods-desc">{{ goods.info }}</view>
						<view class="price-group">
							<text class="current-price">¥{{ goods.price }}</text>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				categories: [{
						name: '全部',
						sort: 'none'
					},
					{
						name: 'IP周边',
						sort: 'none'
					},
					{
						name: '生活用品',
						sort: 'none'
					},
					{
						name: '办公用品',
						sort: 'none'
					},
					{
						name: '其它类',
						sort: 'none'
					}
				],
				currentCategory: 0,
				sortType: 'price',
				goodsList: [],
			}
		},
		created() {
			this.goodsList = uni.getStorageSync('goodsList')
		},
		computed: {
			filteredGoods() {
				let list = [...this.goodsList]
				const currentCategoryName = this.categories[this.currentCategory].name
				if (currentCategoryName !== '全部') {
					list = list.filter(goods => goods.category === currentCategoryName)
				}
				const categorySort = this.categories[this.currentCategory].sort
				if (categorySort === 'asc') {
					list.sort((a, b) => a.price - b.price) 
				} else if (categorySort === 'desc') {
					list.sort((a, b) => b.price - a.price)
				}
				if (this.sortType === 'price') {
					list.sort((a, b) => a.price - b.price)
				}
				return list
			}
		},
		methods: {
			changeCategory(index) {
				this.currentCategory = index
				const currentSort = this.categories[index].sort
				if (currentSort === 'none') {
					this.categories[index].sort = 'asc'
				} else if (currentSort === 'asc') {
					this.categories[index].sort = 'desc'
				} else {
					this.categories[index].sort = 'none'
				}
			},
			addToCart(goods) {
				goods.badge = (goods.badge || 0) + 1
			},
			godetail(item){
				uni.navigateTo({
					url: `/pages/goods/detail?goodsData=${encodeURIComponent(JSON.stringify(item))}`
				});
			},
		}
	}
</script>

<style scoped>
	.titlebox {
		margin-top: 10%;
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
	
	.header {
		display: flex;
		align-items: center;
		justify-content: right;
		padding: 10rpx 20rpx;
		background-color: #fff;
		border-bottom: 1px solid #eee;
	}

	.search-bar {
		flex: 1;
		margin-right: 20rpx;
	}

	.filter-group {
		display: flex;
		align-items: center;
	}

	.filter-item {
		margin-left: 30rpx;
		font-size: 28rpx;
		color: #666;
	}

	.filter-item.active {
		color: #007aff;
		font-weight: bold;
	}


	.main-container {
		display: flex;
		flex: 1;
	}


	.category-scroll {
		width: 200rpx;
		background-color: #fff;
	}

	.category-item {
		height: 80rpx;
		line-height: 80rpx;
		text-align: center;
		font-size: 28rpx;
		border-bottom: 1px solid #eee;
	}

	.category-item.active {
		background-color: #f5f5f5;
		color: #007aff;
	}


	.goods-scroll {
		flex: 1;
		padding: 10rpx;
	}

	.goods-item {
		display: flex;
		align-items: center;
		background-color: #fff;
		margin-bottom: 10rpx;
		padding: 15rpx;
		border-radius: 12rpx;
	}

	.goods-img {
		width: 150rpx;
		height: 150rpx;
		margin-right: 20rpx;
		border-radius: 8rpx;
	}

	.goods-info {
		flex: 1;
	}

	.goods-title {
		font-size: 32rpx;
		font-weight: bold;
		margin-bottom: 8rpx;
	}

	.goods-desc {
		font-size: 26rpx;
		color: #666;
		margin-bottom: 8rpx;
	}

	.stock-sales {
		font-size: 24rpx;
		color: #999;
		display: flex;
		justify-content: space-between;
		margin-bottom: 8rpx;
	}

	.price-group {
		display: flex;
		align-items: baseline;
	}

	.current-price {
		font-size: 32rpx;
		color: #ff4d4f;
		margin-right: 15rpx;
	}

	.original-price {
		font-size: 26rpx;
		color: #999;
		text-decoration: line-through;
	}


	.cart-btn {
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.badge {
		position: absolute;
		top: -10rpx;
		right: -10rpx;
		background-color: red;
		color: #fff;
		font-size: 20rpx;
		padding: 5rpx 10rpx;
		border-radius: 50%;
		min-width: 30rpx;
		text-align: center;
	}

	.category-item {
		position: relative;
		height: 80rpx;
		line-height: 80rpx;
		text-align: center;
		font-size: 28rpx;
		border-bottom: 1px solid #eee;
		display: flex;
		justify-content: center;
		align-items: center;
	}


	.category-text {
		flex: 1;
		padding: 0 20rpx;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.sort-tag {
		position: absolute;
		right: 15rpx;
		width: 24rpx;
		height: 24rpx;
		line-height: 24rpx;
		text-align: center;
		font-size: 20rpx;
		color: #007aff;
	}
</style>