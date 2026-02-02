<template>
  <!-- 外层容器，避免页面空白 -->
  <view class="webview-container">
    <!-- 加载中提示（web-view加载完成前显示） -->
    
    <view class="loading-view" v-if="isLoading">
      <uni-icons type="loading" size="36" color="#1E88E5" spin></uni-icons>
      <text class="loading-text">加载中...</text>
    </view>
    
    <!-- 核心：承载外部网页的web-view组件 -->
    <web-view
      :src="url"
      @load="handleLoad"       
      @error="handleError"     
      class="webview"
    ></web-view>

    <!-- 加载失败提示（链接无效或域名未配置时显示） -->
    <view class="error-view" v-if="isError">
      <uni-icons type="warn" size="40" color="#FF5722"></uni-icons>
      <text class="error-title">页面加载失败</text>
      <text class="error-desc">请检查链接是否有效，或确认业务域名已配置</text>
      <button class="reload-btn" @click="reloadWebview">重新加载</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      url: '',               // 接收的外部链接
      isLoading: true,       // 是否正在加载
      isError: false         // 是否加载失败
    };
  },
  onLoad(options) {
    // 1. 接收并解码跳转传过来的链接（处理特殊字符，如?、&等）
    if (options.url) {
      this.url = decodeURIComponent(options.url);
    } else {
      // 若未传链接，直接提示并返回上一页
      uni.showToast({ title: '未获取到跳转链接', icon: 'none' });
      setTimeout(() => uni.navigateBack(), 1500);
    }
  },
  methods: {
    // 2. web-view加载完成：隐藏加载提示
    handleLoad() {
      this.isLoading = false;
      this.isError = false; // 重置错误状态
    },

    // 3. web-view加载失败：显示错误提示
    handleError() {
      this.isLoading = false;
      this.isError = true;
      console.error('web-view加载失败，链接：', this.url);
    },

    // 4. 重新加载网页
    reloadWebview() {
      this.isLoading = true;
      this.isError = false;
      // 重新赋值url触发web-view刷新
      const tempUrl = this.url;
      this.url = '';
      this.$nextTick(() => {
        this.url = tempUrl;
      });
    },

    
 }
};
</script>

<style scoped>
.webview-container {
  width: 100%;
  height: 100vh; /* 占满整个屏幕高度 */
  overflow: hidden; /* 隐藏滚动条（可选） */
  position: relative; /* 关键：让按钮可悬浮在web-view上方 */
}


/* 加载中样式：居中显示 */
.loading-view {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #fff;
  z-index: 999; /* 确保在web-view上方显示 */
}
.loading-text {
  font-size: 28rpx;
  color: #666;
  margin-top: 20rpx;
}

/* web-view容器（层级最低） */
.webview {
  width: 100%;
  height: 100%;
  z-index: 1; /* 确保在最底层 */
}

/* 加载失败样式：居中显示 */
.error-view {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #fff;
  z-index: 999;
  padding: 0 40rpx;
  box-sizing: border-box;
}
.error-title {
  font-size: 32rpx;
  color: #333;
  margin-top: 30rpx;
  font-weight: 500;
}
.error-desc {
  font-size: 26rpx;
  color: #999;
  margin-top: 16rpx;
  text-align: center;
  line-height: 40rpx;
}
.reload-btn {
  margin-top: 40rpx;
  background-color: #1E88E5;
  color: #fff;
  font-size: 28rpx;
  padding: 12rpx 40rpx;
  border-radius: 60rpx;
}
</style>