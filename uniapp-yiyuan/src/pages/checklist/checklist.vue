<template>
  <view class="checklist-empty">
    <!-- 图标 -->
    <uni-icons 
      type="document" 
      size="80" 
      color="#ccc"
      class="empty-icon"
    ></uni-icons>
    
    <!-- 标题提示 -->
    <view class="empty-title">暂无检查单</view>
    
    <!-- 副标题说明 -->
    <view class="empty-desc">您当前没有已生成的检查单记录</view>
    
    <!-- 可选：返回/去预约按钮（按需保留） -->
    <button class="back-btn" @click="handleBack">返回上一页</button>
  </view>
</template>

<script>
export default {
  methods: {
    // 返回上一页
    handleBack() {
      uni.navigateBack();
    },
    // 若需要跳转到预约页面，可添加：
    // goToAppoint() {
    //   uni.navigateTo({ url: '/pages/appoint/appoint' });
    // }
  }
}
</script>

<style scoped>
.checklist-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #f5f5f5;
  padding: 30rpx;
  box-sizing: border-box;
}

.empty-icon {
  margin-bottom: 40rpx;
  opacity: 0.6;
}

.empty-title {
  font-size: 32rpx;
  color: #333;
  font-weight: 500;
  margin-bottom: 20rpx;
}

.empty-desc {
  font-size: 26rpx;
  color: #999;
  margin-bottom: 80rpx;
  text-align: center;
}

.back-btn {
  width: 60%;
  height: 80rpx;
  line-height: 80rpx;
  font-size: 28rpx;
  background-color: #007aff;
  color: #fff;
  border-radius: 40rpx;
}

/* 若保留去预约按钮，可添加：
.appoint-btn {
  width: 60%;
  height: 80rpx;
  line-height: 80rpx;
  font-size: 28rpx;
  background-color: #fff;
  color: #007aff;
  border: 1rpx solid #007aff;
  border-radius: 40rpx;
  margin-top: 20rpx;
}
*/
</style>