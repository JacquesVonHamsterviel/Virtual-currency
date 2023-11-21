<template>
  <view class="">
    <view :class="{ light: theme == 'light' }">
      <view class="recharge-box">
        <block v-for="(item, index) in rechargelist" :key="index">
          <view class="recharge-list">
            <view class="left">
              <view class="number">
                {{ item.number }}
              </view>
              <view class="time">{{ item.created_at }}</view>
            </view>
            <view>{{ $t(`rechargeList.${status[item.status]}`) }}</view>
          </view>
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
      status: ["unaudited", "adopt", "notPassed"],
      page: 1,
      hasMore: true,
    };
  },
  computed: {
    ...mapState(["theme"]),
  },
  onShow() {
    this.getData();
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
    getData() {
      var that = this;
      this.$utils.initDataToken(
        {
          url: "paychangelist",
          data: { page: that.page },
          type: "get",
        },
        (res, msg) => {
          var data = res.data;
          uni.stopPullDownRefresh();
          that.rechargelist = that.page == 1 ? data : that.rechargelist.concat(data);
          that.hasMore = res.current_page == res.last_page ? false : true;
        }
      );
    },
  },
};
</script>

<style lang="scss" scoped>
.recharge-box {
  padding: 30rpx 0 0;

  .recharge-list {
    padding: 30rpx;
    background-color: #ffffff;
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 30rpx;
  }
  .number {
    font-size: 32rpx;
  }
  .time {
    margin-top: 10rpx;
    color: #7e7a7a;
  }
}
</style>
