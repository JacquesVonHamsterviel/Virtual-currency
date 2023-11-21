<template>
  <view class="">
    <view :class="{ light: theme == 'light' }">
      <view class="recharge-box">
        <block v-for="(item, index) in rechargelist" :key="index">
          <view class="recharge-list" @click="rechargeClick(item.id)">
            <text>{{ item.title }}</text>
            <image src="/static/image/arrrowr.png" mode=""></image>
          </view>
          <!-- 					<view class="recharge-list" @click="rechargeClick('ETH')">
						<text>ETH</text>
						<image src="/static/image/arrrowr.png" mode=""></image>
					</view>
					<view class="recharge-list" @click="rechargeClick('BTC')">
						<text>BTC</text>
						<image src="/static/image/arrrowr.png" mode=""></image>
					</view> -->
        </block>
      </view>
    </view>
  </view>
</template>

<script>
import { mapState } from "vuex";

export default {
  data() {
    return {
      rechargelist: [],
    };
  },

  computed: {
    ...mapState(["theme"]),
  },
  onShow() {
    this.getRecharge();
  },
  onNavigationBarButtonTap(e) {
    uni.navigateTo({
      url: "/pages/mine/rechargeList",
    });
  },
  methods: {
    getRecharge() {
      var that = this;
      this.$utils.initDataToken(
        {
          url: "paylist",
          data: {},
          type: "get",
        },
        (res, msg) => {
          console.log("充值方式", res);
          that.rechargelist = res.data;
        }
      );
    },
    // 点击跳转
    rechargeClick(type) {
      console.log("--------------" + type);
      // if (type == 'USDT') {
      uni.navigateTo({
        url: "/pages/mine/recharge_form?id=" + type,
      });
      // } else if (type == 'ETH') {
      // 	uni.navigateTo({
      // 		url: '/pages/mine/recharge_form?id=2'
      // 	})
      // } else if (type == 'BTC') {
      // 	uni.navigateTo({
      // 		url: '/pages/mine/recharge_form?id=3'
      // 	})
      // }
    },
  },
};
</script>

<style lang="scss">
.recharge-box {
  padding: 30rpx 0 0;

  .recharge-list {
    padding: 30rpx;
    background-color: #ffffff;
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 30rpx;

    text {
      font-size: 32rpx;
    }

    image {
      width: 32rpx;
      height: 32rpx;
      vertical-align: middle;
    }
  }
}
</style>
