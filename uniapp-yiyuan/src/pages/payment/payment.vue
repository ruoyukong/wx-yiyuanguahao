<template>
  <view class="payment-container">
    <view class="page-header">待缴费订单</view>

    <!-- 无订单提示 -->
    <view v-if="!loading && unpaidOrders.length === 0" class="empty-tip">
      <image src="/static/empty-order.png" class="empty-img" mode="widthFix"></image>
      <view class="empty-text">暂无需要缴费订单</view>
    </view>

    <!-- 订单列表 -->
    <view v-else-if="!loading" class="order-list">
      <view 
        class="order-card" 
        v-for="(order, index) in unpaidOrders" 
        :key="order.order_id"
        @click="showOrderDetail(index)"
      >
        <!-- 订单状态标签 -->
        <view class="order-tag" :class="order.is_expired ? 'expired-tag' : 'pending-tag'">
          {{ order.is_expired ? '已过期' : '待支付' }}
        </view>

        <!-- 订单核心信息 -->
        <view class="order-info">
          <view class="info-row">
            <view class="info-label">就诊医生：</view>
            <view class="info-value">{{ order.doctor_name }}</view>
          </view>
          <view class="info-row">
            <view class="info-label">就诊日期：</view>
            <view class="info-value">{{ order.visit_date | formatYmdToShow }}</view>
          </view>
          <view class="info-row">
            <view class="info-label">就诊时间：</view>
            <view class="info-value">{{ order.time_slot | formatTimeSlot }}</view>
          </view>
          <view class="info-row">
            <view class="info-label">预约门诊：</view>
            <view class="info-value">{{ order.disease_name }}</view>
          </view>
          <view class="info-row">
            <view class="info-label">缴费金额：</view>
            <view class="info-value fee">¥{{ order.unit_price }}</view>
          </view>
        </view>

        <!-- 倒计时/过期提示 -->
        <view class="countdown" v-if="!order.is_expired">
          剩余支付时间：{{ order.countdown }}
        </view>
        <view class="expired-text" v-else>
          订单已过期（超时2分钟未支付）
        </view>

        <!-- 支付按钮（未过期时显示） -->
        <button 
          class="pay-btn" 
          v-if="!order.is_expired && !order.show_detail"
          @click.stop="handlePay(order.order_id)"
        >
          立即支付
        </button>
      </view>
    </view>

    <!-- 加载中 -->
    <view v-if="loading" class="loading-tip">
      <uni-loading type="circle" size="24"></uni-loading>
      <view class="loading-text">正在加载订单...</view>
    </view>
  </view>
</template>

<script>
const BASE_URL = 'http://localhost/api'; // 后端基础地址
const PAYMENT_TIMEOUT = 2 * 60 * 1000; // 支付超时时间：2分钟（毫秒）

export default {
  data() {
    return {
      loading: true, // 加载状态
      openid: '', // 用户openid（从全局获取）
      unpaidOrders: [], // 未支付订单列表（含过期）
      countdownTimer: null // 倒计时定时器
    };
  },
  onLoad() {
    // 1. 获取全局存储的openid
    this.openid = uni.getStorageSync('openid') || '';
    if (!this.openid) {
      uni.showToast({ title: '用户未登录，请先登录', icon: 'none' });
      setTimeout(() => {
        uni.navigateTo({ url: '/pages/login/login' });
      }, 1500);
      return;
    }

    // 2. 加载未支付订单
    this.getUnpaidOrders();
  },
  onUnload() {
    // 页面销毁时清除定时器，避免内存泄漏
    if (this.countdownTimer) {
      clearInterval(this.countdownTimer);
    }
  },
  filters: {
    // 时间格式转换（如 9-10 → 9:00-10:00）
    formatTimeSlot(timeSlot) {
      if (!timeSlot) return '';
      return timeSlot.replace(/(\d+)-(\d+)/, '$1:00-$2:00');
    },
    // 新增：日期格式化（YYYY-MM-DD → 11月28日）
  formatYmdToShow(ymd) {
    if (!ymd) return '暂无'; // 兼容异常情况
    const [year, month, day] = ymd.split('-');
    return `${month}月${day}日`;
  }
  },
  methods: {
    /**
     * 调用后端接口，获取当前用户的未支付订单
     */
    getUnpaidOrders() {
      this.loading = true;
      uni.request({
        url: `${BASE_URL}/order/getByOpenid`,
        method: 'GET',
        data: { openid: this.openid },
        success: (res) => {
          this.loading = false;
          const { code, data } = res.data;
          if (code === 200) {
            // 筛选出未支付的订单（pay_status=0）
            const unpaid = data.filter(order =>order.pay_status === 0 || order.pay_status === 2);
            // 处理订单：添加过期状态和初始倒计时
            this.unpaidOrders = this.handleOrderStatus(unpaid);
            // 启动倒计时定时器
            this.startCountdown();
          } else {
            uni.showToast({ title: '获取订单失败', icon: 'none' });
          }
        },
        fail: (err) => {
          this.loading = false;
          console.error('获取未支付订单失败:', err);
          uni.showToast({ title: '网络异常，请重试', icon: 'none' });
        }
      });
    },

    /**
     * 处理订单状态：判断是否过期，计算初始倒计时
     * @param {Array} orders - 未支付订单列表
     * @returns {Array} 处理后的订单列表
     */
   handleOrderStatus(orders) {
    const now = Date.now(); // 当前时间戳（毫秒）
    return orders.map(order => {
      // 优先判断后端状态：pay_status=2 直接视为过期
      if (order.pay_status === 2) {
        return {
          ...order,
          is_expired: true,
          countdown: '',
          show_detail: false
        };
      }

      // 前端本地判断（针对 pay_status=0 的订单）
      const createTime = new Date(order.create_time).getTime();
      const elapsed = now - createTime;
      const isExpired = elapsed >= PAYMENT_TIMEOUT;
      let countdown = '';
      
      if (!isExpired) {
        const remaining = Math.ceil((PAYMENT_TIMEOUT - elapsed) / 1000);
        countdown = this.formatCountdown(remaining);
      }

      return {
        ...order,
        is_expired: isExpired,
        countdown: countdown,
        show_detail: false
      };
    });
  },

    /**
     * 格式化倒计时（秒 → 分:秒）
     * @param {number} seconds - 剩余秒数
     * @returns {string} 格式化后的倒计时（如 "01:30"）
     */
    formatCountdown(seconds) {
      const min = Math.floor(seconds / 60);
      const sec = seconds % 60;
      return `${min.toString().padStart(2, '0')}:${sec.toString().padStart(2, '0')}`;
    },

    /**
     * 启动倒计时定时器（每秒更新一次）
     */
    startCountdown() {
      // 清除已有定时器，避免重复
      if (this.countdownTimer) {
        clearInterval(this.countdownTimer);
      }

      this.countdownTimer = setInterval(() => {
        this.unpaidOrders = this.unpaidOrders.map(order => {
          if (order.is_expired) return order; // 已过期订单不更新

          // 重新计算剩余时间
          const now = Date.now();
          const createTime = new Date(order.create_time).getTime();
          const elapsed = now - createTime;
          const remaining = Math.ceil((PAYMENT_TIMEOUT - elapsed) / 1000);

          // 检查是否过期
          if (remaining <= 0) {
            return { ...order, is_expired: true, countdown: '' };
          }

          // 更新倒计时
          return { ...order, countdown: this.formatCountdown(remaining) };
        });
      }, 1000);
    },

    /**
     * 显示/隐藏订单完整详情
     * @param {number} index - 订单索引
     */
    showOrderDetail(index) {
      this.unpaidOrders = this.unpaidOrders.map((order, i) => {
        if (i === index) {
          return { ...order, show_detail: !order.show_detail };
        }
        return { ...order, show_detail: false }; // 关闭其他订单详情
      });
    },

    /**
     * 立即支付（调用后端更新支付状态接口）
     * @param {string} orderId - 订单ID
     */
    handlePay(orderId) {
      // 显示支付确认弹窗
      this.showPaymentConfirmModal(orderId);
    },

    /**
     * 显示支付确认弹窗
     * @param {string} orderId - 订单ID
     */
    showPaymentConfirmModal(orderId) {
      const order = this.unpaidOrders.find(item => item.order_id === orderId);
      if (!order) return;

      uni.showModal({
        title: '支付确认',
        content: `请支付预约费用：¥${order.unit_price}`,
        confirmText: '确认支付',
        cancelText: '取消支付',
        success: (res) => {
          if (res.confirm) {
            this.processPayment(orderId); // 处理支付
          } else {
            // 取消支付：显示提示弹窗
            this.showCancelPaymentModal();
          }
        }
      });
    },

    /**
     * 处理支付
     * @param {string} orderId - 订单ID
     */
    processPayment(orderId) {
      uni.showLoading({ title: '正在处理支付...' });

      // 模拟真实支付流程
      setTimeout(() => {
        // 支付成功后，调用后端更新支付状态
        this.updatePayStatus(orderId, 1);
      }, 1500);
    },

    /**
     * 调用后端接口更新支付状态
     * @param {string} orderId - 订单ID
     * @param {number} payStatus - 支付状态（1=已支付）
     */
    updatePayStatus(orderId, payStatus) {
      uni.request({
        url: `${BASE_URL}/order/updatePayStatus`,
        method: 'POST',
        data: { order_id: orderId, pay_status: payStatus },
        success: (res) => {
          uni.hideLoading();
          const { code, msg } = res.data;
          if (code === 200) {
            // 支付成功，显示弹窗
            this.showPaySuccessModal();
            // 移除已支付的订单（刷新列表）
            this.unpaidOrders = this.unpaidOrders.filter(order => order.order_id !== orderId);
          } else {
            uni.showToast({ title: msg || '支付失败', icon: 'none' });
          }
        },
        fail: (err) => {
          uni.hideLoading();
          console.error('更新支付状态失败:', err);
          uni.showToast({ title: '网络异常，支付失败', icon: 'none' });
        }
      });
    },

    /**
     * 支付成功弹窗
     */
    showPaySuccessModal() {
      uni.showModal({
        title: '支付成功',
        content: '支付成功，预约信息可在"我的"界面"挂号记录"查看',
        showCancel: false,
        confirmText: '确定',
        success: (res) => {
          if (res.confirm) {
            // 用户点击确定，不需要额外操作
          }
        }
      });
    },

    /**
     * 取消支付弹窗
     */
    showCancelPaymentModal() {
      uni.showModal({
        title: '支付取消',
        content: '在两分钟内可在缴费处支付',
        showCancel: false,
        confirmText: '确定',
        success: (res) => {
          if (res.confirm) {
            // 用户点击确定，不需要额外操作
          }
        }
      });
    }
  }
};
</script>

<style scoped>
/* 页面整体样式 */
.payment-container {
  background-color: #f5f5f5;
  min-height: 100vh;
  padding-bottom: 30rpx;
}

/* 页面标题 */
.page-header {
  font-size: 32rpx;
  font-weight: 500;
  text-align: center;
  padding: 30rpx 0;
  background-color: #fff;
  border-bottom: 1rpx solid #eee;
}

/* 无订单提示 */
.empty-tip {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 200rpx;
}
.empty-img {
  width: 200rpx;
  height: auto;
  margin-bottom: 30rpx;
}
.empty-text {
  font-size: 28rpx;
  color: #999;
}

/* 订单列表 */
.order-list {
  padding: 20rpx;
}

/* 订单卡片 */
.order-card {
  background-color: #fff;
  border-radius: 20rpx;
  padding: 25rpx;
  margin-bottom: 20rpx;
  position: relative;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

/* 订单状态标签 */
.order-tag {
  position: absolute;
  top: 25rpx;
  right: 25rpx;
  padding: 8rpx 15rpx;
  border-radius: 20rpx;
  font-size: 22rpx;
  font-weight: 500;
}
.pending-tag {
  background-color: #e8f4ff;
  color: #007aff;
}
.expired-tag {
  background-color: #fef0f0;
  color: #ff4d4f;
}

/* 订单信息 */
.order-info {
  margin-bottom: 20rpx;
}
.info-row {
  display: flex;
  margin-bottom: 15rpx;
  font-size: 26rpx;
}
.info-row:last-child {
  margin-bottom: 0;
}
.info-label {
  flex: 0 0 180rpx;
  color: #666;
}
.info-value {
  flex: 1;
  color: #333;
}
.fee {
  color: #ff5722;
  font-weight: bold;
  font-size: 28rpx;
}

/* 倒计时 */
.countdown {
  font-size: 24rpx;
  color: #ff4d4f;
  margin-bottom: 20rpx;
  padding: 10rpx;
  background-color: #fef0f0;
  border-radius: 10rpx;
}

/* 过期提示 */
.expired-text {
  font-size: 24rpx;
  color: #ff4d4f;
  margin-bottom: 20rpx;
}

/* 支付按钮 */
.pay-btn {
  width: 100%;
  height: 80rpx;
  line-height: 80rpx;
  background-color: #007aff;
  color: #fff;
  font-size: 28rpx;
  border-radius: 15rpx;
}

/* 加载中提示 */
.loading-tip {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 200rpx;
}
.loading-text {
  font-size: 26rpx;
  color: #999;
  margin-top: 20rpx;
}
</style>