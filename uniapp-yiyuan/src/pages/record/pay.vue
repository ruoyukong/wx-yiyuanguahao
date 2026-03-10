<template>
  <view class="pay-record-container">
    <view class="page-header">支付记录</view>

    <!-- 无记录提示 -->
    <view v-if="!loading && payOrders.length === 0" class="empty-tip">
      <image src="/static/empty-pay.png" class="empty-img" mode="widthFix"></image>
      <view class="empty-text">暂无支付记录</view>
    </view>

    <!-- 支付记录列表 -->
    <view v-else-if="!loading" class="order-list">
      <view 
        class="pay-card" 
        v-for="(order, index) in payOrders" 
        :key="order.order_id"
        @click="toggleDetail(index)"
      >
        <!-- 简洁卡片信息（侧重支付） -->
        <view class="card-header">
          <view class="card-title">
            <text class="tag paid-tag">已支付</text>
            <text class="fee">¥{{ order.unit_price }}</text>
          </view>
          <view class="card-subtitle">
            <view class="info-item">就诊人：{{ order.real_name }}</view>
            <view class="info-item">预约门诊：{{ order.disease_name }}</view>
          </view>
          <view class="card-time">支付时间：{{ formatTime(order.create_time) }}</view>
        </view>

        <!-- 完整记录（点击展开，显示更多支付/订单信息） -->
        <view class="card-detail" v-if="order.show_detail">
          <view class="detail-item">
            <view class="detail-label">订单编号：</view>
            <view class="detail-value">{{ order.order_id }}</view>
          </view>
          <view class="detail-item">
            <view class="detail-label">就诊医生：</view>
            <view class="detail-value">{{ order.doctor_name }}</view>
          </view>
          <view class="detail-item">
            <view class="detail-label">就诊日期：</view>
            <view class="detail-value">{{ order.visit_date | formatYmdToShow }}</view>
          </view>
          <view class="detail-item">
            <view class="detail-label">就诊时段：</view>
            <view class="detail-value">{{ order.time_slot | formatTimeSlot }}</view>
          </view>
          <view class="detail-item">
            <view class="detail-label">支付状态：</view>
            <view class="detail-value">{{ order.pay_status_text }}</view>
          </view>
        </view>

        <!-- 展开/收起箭头 -->
        <view class="arrow" :class="order.show_detail ? 'up' : 'down'">
          <uni-icons type="arrow-down" size="16" v-if="!order.show_detail"></uni-icons>
          <uni-icons type="arrow-up" size="16" v-if="order.show_detail"></uni-icons>
        </view>
      </view>
    </view>

    <!-- 加载中 -->
    <view v-if="loading" class="loading-tip">
      <uni-loading type="circle" size="24"></uni-loading>
      <view class="loading-text">正在加载支付记录...</view>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'http://localhost/api';

export default {
  data() {
    return {
      loading: true,
      openid: '',
      payOrders: [] // 已支付订单列表
    };
  },
  onLoad() {
    // 获取用户openid
    this.openid = uni.getStorageSync('openid') || '';
    if (!this.openid) {
      uni.showToast({ title: '用户未登录，请先登录', icon: 'none' });
      setTimeout(() => uni.navigateTo({ url: '/pages/login/login' }), 1500);
      return;
    }
    // 加载已支付订单
    this.getPayOrders();
  },
  filters: {
    // 日期格式化（YYYY-MM-DD → 11月28日）
    formatYmdToShow(ymd) {
      if (!ymd) return '暂无';
      const [year, month, day] = ymd.split('-');
      return `${month}月${day}日`;
    },
    // 时段格式化（9-10 → 9:00-10:00）
    formatTimeSlot(timeSlot) {
      if (!timeSlot) return '';
      return timeSlot.replace(/(\d+)-(\d+)/, '$1:00-$2:00');
    }
  },
  methods: {
    // 获取已支付订单（筛选 pay_status=1）
    getPayOrders() {
      this.loading = true;
      uni.request({
        url: `${BASE_URL}/order/getByOpenid`,
        method: 'GET',
        data: { openid: this.openid },
        success: (res) => {
          this.loading = false;
          if (res.data.code === 200) {
            // 筛选已支付的订单
            this.payOrders = res.data.data
              .filter(order => order.pay_status === 1)
              .map(order => ({ ...order, show_detail: false })); // 初始隐藏详情
          } else {
            uni.showToast({ title: '获取支付记录失败', icon: 'none' });
          }
        },
        fail: () => {
          this.loading = false;
          uni.showToast({ title: '网络异常，请重试', icon: 'none' });
        }
      });
    },
    // 格式化时间（create_time → 2025-11-28 15:30:20）
    formatTime(timeStr) {
      if (!timeStr) return '暂无';
      const date = new Date(timeStr);
      return `${date.getFullYear()}-${(date.getMonth() + 1).toString().padStart(2, '0')}-${date.getDate().toString().padStart(2, '0')} ${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}:${date.getSeconds().toString().padStart(2, '0')}`;
    },
    // 展开/收起详情
    toggleDetail(index) {
      this.payOrders = this.payOrders.map((order, i) => {
        if (i === index) {
          return { ...order, show_detail: !order.show_detail };
        }
        return { ...order, show_detail: false }; // 关闭其他详情
      });
    }
  }
};
</script>

<style scoped>
.pay-record-container {
  padding: 16rpx;
  background-color: #f5f5f5;
  min-height: 100vh;
}
.page-header {
  font-size: 32rpx;
  font-weight: 600;
  margin-bottom: 20rpx;
  color: #333;
}
.empty-tip {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 200rpx;
}
.empty-img {
  width: 200rpx;
  margin-bottom: 20rpx;
}
.empty-text {
  font-size: 28rpx;
  color: #999;
}
.order-list {
  gap: 16rpx;
  display: flex;
  flex-direction: column;
}
.pay-card {
  background-color: #fff;
  border-radius: 16rpx;
  padding: 20rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
  position: relative;
}
.card-header {
  border-bottom: 1px solid #f0f0f0;
  padding-bottom: 16rpx;
  margin-bottom: 16rpx;
}
.card-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12rpx;
}
.paid-tag {
  font-size: 24rpx;
  color: #4cd964;
  background-color: #e8fff3;
  padding: 4rpx 12rpx;
  border-radius: 30rpx;
}
.fee {
  font-size: 32rpx;
  font-weight: 600;
  color: #ff5252;
}
.card-subtitle {
  display: flex;
  justify-content: space-between;
  font-size: 24rpx;
  color: #666;
  margin-bottom: 8rpx;
}
.card-time {
  font-size: 22rpx;
  color: #999;
}
.card-detail {
  margin-top: 16rpx;
  gap: 12rpx;
  display: flex;
  flex-direction: column;
}
.detail-item {
  display: flex;
  font-size: 24rpx;
  color: #666;
}
.detail-label {
  width: 160rpx;
  color: #999;
}
.arrow {
  position: absolute;
  right: 20rpx;
  bottom: 20rpx;
  color: #999;
}
.loading-tip {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 200rpx;
}
.loading-text {
  margin-top: 16rpx;
  font-size: 24rpx;
  color: #999;
}
</style>