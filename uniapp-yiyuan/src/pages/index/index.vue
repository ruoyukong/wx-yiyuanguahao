<template>
  <view class="container">
    <!-- 顶部轮播图 -->
    <view class="swiper-container">
      <swiper class="swiper" indicator-dots circular autoplay interval="3000" indicator-active-color="#4a90e2">
        <swiper-item v-for="(item, index) in swiperList" :key="index">
          <image :src="item.src" mode="aspectFill" class="swiper-img" @click="handleItemClick(index)"></image>
          <view class="swiper-overlay"></view>
        </swiper-item>
      </swiper>
    </view>

    <!-- 功能按钮区：四宫格布局 -->
    <view class="func-container">
      <view class="section-header">
        <text class="section-title">便捷服务</text>
        <text class="section-desc">快速办理医疗业务</text>
      </view>
      
      <view class="func-grid">
        <!-- 预约挂号 -->
        <view class="func-item" @click="navigateTo('/pages/register/register')">
          <view class="func-icon-wrapper">
            <image class="func-icon" src="/static/icons/yiyuanguahao.png" mode="widthFix" lazy-load></image>
          </view>
          <text class="func-text">预约挂号</text>
        </view>

        <!-- 缴费 -->
        <view class="func-item" @click="navigateTo('/pages/payment/payment')">
          <view class="func-icon-wrapper">
            <image class="func-icon" src="/static/icons/jiaofeijilu.png" mode="widthFix" lazy-load></image>
          </view>
          <text class="func-text">缴费</text>
        </view>

        <!-- 检查单 -->
        <view class="func-item" @click="navigateTo('/pages/checklist/checklist')">
          <view class="func-icon-wrapper">
            <image class="func-icon" src="/static/icons/guahaoxinxi.png" mode="widthFix" lazy-load></image>
          </view>
          <text class="func-text">检查单</text>
        </view>

        <!-- 科室信息 -->
        <view class="func-item" @click="navigateTo('/pages/department/department')">
          <view class="func-icon-wrapper">
            <image class="func-icon" src="/static/icons/keshixinxi.png" mode="widthFix" lazy-load></image>
          </view>
          <text class="func-text">科室信息</text>
        </view>
      </view>
    </view>

    <!-- 医院信息入口 -->
    <view class="hospital-entry" @click="switchToTab('/pages/hospital/hospital')">
      <view class="entry-content">
        <view class="entry-icon">🏥</view>
        <view class="entry-text">
          <text class="entry-title">医院信息</text>
          <text class="entry-desc">查看医院详情与科室介绍</text>
        </view>
      </view>
      <uni-icons type="right" size="18" color="#999"></uni-icons>
    </view>

    <!-- 底部装饰 -->
    <view class="bottom-decoration">
      <text class="slogan">您的健康 · 我们的责任</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      swiperList: [
        {
          src: '/static/xinzangjiankang.png',
          link: 'https://y.dxy.cn/hospital/12573/988346.html'
        },
        {
          src: '/static/xinlijiankang.png',
          link: 'https://dxs.moe.gov.cn/zx/a/xl_xlyz_xlzs/241018/1975830.shtml'
        },
        {
          src: '/static/mbti.png',
          link: 'https://16ceshi.com/pages/assessment/start/start?source=bing_mbti_jz01&msclkid=7081374cd06b1653cfae5e4c0a2f05f3&timestamp=1761663559564#'
        },
      ]
    };
  },
  methods: {
    switchToTab(url) {
      uni.switchTab({ url });
    },

    handleItemClick(index) {
      uni.navigateTo({
        url: `/pages/webview/webview?url=${encodeURIComponent(this.swiperList[index].link)}`
      });
    },

    checkUserInfo() {
      const userInfo = uni.getStorageSync('userInfo');
      if (userInfo && userInfo.nickName && userInfo.avatarUrl) {
        return true;
      }
      return false;
    },

    navigateTo(targetUrl) {
      if (!this.checkUserInfo()) {
        uni.showModal({
          title: '提示',
          content: '请先在"我的"页面授权您的个人信息',
          showCancel: false,
          confirmText: '前往授权',
          success: (res) => {
            if (res.confirm) {
              uni.switchTab({
                url: '/pages/mine/mine' 
              });
            }
          }
        });
        return;
      }

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
  background: linear-gradient(135deg, #f8fafc 0%, #e8f0fe 100%);
  min-height: 100vh;
  padding-bottom: 40rpx;
}

/* 轮播图样式优化 */
.swiper-container {
  position: relative;
  border-radius: 0 0 40rpx 40rpx;
  overflow: hidden;
  box-shadow: 0 10rpx 40rpx rgba(74, 144, 226, 0.15);
  margin: 0 24rpx; /* 左右添加边距 */
  width: auto; /* 改为自动宽度 */
}

.swiper {
  width: 100%;
  height: 380rpx;
}

.swiper-img {
  width: 100%;
  height: 100%;
  display: block;
}

.swiper-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 120rpx;
  background: linear-gradient(to top, rgba(0,0,0,0.3) 0%, rgba(0,0,0,0) 100%);
}

/* 功能区域样式优化 */
.func-container {
  margin: 40rpx 24rpx;
  background: linear-gradient(135deg, #ffffff 0%, #f8fbff 100%);
  border-radius: 28rpx;
  padding: 40rpx 0 20rpx;
  box-shadow: 0 12rpx 40rpx rgba(74, 144, 226, 0.08);
  position: relative;
  overflow: hidden;
}

.func-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4rpx;
  background: linear-gradient(90deg, #4a90e2, #67b26f);
}

.section-header {
  padding: 0 40rpx 30rpx;
  border-bottom: 1rpx solid #f0f4f8;
  margin-bottom: 20rpx;
}

.section-title {
  font-size: 36rpx;
  font-weight: 600;
  color: #1a1a1a;
  display: block;
  margin-bottom: 8rpx;
}

.section-desc {
  font-size: 26rpx;
  color: #888;
  display: block;
}

.func-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(2, 1fr);
  gap: 40rpx 20rpx;
  padding: 20rpx 40rpx;
}

.func-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40rpx 20rpx;
  background: #fff;
  border-radius: 24rpx;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
  box-shadow: 0 6rpx 20rpx rgba(74, 144, 226, 0.08);
  border: 1rpx solid #f0f4f8;
}

.func-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4rpx;
  background: linear-gradient(90deg, #4a90e2, #67b26f);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.func-item:active {
  transform: translateY(-4rpx);
  box-shadow: 0 12rpx 30rpx rgba(74, 144, 226, 0.15);
}

.func-item:active::before {
  opacity: 1;
}

.func-icon-wrapper {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  background: linear-gradient(135deg, #e8f2ff 0%, #d4e6ff 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 24rpx;
  transition: all 0.3s ease;
  border: 2rpx solid #e8f2ff;
}

.func-item:active .func-icon-wrapper {
  transform: scale(1.05);
  background: linear-gradient(135deg, #d4e6ff 0%, #c2dcff 100%);
}

.func-icon {
  width: 60rpx;
  height: 60rpx;
}

.func-text {
  font-size: 30rpx;
  font-weight: 500;
  color: #333;
  text-align: center;
  transition: color 0.3s ease;
}

.func-item:active .func-text {
  color: #4a90e2;
}

/* 医院信息入口样式优化 */
.hospital-entry {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 40rpx;
  background: linear-gradient(135deg, #ffffff 0%, #f8fbff 100%);
  margin: 30rpx 24rpx;
  border-radius: 28rpx;
  box-shadow: 0 8rpx 30rpx rgba(74, 144, 226, 0.08);
  border: 1rpx solid #f0f4f8;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.hospital-entry::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4rpx;
  background: linear-gradient(90deg, #4a90e2, #67b26f);
}

.hospital-entry:active {
  transform: translateY(-2rpx);
  box-shadow: 0 12rpx 35rpx rgba(74, 144, 226, 0.12);
}

.entry-content {
  display: flex;
  align-items: center;
  flex: 1;
}

.entry-icon {
  font-size: 48rpx;
  margin-right: 24rpx;
  width: 80rpx;
  height: 80rpx;
  border-radius: 50%;
  background: linear-gradient(135deg, #e8f2ff 0%, #d4e6ff 100%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.entry-text {
  display: flex;
  flex-direction: column;
}

.entry-title {
  font-size: 34rpx;
  font-weight: 600;
  color: #1a1a1a;
  margin-bottom: 6rpx;
}

.entry-desc {
  font-size: 26rpx;
  color: #888;
}

/* 底部装饰 */
.bottom-decoration {
  text-align: center;
  padding: 40rpx 0 20rpx;
}

.slogan {
  font-size: 26rpx;
  color: #999;
  font-style: italic;
  letter-spacing: 2rpx;
}
</style>