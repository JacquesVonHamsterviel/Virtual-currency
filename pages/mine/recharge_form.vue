<template>
	<view class="">
		<view :class="{'light':theme == 'light'}">
			<view class="form-box">
				<view class="form-item">
					<view class="item-cont" style="border-bottom: 1rpx solid #E6E6E6;">
						<text>{{$t('chongzhi.type')}}</text>
						<input v-model="typename" placeholder="" :disabled="true" />
					</view>
					<view class="item-cont">
						<text>{{$t('chongzhi.address')}}</text>
						<input v-model="address" placeholder="" :disabled="true" />
						<view class="copys">
							<image src="/static/image/pay_ico.jpg" mode="widthFix"></image>
						</view>
					</view>
				</view>
			</view>
			<view class="form-box">
				<view class="form-item">
					<view class="item-cont" style="padding-bottom: 0;">
						<text>{{$t('chongzhi.amount')}}</text>
					</view>
					<view class="item-cont">
						<input v-model="number" placeholder="0"
							style="margin: 0;padding: 0 10rpx;height: 60rpx;background-color: #E6E6E6;border-radius: 10rpx;" />
				<!-- 		<text style="margin-left: 20rpx;"> USDT</text> -->
					</view>
				</view>
			</view>
			<view class="form-box">
				<view class="form-item">
					<view class="item-cont">
						<text>{{$t('chongzhi.upload')}}</text>
					</view>
					<view class="item-cont">
						<view v-if="imageurl" class="upload-box">
							<view class="imageurl">
								<image :src="imageurl" @longpress="deletes" mode=""></image>
							</view>
						</view>
						<view v-else class="upload-box" @click="upload">
							<view class="upload">
								<image src="/static/image/add.png" mode="widthFix"></image>
							</view>
						</view>
					</view>
				</view>
			</view>
			<view class="button-box">
				<button @click="submitClick()">{{$t('chongzhi.submit')}}</button>
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
				tid: null,
				typename: null,
				address: null,
				number: null,
				imageurl: null
			}
		},
		computed: {
			...mapState(['theme']),
		},
		onLoad(options) {
			if (options && options.id) {
				this.tid = options.id
			}
			this.getPayfrist()
		},
		methods: {
			getPayfrist() {
				var that = this
				this.$utils.initDataToken({
					url: 'payfrist',
					data: {
						id: that.tid
					},
					type: 'get'
				}, (res, msg) => {
					console.log('方式', res)
					that.typename = res.title
					that.address=res.address
				});
			},
			upload() {
				var that = this;
				uni.chooseImage({
					count: 1,
					sizeType: ['compressed'],
					success: (chooseImageRes) => {
						const tempFilePaths = chooseImageRes.tempFilePaths;
						uni.uploadFile({
							url: '/api/common/image_upload', //仅为示例，非真实的接口地址
							// #ifdef APP-PLUS
							url: domain + '/api/common/image_upload',
							// #endif
							filePath: tempFilePaths[0],
							name: 'file',
							formData: {
								'user': 'test'
							},
							success: (uploadFileRes) => {
								console.log(typeof uploadFileRes.data);
								var data = JSON.parse(uploadFileRes.data);
								if (data.code == 1) {
									that.imageurl = data.data.url
								}
							}
						});
					}
				});
			},
			deletes() {
				var that = this
				uni.showModal({
					title: this.$t('chongzhi.tishi'),
					content: this.$t('chongzhi.del'),
					success(res) {
						if (res.confirm) {
							that.imageurl = null
						}
					}
				});
			},
			submitClick() {
				// if (!this.typename) {
				// 	return this.$utils.showLayer("this.$t('chongzhi.p_name')")
				// }
				// if (!this.address) {
				// 	return this.$utils.showLayer("地址不能为空")
				// }
				if (!this.number) {
					return this.$utils.showLayer(this.$t('chongzhi.p_number'))
				}
				if (!this.imageurl) {
					return this.$utils.showLayer(this.$t('chongzhi.p_img'))
				}
				this.$utils.initDataToken({
					url: '/paysubmit',
					type: 'POST',
					data: {
						typename: this.typename,
						address: this.address,
						number: this.number,
						imageurl: this.imageurl
					}
				}, (res, msg) => {
					this.$utils.showLayer(msg);
					setTimeout(() => {
						uni.navigateBack({
							delta: 1
						})
					}, 1500)
				})
			}
		}
	}
</script>

<style lang="scss">
	.form-box {
		margin: 40rpx 30rpx;

		.form-item {
			padding: 0 30rpx;
			background-color: #FFFFFF;
			border-radius: 30rpx;

			.item-cont {
				padding: 30rpx 0;
				display: flex;
				align-items: center;

				text {
					font-size: 32rpx;
				}

				input {
					width: 450rpx;
					margin-left: 20rpx;
					font-size: 30rpx;
				}

				.copys {
					width: 32rpx;
					height: 32rpx;

					image {
						width: 100%;
						height: 100%;
					}
				}

				.upload-box {
					width: 100%;
					height: 300rpx;
					background-color: #E6E6E6;
					display: flex;
					justify-content: center;
					align-items: center;

					.imageurl {
						width: 100%;
						height: 100%;

						image {
							width: 100%;
							height: 100%;
						}
					}

					.upload {
						width: 88rpx;

						image {
							width: 100%;
							height: 100%;
						}
					}
				}
			}
		}
	}

	.button-box {
		margin-top: 100rpx;

		button {
			width: 80%;
			background-color: #680ED3;
			border-radius: 50rpx;
			font-size: 30rpx;
			color: #FFFFFF;
			margin: 0 auto;
		}
	}
</style>