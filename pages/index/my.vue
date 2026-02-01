<template>
	<div class="container">
		<div class="head_box">
			<div class="head">
				<text style="color: white;font-weight: bold;font-size: 22px;margin-left: 10px;">我的</text>
			</div>
		</div>
		<div class="usercard">
			<div class="userfirst">
				<template v-if="username!=null">
					<div class="userAvatar" @click="showSettingCard()">
						<image src="/static/avatar.png" style="width: 70px;height: 70px;"></image>
					</div>
					<div class="userNickName" @click="showSettingCard()">
						<text>{{ username }}</text>
					</div>
				</template>
				<template v-else>
					<div class="userAvatar" @click="showpopup()">
						<image style="height: 70px;width: 70px;" src="https://mmbiz.qpic.cn/mmbiz/icTdbqWNOwNRna42FI242Lcia07jQodd2FJGIYQfG0LAJGFxM4FbnQP6yfMxBgJ0F3YRqJCJ1aPAK2dQagdusBZg/0"></image>
					</div>
					<div class="userNickName" @click="showpopup()">
						<text>请登录账号</text>
					</div>
				</template>
			</div>
			<div class="line"></div>
			<div class="usersecond">
				<div class="userinfo">
					<text style="font-weight: bold;">消费数量</text>
					<text style="color: #007AFF;font-weight: bold;">1</text>
				</div>
				<div class="userinfo">
					<text style="font-weight: bold;">消费金额</text>
					<text style="color: #007AFF;font-weight: bold;">100</text>
				</div>
			</div>
		</div>
		<div class="func_box">
			<div v-for="item in funList" class="function_bar" @click="func(item)">
				<text class="funcText">{{ item }}</text>
			</div>
		</div>
		<div class="mask" v-if="isActive||isMask" :class="{'active':isMask}" @click="hidepopup()"></div>
		<div class="popup" :class="{'active':isActive,'input':isLogin===false}">
			<image src="/static/logo.png" style="border-radius: 10px;height: 90px;width: 90px;margin: 30px 0 10px;">
			</image>
			<text style="font-size: 20px;font-weight: bold;letter-spacing:1px;">文创商城</text>
			<button v-if="isLogin"
				style="border-radius: 28px;background-color: black;color: white;width: 90%;margin: 40px auto;"
				@click="getUserinfo()">微信一键登陆</button>
			<div v-if="isLogin===false" class="registerLayout">
				<div class="setbody">
					<text>昵称：</text>
					<input type="nickname" v-model="registerName" style="padding-left: 2px; " placeholder="请输入昵称" />
				</div>
				<button style="border-radius: 28px;background-color: black;color: white;width: 90%;margin: 30px auto;"
					@click="regist()">注册</button>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				funList:["退出账号"],
				username:null,
				isActive:'',
				isMask:'',
				isLogin:'true',
				registerName:'',
				openid:'',
			}
		},
		onLoad() {
			this.getStorageinfo()
		},
		methods: {
			func(item){
				if(item=='退出账号'){
					uni.showModal({
						title: '确定退出账号？',
						position: 'center',
						confirmColor:'#007AFF',
						success: function(res) {
							if (res.confirm) {
								uni.showLoading({
									title: '正在退出'
								})
								setTimeout(() => {
									uni.removeStorageSync('Storageuser')
									this.username = null
									this.openid=''
									uni.hideLoading()
								}, 500)
							}
						}.bind(this)
					})
					
				}
			},
			getStorageinfo() {
				var Storageuserinfo = uni.getStorageSync('Storageuser')
				this.openid = Storageuserinfo.id
				this.username = Storageuserinfo.name
			},
			showpopup() {
				this.isActive = true
				this.isMask = true
			},
			hidepopup() {
				this.isActive = false
				setTimeout(() => {
					this.isLogin = true
					this.registerName = ''
				}, 500)
				this.isMask = false
			},
			getUserinfo() {
				uni.showLoading({
					title: '登陆中'
				})
				setTimeout(() => {
					uni.hideLoading()
					if (this.username === '') {
						uni.showToast({
							title: '登陆超时',
							duration: 2000,
							icon: 'error'
						})
						this.hidepopup()
					}
				}, 5000);
				uni.login({
					"provider": "weixin",
					"onlyAuthorize": true,
					success: (res) => {
						uni.request({
							url: 'http://localhost:8080/wechat/code2Session',
							method: 'GET',
							header: {
								'content-type': 'application/json',
							},
							data: {
								code: res.code,
							},
							success: (res) => {
								console.log(res)
								this.userlogin(res.data.openid)
								uni.hideLoading()
							}
						})
					}
				})
			},
			userlogin(tempopenid){
				uni.request({
					url: 'http://localhost:8080/login',
					method: 'POST',
					header: {
						'content-type': 'application/json',
					},
					data: tempopenid,
					success: (res) => {
						if (res.data.error === '未找到数据') {
							this.isLogin = false
							this.openid=tempopenid
						} else {
							console.log(res.data.data.name)
							this.username = res.data.data.name
							let storageuserinfo = {
								"id": tempopenid,
								"name": this.username,
							}
							uni.setStorageSync("Storageuser", storageuserinfo)
							this.hidepopup()
						}
					}
				})
			},
			regist() {
				if (this.registerName === '') {
					uni.showToast({
						title: '请输入完整',
						duration: 2000,
						icon: 'none'
					})
				} else {
					let userinfo = {
						"id": this.openid,
						"name": this.registerName,
					}
					uni.request({
						url: 'http://localhost:8080/register',
						method: 'POST',
						header: {
							'content-type': 'application/json',
						},
						data: userinfo,
						success: (res) => {
							if (res.data === 'New record created successfully') {
								this.getUserinfo()
							} else {
								uni.showToast({
									title: '注册失败，请再试一下',
									duration: 2000,
									icon: 'none'
								})
							}
						}
					})
				}
			},
		}
	}
</script>

<style>
	.container {
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		align-items: left;
		justify-content: center;
		background: linear-gradient(135deg, #007aff 0%, #43a2f8 38%, #ffffff 100%);
	}

	.head {
		top: 0;
		display: flex;
		flex-direction: row;
		align-items: center;
		margin-left: 15px;
		margin-bottom: 15px;
		letter-spacing: 1px;
	}

	.head_box {
		margin: 14.2% 0 0 0;
		display: flex;
		flex-direction: column;
		align-items: left;
		justify-content: center;
	}

	.usercard {
		height: 150px;
		display: flex;
		flex-direction: column;
		background-color: white;
		border-radius: 18px;
		margin: 10px 10px;
	}

	.userfirst {
		width: 100%;
		height: 50%;
		display: flex;
		flex-direction: row;
		margin-left: 15px;
		align-items: center;
	}

	.usersecond {
		width: 100%;
		height: 50%;
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		align-items: center;
	}

	.line {
		width: 100%;
		height: 1px;
		background: linear-gradient(244deg, rgba(0, 122, 255, 0) 0%, rgba(0, 122, 255, 1) 50%, rgba(0, 122, 255, 0) 100%);
	}

	.userinfo {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.function_bar {
		background-color: white;  
		height: 50px;
		width: 100%;
		border-bottom: 0.5px solid grey;
		display: flex;
		align-items: center;
	}

	.funcText {
		font-size: 16px;
		margin: 0 15px;
	}
	.userAvatar {
		border-radius: 100%;
		width: 70px;
		height: 70px;
		margin-left: 20px;
		overflow: hidden;
	}
	
	.userNickName {
		display: flex;
		flex-direction: column;
		color: black;
		font-size: 20px;
		margin: 0 0 5px 25px;
	}
	.mask {
		position: fixed;
		height: 100%;
		width: 100%;
		background-color: grey;
		opacity: 0;
		bottom: 0;
		transition: opacity 0.5s ease-in-out;
	}
	
	.mask.active {
		opacity: 0.5;
	}
	
	.popup {
		position: fixed;
		display: flex;
		align-items: center;
		flex-direction: column;
		border-style: solid;
		box-shadow: #f1f1f1;
		border-color: #f1f1f1 #f1f1f1 #f8f8f8 #f1f1f1;
		border-radius: 24px 24px 0 0;
		bottom: 0;
		width: 100%;
		height: 280px;
		background-color: #f8f8f8;
		z-index: 999;
		transform: translateY(100%);
		transition: all 0.5s ease;
	}
	
	.popup.active {
		transform: translateY(0);
	}
	
	.popup.input {
		height: 400px;
	}
	
	.registerLayout {
		display: flex;
		flex-direction: column;
		margin-top: 20px;
	}
	
	.settingLayout {
		display: flex;
		flex-direction: column;
		margin-top: 5px;
	}
	
	.setbody {
		padding-left: 10px;
		padding-right: 10px;
		background-color: white;
		border-radius: 7px;
		height: 30px;
		margin-top: 10px;
		display: flex;
		flex-direction: row;
		font-size: 18px;
		align-items: center;
		justify-content: space-between;
	}
</style>