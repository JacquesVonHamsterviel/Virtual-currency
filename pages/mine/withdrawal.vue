<template>
	<view class="">
		<view :class="{ light: theme == 'light' }">
			<view class="tixian-head">
				<view class="title">EUR</view>
				<view class="head-box">{{ $t("withdraw.remaining") }}:{{ info.rub_balance }}</view>
				<input type="number" v-model="number" class="h40 lh40 input" @input="handleInput" />
				<view class="mt20 bgBlue radius4 ptb10 white ft14 tc mb10" @tap="submit">{{
          $t("withdraw.submit")
        }}</view>
			</view>
			<view class="tixian-list">
				<!-- <view class="title">{{ $t("assets.submit") }}</view> -->
				<view class="list-item" v-for="(item, index) in tixianlist" :key="index">
					<view class="item-box">
						<view class="item-number">{{ $t("withdraw.quantity") }}:{{ item.number }}</view>
						<view class="item-content">
							{{ $t("withdraw.status") }}:{{ $t(`withdraw.${status[item.status]}`) }}
						</view>
					</view>
					<view class="item-time">{{ item.updated_at }}</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		mapState
	} from "vuex";

	export default {
		data() {
			return {
				tixianlist: [],
				status: ['', "pendingReview", "adopt", "rejected"],
				number: 0,
				info: {},
				page: 1,
				hasMore: true,
			};
		},
		computed: {
			...mapState(["theme"]),
		},
		onShow() {
			this.getData();
			this.getUserInfo();
		},
		onPullDownRefresh() {
			this.page = 1;
			this.getData();
		},
		onReachBottom() {
			if (!this.hasMore) return;
			this.page++;
			this.getData();
		},

		methods: {
			handleInput() {
				if (parseFloat(this.number) > parseFloat(this.info.rub_balance)) {
					this.$nextTick(() => {
						this.number = this.info.rub_balance;
					});
				}
			},
			submit() {
				if (this.number > 0) {

					this.$utils.initDataToken({
							url: "submitWithdraw",
							data: {
								number: this.number,
							},
						},
						(res) => {
							this.page = 1;
							this.number = 0;
							uni.showToast({
								icon: 'success',
								title: this.$t('withdraw.success')
							})
							setTimeout(() => {
								this.getUserInfo();
								this.getData();
							}, 1000)
						}
					);
				} else {
					uni.showToast({
						icon: 'error',
						title: this.$t('withdraw.not0')
					})
				}
			},
			getData() {
				let that = this;
				that.$utils.initDataToken({
						url: "withdrawlist",
						data: {
							page: that.page,
						},
					},
					(res) => {
						var data = res.data;
						uni.stopPullDownRefresh();
						that.tixianlist = that.page == 1 ? data : that.tixianlist.concat(data);
						that.hasMore = res.current_page == res.last_page ? false : true;
					}
				);
			},
			getUserInfo() {
				var that = this;
				this.$utils.initDataToken({
						url: "user/info",
						data: {},
						type: "get",
					},
					(res, msg) => {
						that.info = res;
					}
				);
			},
		},
	};
</script>

<style lang="scss">
	.tixian-head {
		padding: 30rpx;
		background-color: #ffffff;

		.input {
			border-bottom: 1px solid #e1e1e1;
		}

		.title {
			font-size: 36rpx;
			color: #680ed3;
		}

		.head-box {
			padding-top: 30rpx;
			display: flex;
			align-items: center;
			font-size: 28rpx;
		}
	}

	.tixian-list {
		margin-top: 30rpx;
		padding: 0 30rpx;
		background-color: #ffffff;

		.title {
			padding: 30rpx 0;
			font-size: 36rpx;
			color: #680ed3;
		}

		.list-item {
			padding: 15rpx;
			border-top: 2rpx solid #e6e6e6;

			.item-box {
				display: flex;
				justify-content: space-between;
				align-items: center;
				flex-wrap: nowrap;

				.item-number {
					font-size: 30rpx;
				}

				.item-content {
					font-size: 28rpx;
				}
			}

			.item-time {
				margin-top: 20rpx;
				font-size: 26rpx;
			}
		}
	}
</style>