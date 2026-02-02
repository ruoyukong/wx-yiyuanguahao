<template>
  <view class="container">
    <!-- 第一个医院：dgut市第一人民医院 -->
    <view class="hospital-card">
      <!-- 图片容器固定在顶部 -->
      <view class="image-container">
        <image src="/static/yiyuan2.jpg" mode="widthFix" class="hospital-img"></image>
        <view class="image-overlay"></view>
      </view>
      
      <!-- 内容容器，通过 translateY 实现滑动覆盖效果 -->
      <view
        class="content-container"
        :style="{ transform: `translateY(${offsetY}px)` }"
        @touchstart="onTouchStart"
        @touchmove="onTouchMove"
        @touchend="onTouchEnd"
      >
        <view class="hospital-header">
          <view class="hospital-name">dgut市第一人民医院</view>
          <view class="hospital-level">三级甲等 · 综合医院</view>
          <view class="hospital-tag">
            <text class="tag">医保定点</text>
            <text class="tag">三甲医院</text>
          </view>
        </view>

        <view class="hospital-info">
          <view class="info-item">
            <view class="icon-box">
              <text class="icon">📋</text>
            </view>
            <view class="info-content">
              <text class="label">医院简介：</text>
              <text class="value">dgut市第一人民医院始建于1950年，是一所集医疗、教学、科研、预防为一体的三级甲等综合医院，开放床位1500张，年门诊量超200万人次。</text>
            </view>
          </view>
          <view class="info-item">
            <view class="icon-box">
              <text class="icon">📍</text>
            </view>
            <view class="info-content">
              <text class="label">地址：</text>
              <text class="value">dgut市gy区18路510号</text>
            </view>
          </view>
          <view class="info-item">
            <view class="icon-box">
              <text class="icon">📞</text>
            </view>
            <view class="info-content">
              <text class="label">联系电话：</text>
              <text class="value">066-34567890</text>
            </view>
          </view>
          <view class="info-item">
            <view class="icon-box">
              <text class="icon">🕒</text>
            </view>
            <view class="info-content">
              <text class="label">营业时间：</text>
              <text class="value">周一至周日 08:00-18:30（急诊科24小时）</text>
            </view>
          </view>
        </view>
        
        <view class="action-buttons">
          <button class="btn btn-primary" @click="navigateTo('/pages/register/register')">预约挂号</button>
        </view>
      </view>
    </view>

    <!-- 第二个医院：dgut市中医院 -->
    <view class="hospital-card">
      <!-- 图片容器固定在顶部 -->
      <view class="image-container">
        <image src="/static/yiyuan1.jpg" mode="widthFix" class="hospital-img"></image>
        <view class="image-overlay"></view>
      </view>
      
      <!-- 内容容器，通过 translateY 实现滑动覆盖效果 -->
      <view
        class="content-container"
        :style="{ transform: `translateY(${offsetY_2}px)` }"
        @touchstart="(e) => onTouchStart(e, 2)"
        @touchmove="(e) => onTouchMove(e, 2)"
        @touchend="(e) => onTouchEnd(e, 2)"
      >
        <view class="hospital-header">
          <view class="hospital-name">dgut市中医院</view>
          <view class="hospital-level">三甲 · 中医医院</view>
          <view class="hospital-tag">
            <text class="tag">医保定点</text>
            <text class="tag">三甲医院</text>
            <text class="tag">中医特色</text>
          </view>
        </view>

        <view class="hospital-info">
          <view class="info-item">
            <view class="icon-box">
              <text class="icon">📋</text>
            </view>
            <view class="info-content">
              <text class="label">医院简介：</text>
              <text class="value">dgut市中医院始建于1965年，是集医疗、教学、科研、预防保健、康复于一体的三级甲等中医医院，是全国示范中医医院，开放床位1200张，年门诊量超150万人次。</text>
            </view>
          </view>
          <view class="info-item">
            <view class="icon-box">
              <text class="icon">📍</text>
            </view>
            <view class="info-content">
              <text class="label">地址：</text>
              <text class="value">dgut市gx区4路200号</text>
            </view>
          </view>
          <view class="info-item">
            <view class="icon-box">
              <text class="icon">📞</text>
            </view>
            <view class="info-content">
              <text class="label">联系电话：</text>
              <text class="value">066-87654321</text>
            </view>
          </view>
          <view class="info-item">
            <view class="icon-box">
              <text class="icon">🕒</text>
            </view>
            <view class="info-content">
              <text class="label">营业时间：</text>
              <text class="value">周一至周六 08:00-17:30，周日 08:30-16:30（中医急诊科24小时）</text>
            </view>
          </view>
        </view>
        
        <view class="action-buttons">
          <button class="btn btn-primary" @click="navigateTo('/pages/register/register')">预约挂号</button>
         
        </view>
      </view>
    </view>
  </view>  
</template>

<script>
export default {
  data() {
    return {
      startY: 0,
      offsetY: 0, // 当前滑动偏移量
      maxOffset: 0, // 最大上滑距离（负值）
      minOffset: 0, // 最大下滑距离（0）
      startY_2: 0,
      offsetY_2: 0,
      maxOffset_2: 0,
      minOffset_2: 0,
    };
  },
  onLoad() {
    const systemInfo = uni.getSystemInfoSync();
    const rpxToPx = systemInfo.windowWidth / 750;
    const imageHeight = 600 * rpxToPx; // 600rpx 转 px
    this.maxOffset = -(imageHeight * 2 / 3); // 最多上滑 2/3
    this.maxOffset_2 = -(imageHeight * 2 / 3);
  },
  methods: {
    onTouchStart(e, index = 1) {
      if (index === 1) {
        this.startY = e.touches[0].clientY;
      } else {
        this.startY_2 = e.touches[0].clientY;
      }
    },
    onTouchMove(e, index = 1) {
      if (index === 1) {
        const deltaY = e.touches[0].clientY - this.startY;
        let newOffset = this.offsetY + deltaY;

        // 限制范围
        if (newOffset > this.minOffset) newOffset = this.minOffset;
        if (newOffset < this.maxOffset) newOffset = this.maxOffset;

        this.offsetY = newOffset;
        this.startY = e.touches[0].clientY;
      } else {
        const deltaY = e.touches[0].clientY - this.startY_2;
        let newOffset = this.offsetY_2 + deltaY;

        // 限制范围
        if (newOffset > this.minOffset_2) newOffset = this.minOffset_2;
        if (newOffset < this.maxOffset_2) newOffset = this.maxOffset_2;

        this.offsetY_2 = newOffset;
        this.startY_2 = e.touches[0].clientY;
      }
    },
        checkUserInfo() {
      const userInfo = uni.getStorageSync('userInfo');
      // 检查缓存中是否存在 userInfo，并且 nickName 和 avatarUrl 不为空
      if (userInfo && userInfo.nickName && userInfo.avatarUrl) {
        return true;
      }
      return false;
    },

    onTouchEnd(e, index = 1) {
      // 可选：惯性滑动或回弹动画
    },
   navigateTo(targetUrl) {
      // 1. 先检查用户信息
      if (!this.checkUserInfo()) {
        // 2. 如果信息不完整，弹出提示
        uni.showModal({
          title: '提示',
          content: '请先在“我的”页面授权您的个人信息',
          showCancel: false, // 不显示取消按钮，强制用户前往
          confirmText: '前往授权',
          success: (res) => {
            if (res.confirm) {
              // 3. 用户点击确认后，跳转到“我的”页面
              // 注意：请确保 '/pages/mine/mine' 是你“我的”页面的正确路径
              uni.switchTab({
                url: '/pages/mine/mine' 
              });
            }
          }
        });
        // 4. 终止后续的跳转操作
        return;
      }

      // 5. 如果信息完整，则执行原来的跳转逻辑
      uni.navigateTo({
        url: targetUrl,
        success: () => {
          console.log(`跳转至 ${targetUrl} 成功`);
        },
        fail: (err) => {
          console.error('跳转失败：', err);
          uni.redirectTo({ url: targetUrl });
        }
      });
    }
  }
};

</script>

<style scoped>
.container {
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e8f0 100%);
  min-height: 100vh;
  padding: 20rpx;
  box-sizing: border-box;
}

.hospital-card {
  background-color: #fff;
  border-radius: 24rpx;
  box-shadow: 0 10rpx 30rpx rgba(0, 0, 0, 0.08);
  margin-bottom: 40rpx;
  overflow: hidden;
  position: relative;
}

/* 图片容器固定在顶部 */
.image-container {
  position: relative;
  width: 100%;
  height: 600rpx;
  overflow: hidden;
}

.hospital-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.image-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 120rpx;
  background: linear-gradient(to top, rgba(255,255,255,0.9) 0%, rgba(255,255,255,0) 100%);
}

/* 内容容器，初始位置在图片下方 */
.content-container {
  position: relative;
  z-index: 2;
  transition: transform 0.1s ease-out;
  background-color: #fff;
  border-radius: 24rpx 24rpx 0 0;
  margin-top: -40rpx;
}

.hospital-header {
  background-color: #fff;
  padding: 40rpx 30rpx 30rpx;
  border-radius: 24rpx 24rpx 0 0;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.03);
}

.hospital-name {
  font-size: 44rpx;
  font-weight: bold;
  margin-bottom: 12rpx;
  color: #1a1a1a;
  letter-spacing: 1rpx;
}

.hospital-level {
  font-size: 32rpx;
  color: #4a90e2;
  margin-bottom: 20rpx;
  font-weight: 500;
}

.hospital-tag {
  display: flex;
  flex-wrap: wrap;
  gap: 16rpx;
}

.tag {
  font-size: 24rpx;
  color: #4a90e2;
  background-color: rgba(74, 144, 226, 0.1);
  padding: 8rpx 20rpx;
  border-radius: 40rpx;
  border: 1rpx solid rgba(74, 144, 226, 0.2);
}

.hospital-info {
  background-color: #fff;
  padding: 0 30rpx 30rpx;
}

.info-item {
  margin-bottom: 40rpx;
  display: flex;
  align-items: flex-start;
  padding: 20rpx 0;
  border-bottom: 1rpx solid #f0f0f0;
}

.info-item:last-child {
  border-bottom: none;
}

.icon-box {
  width: 60rpx;
  height: 60rpx;
  border-radius: 50%;
  background-color: rgba(74, 144, 226, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 24rpx;
  flex-shrink: 0;
}

.icon {
  font-size: 32rpx;
}

.info-content {
  flex: 1;
}

.label {
  font-size: 32rpx;
  color: #333;
  font-weight: 500;
  display: block;
  margin-bottom: 12rpx;
}

.value {
  font-size: 30rpx;
  color: #666;
  line-height: 1.6;
  display: block;
}

.action-buttons {
  display: flex;
  padding: 30rpx;
  gap: 20rpx;
  background-color: #f9fafc;
  border-top: 1rpx solid #f0f0f0;
}

.btn {
  flex: 1;
  height: 80rpx;
  border-radius: 40rpx;
  font-size: 32rpx;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
}

.btn-primary {
  background-color: #4a90e2;
  color: white;
  border: none;
}

.btn-primary:active {
  background-color: #3a7bc8;
  transform: scale(0.98);
}

.btn-secondary {
  background-color: white;
  color: #4a90e2;
  border: 1rpx solid #4a90e2;
}

.btn-secondary:active {
  background-color: rgba(74, 144, 226, 0.05);
  transform: scale(0.98);
}
</style>