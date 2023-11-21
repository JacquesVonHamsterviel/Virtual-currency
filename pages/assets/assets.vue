<template>
	<view class="">
		<view class="status_bar">
			<view class="top_view" :class="tabBg"></view>
		</view>
		<view class="header fixed bdb1f flex alcenter jscenter blue " :class="tabBg" id="tab-bar"
			:scroll-left="scrollLeft">
			<text :class="{'white':i==tabIndex}" class="plr10 ft12 bold" v-for="(tab,i) in wallet_data" :key="i"
				:id="'type'+i" :data-current="i" @click="tapTab">{{tab.name}}</text>
		</view>
		<view :class="{'light':theme=='light'}">
			<view class="head-box">
				<view class="head-icon">
					<image src="/static/image/head.png" mode=""></image>
				</view>
				<view class="head-name">{{ info.email }}</view>
				<view class="head-vips">VIP1</view>
			</view>
			<view class="middle-box">
				<view class="middle-item">
					<view class="item-number">Balance(USDT) {{ info.balance }}</view>
					<view class="item-name" @click="chongzhi()">{{$t('my.charge')}}</view>
				</view>
				<view class="middle-item">
					<view class="item-number">Balance(EUR) {{ info.rub_balance }}</view>
					<view class="item-name" @click="tixian()">{{$t('my.withdraw')}}</view>
				</view>
			</view>
			<view class="foot-box">
				<view class="foot-list" @click="footClick('team')">
					<view class="list-name">
						<image src="/static/image/shops.png" mode=""></image>
						<text>Team List</text>
					</view>
					<view class="list-icon">
						<image src="/static/image/arrrowr.png" mode=""></image>
					</view>
				</view>
				<view class="foot-list" @click="footClick('sfrz')">
					<view class="list-name">
						<image src="/static/image/personal.png" mode=""></image>
						<text>{{$t('authentication.renzheng')}}</text>
					</view>
					<view class="list-icon">
						<image src="/static/image/arrrowr.png" mode=""></image>
					</view>
				</view>
				<view class="foot-list" @click="footClick('skfs')">
					<view class="list-name">
						<image src="/static/image/loss.png" mode=""></image>
						<text>{{$t('collect.method')}}</text>
					</view>
					<view class="list-icon">
						<image src="/static/image/arrrowr.png" mode=""></image>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		mapState
	} from 'vuex'
	export default {
		data() {
			return {
				info: '',
				showtixian: true,
				scrollLeft: 0,
				isClickChange: false,
				tabIndex: 0,
				wallet_data: [],
				isClose: false,
				tabBg: 'bg1',
				closeAccount: '****',
				sign: 'USD',
				sign2: '$',
			}
		},
		onLoad() {
			this.getUserInfo();
		},
		onPullDownRefresh() {
			this.getAssets();
		},
		computed: {
			...mapState(['theme']),
		},
		onShow() {
			this.$utils.setThemeTop(this.theme)
			this.$utils.setThemeBottom(this.theme)
			this.getAssets();
		},
		methods: {
			//获取用户信息
			getUserInfo() {
				var that = this;
				this.$utils.initDataToken({
					url: 'user/info',
					data: {},
					type: 'get'
				}, (res, msg) => {
					uni.stopPullDownRefresh();
					uni.setStorageSync('uid', msg.id);
					that.info = res;
				});
			},
			// 充值
			chongzhi() {
				uni.navigateTo({
					url: '/pages/mine/recharge'
				})
			},
			// 提现
			tixian() {
				uni.navigateTo({
					url: '/pages/mine/withdrawal'
				})
			},
			// 点击跳转
			footClick(type) {
				if (type == 'team') {
					uni.showToast({
						title: '暂未开放',
						duration: 2000,
						icon: 'none'
					});
				} else if (type == 'sfrz') {
					uni.navigateTo({
						url: '/pages/mine/real_authentication'
					})
				} else if (type == 'skfs') {
					uni.navigateTo({
						url: '/pages/mine/collect'
					})
				}
			},
			close(index1, index2) {
				uni.showModal({
					content: '是否删除本条信息？',
					success: (res) => {
						if (res.confirm) {
							this.newsitems[index1].data.splice(index2, 1);
						}
					}
				})
			},
			async changeTab(e) {
				let index = e.target.current;
				this.tabBg = 'bg' + (index + 1)
				if (this.isClickChange) {
					this.tabIndex = index;
					this.isClickChange = false;
					return;
				}
				let tabBar = await this.getElSize("tab-bar"),
					tabBarScrollLeft = tabBar.scrollLeft;
				let width = 0;

				for (let i = 0; i < index; i++) {
					// console.log(index,'type'+i);
					let result = await this.getElSize('type' + i);
					width += result.width;
				}
				// console.log('type'+index)
				let winWidth = uni.getSystemInfoSync().windowWidth,
					nowElement = await this.getElSize('type' + index),
					nowWidth = nowElement.width;
				if (width + nowWidth - tabBarScrollLeft > winWidth) {
					this.scrollLeft = width + nowWidth - winWidth;
				}
				if (width < tabBarScrollLeft) {
					this.scrollLeft = width;
				}
				this.isClickChange = false;
				this.tabIndex = index; //一旦访问data就会出问题
			},
			getElSize(id) { //得到元素的size
				return new Promise((res, rej) => {
					uni.createSelectorQuery().select("#" + id).fields({
						size: true,
						scrollOffset: true
					}, (data) => {
						res(data);
					}).exec();
				})
			},
			async tapTab(e) { //点击tab-bar
				let tabIndex = e.target.dataset.current;
				if (this.tabIndex === tabIndex) {
					return false;
				} else {
					let tabBar = await this.getElSize("tab-bar"),
						tabBarScrollLeft = tabBar.scrollLeft; //点击的时候记录并设置scrollLeft
					this.scrollLeft = tabBarScrollLeft;
					this.isClickChange = true;
					this.tabIndex = tabIndex;
				}
				this.tabBg = 'bg' + (tabIndex + 1)
			},
			getAssets() {
				this.$utils.initDataToken({
					url: 'account/list'
				}, res => {
					console.log(res)
					uni.stopPullDownRefresh()
					// this.sign= res.quote_symbol;
					// this.sign2= res.quote_symbol_2;
					this.wallet_data = res;
				})
			}
		}
	}
</script>

<style lang="scss">
	.head-box {
		padding: 50rpx 30rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 55px;

		.head-icon {
			width: 150rpx;
			height: 150rpx;

			image {
				width: 100%;
				height: 100%;
				border-radius: 50%;
			}
		}

		.head-name {
			margin-top: 30rpx;
			font-size: 32rpx;
		}

		.head-vips {
			margin-top: 20rpx;
			font-size: 28rpx;
		}
	}

	.middle-box {
		padding: 20rpx 30rpx;
		background-color: #FFFFFF;
		border-radius: 20rpx;
		margin: 0 30rpx;

		.middle-item {
			padding: 20rpx 0;
			display: flex;
			justify-content: space-between;
			align-items: center;

			.item-number {
				font-size: 32rpx;
			}

			.item-name {
				padding: 8rpx 28rpx;
				border: 2rpx solid #999999;
				border-radius: 8rpx;
				font-size: 26rpx;
			}
		}
	}

	.foot-box {
		margin: 30rpx;

		.foot-list {
			padding: 30rpx 20rpx;
			background-color: #FFFFFF;
			display: flex;
			justify-content: space-between;
			align-items: center;
			margin-bottom: 20rpx;

			.list-name {
				display: flex;
				align-items: center;

				image {
					width: 44rpx;
					height: 44rpx;
					vertical-align: middle;
				}

				text {
					margin-left: 20rpx;
					font-size: 30rpx;
				}
			}

			.list-icon {
				width: 32rpx;
				height: 32rpx;

				image {
					width: 100%;
					height: 100%;
				}
			}
		}
	}

	.bg1 {
		background-color: #0f2441 !important;
	}

	.bg2 {
		background-color: #15376f !important;
	}

	.bg3 {
		background-color: #37365d !important;
	}

	.bg4 {
		background-color: #1d1842 !important;
	}

	.bg5 {
		background-color: #004150 !important;
	}

	.light .bg1 {
		background-color: #0f2441 !important;
	}

	.light .bg2 {
		background-color: #15376f !important;
	}

	.light .bg3 {
		background-color: #37365d !important;
	}

	.light .bg4 {
		background-color: #1d1842 !important;
	}

	.light .bg5 {
		background-color: #004150 !important;
	}

	.myht100 {
		/* height: 100.0vh; */
		overflow-y: scroll;
		min-height: 100.0vh !important;
	}

	/* uni-swiper .uni-swiper-wrapper{
		overflow-y: scroll !important; 
	} */
	swiper-item {
		overflow-y: scroll !important;
	}
</style>