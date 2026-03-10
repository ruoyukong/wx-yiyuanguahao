<template>
  <view class="container">
    <!-- 用户信息区 -->
    <view class="user-header">
      <!-- 1. 头像：官方 chooseAvatar 按钮 -->
      <button
        class="avatar-btn"
        :disabled="isInfoComplete"
        open-type="chooseAvatar"
        @chooseavatar="onChooseAvatar"
      >
        <image
          :src="userInfo.avatarUrl || '/static/wode-001.png'"
          class="avatar"
        />
      </button>

      <!-- 2. 昵称：官方 type=nickname 输入框 -->
      <view class="nickname-box" v-if="!isInfoComplete">
      <input
        v-if="!isInfoComplete"
        class="nickname-input"
        type="nickname"
        placeholder="点击选择昵称"
        :value="userInfo.nickName"
        @blur="onNicknameInput"
      />
      </view>
      <text v-else class="username-text">{{ userInfo.nickName }}</text>
    </view>

    <!-- 功能列表 -->
    <view class="func-list">
      <view class="list-title">我的服务</view>
      <view class="func-item" @click="navigateTo('/pages/record/record')">
        <view class="func-left">
          <view class="func-icon">📋</view>
          <text class="func-text">挂号记录</text>
        </view>
        <uni-icons type="right" size="18" color="#c0c4cc" />
      </view>
      <view class="func-item" @click="navigateTo('/pages/record/pay')">
        <view class="func-left">
          <view class="func-icon">💰</view>
          <text class="func-text">缴费记录</text>
        </view>
        <uni-icons type="right" size="18" color="#c0c4cc" />
      </view>
    </view>

    <!-- 协议列表 -->
    <view class="agreement-list">
      <view class="list-title">协议与隐私</view>
      <view class="agreement-item" @click="openAgreement('user')">
        <view class="func-left">
          <view class="func-icon">📝</view>
          <view class="text-content">
            <text class="func-text">用户授权协议</text>
            <text class="agreement-desc">了解您的权利和义务</text>
          </view>
        </view>
        <uni-icons type="right" size="18" color="#c0c4cc" />
      </view>
      <view class="agreement-item" @click="openAgreement('privacy')">
        <view class="func-left">
          <view class="func-icon">🔒</view>
          <view class="text-content">
            <text class="func-text">隐私政策</text>
            <text class="agreement-desc">我们如何保护您的个人信息</text>
          </view>
        </view>
        <uni-icons type="right" size="18" color="#c0c4cc" />
      </view>
      <view class="agreement-item" @click="openAgreement('service')">
        <view class="func-left">
          <view class="func-icon">📄</view>
          <view class="text-content">
            <text class="func-text">服务协议</text>
            <text class="agreement-desc">使用医疗服务相关条款</text>
          </view>
        </view>
        <uni-icons type="right" size="18" color="#c0c4cc" />
      </view>
      <view class="agreement-item" @click="openAgreement('about')">
        <view class="func-left">
          <view class="func-icon">ℹ️</view>
          <view class="text-content">
            <text class="func-text">关于我们</text>
            <text class="agreement-desc">医院信息与服务介绍</text>
          </view>
        </view>
        <uni-icons type="right" size="18" color="#c0c4cc" />
      </view>
    </view>

    <!-- 退出登录 -->
    <view class="tuichu-entry" @click="onLogout">
      <text class="title">退出登录</text>
      <uni-icons type="right" size="16" color="#e53935" />
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      userInfo: { avatarUrl: '', nickName: '' },
      openid: '',
      isInfoComplete: false
    };
  },

  onLoad() {
    this.getOpenid();
    this.loadUserInfo();
  },

  methods: {
    /* 加载本地缓存 */
    loadUserInfo() {
      const saved = uni.getStorageSync('userInfo');
      if (saved && saved.avatarUrl && saved.nickName) {
        this.userInfo = saved;
        this.isInfoComplete = true;
      } else {
        this.isInfoComplete = false;
      }
    },

    /* 获取 openid（原逻辑） */
    getOpenid() {
      const cache = uni.getStorageSync('openid');
      if (cache) { this.openid = cache; return; }
      uni.login({
        provider: 'weixin',
        success: loginRes => {
          if (loginRes.code) {
            uni.request({
              url: 'http://localhost/api/user/openid',
              method: 'POST',
              data: { code: loginRes.code },
              success: res => {
                if (res.data.code === 200 && res.data.data.openid) {
                  this.openid = res.data.data.openid;
                  uni.setStorageSync('openid', this.openid);
                } else {
                  uni.showToast({ title: '获取用户标识失败', icon: 'none' });
                }
              },
              fail: () => uni.showToast({ title: '网络请求失败', icon: 'none' })
            });
          }
        },
        fail: () => uni.showToast({ title: '微信登录失败', icon: 'none' })
      });
    },

    /* 头像选择回调 */
    onChooseAvatar(e) {
      this.userInfo.avatarUrl = e.detail.avatarUrl;
      uni.setStorageSync('userInfo', this.userInfo);
      this.checkCompleted();
    },

    /* 昵称输入/回填回调 */
    onNicknameInput(e) {
      const nickName = e.detail.value.trim();
      if (!nickName) return;
      this.userInfo.nickName = nickName;
      uni.setStorageSync('userInfo', this.userInfo);
      this.checkCompleted();
    },

    /* 检查是否全部完成 */
    checkCompleted() {
      if (this.userInfo.avatarUrl && this.userInfo.nickName) {
        this.isInfoComplete = true;
        uni.setStorageSync('isInfoComplete', true);
        this.syncUserInfoToBackend();
      }
    },

    /* 同步到后端 */
    syncUserInfoToBackend() {
      if (!this.openid || !this.userInfo.avatarUrl || !this.userInfo.nickName) return;
      uni.request({
        url: 'http://localhost/api/user/login',
        method: 'POST',
        data: {
          openid: this.openid,
          nickname: this.userInfo.nickName,
          avatar_url: this.userInfo.avatarUrl
        },
        success: res => {
          if (res.data.code !== 200) console.log('同步失败：', res.data.msg);
        },
        fail: () => console.log('同步网络失败')
      });
    },

    /* 退出登录 */
    onLogout() {
      uni.showModal({
        title: '提示',
        content: '确定要退出登录吗？',
        success: res => {
          if (res.confirm) {
            uni.removeStorageSync('userInfo');
            uni.removeStorageSync('openid');
            uni.removeStorageSync('isInfoComplete');
            this.userInfo = { avatarUrl: '', nickName: '' };
            this.openid = '';
            this.isInfoComplete = false;
            uni.showToast({ title: '已退出登录', icon: 'success' });
          }
        }
      });
    },

    /* 页面跳转 */
    navigateTo(url) { uni.navigateTo({ url }); },

    /* 打开协议 */
    openAgreement(type) {
      const titles = {
        'user': '用户授权协议',
        'privacy': '隐私政策', 
        'service': '服务协议',
        'about': '关于我们'
      };
      const title = titles[type] || '协议详情';
      uni.navigateTo({ url: `/pages/agreement/agreement?title=${title}&type=${type}` });
    }
  }
};
</script>

<style scoped>
/* 页面整体样式（与原样式一致） */
.container {
  background-color: #f5f5f5;
  min-height: 100vh;
}

/* 用户信息区（与原样式一致） */
.user-header {
  background-color: hsl(208, 100%, 88%);
  padding: 48rpx 24rpx;
  display: flex;
  align-items: center;
}

/* 头像按钮：固定尺寸 + 禁止压缩 */
.avatar-btn {
  padding: 0;
  background: transparent;
  border: none;
  line-height: 1;
  width: 160rpx;
  height: 160rpx;
  border-radius: 50%;
  overflow: hidden;
  flex-shrink: 0; /* 关键：禁止 Flex 压缩头像尺寸 */
  text-align: left !important;   /* 关键：阻止居中 */
  line-height: 50rpx;            /* 与 input 内文字同高 */
}

/* 禁用状态完全继承样式，无变化 */
.avatar-btn:disabled {
  opacity: 1;
}

/* 移除按钮默认边框 */
.avatar-btn::after {
  border: none;
}

/* 头像样式（与原样式一致） */
.avatar {
  width: 100%;
  height: 100%;
  display: block;
  border-radius: 50%;
  border: 6rpx solid #fff;
  box-sizing: border-box;
}
/* 外层盒子：与授权后 text 同尺寸同间距 */
.nickname-box {
  height: 50rpx;
  line-height: 50rpx;
  margin-left: 24rpx;
  flex: 1;
  display: flex;
  align-items: center;
}

/* input 撑满容器，所有偏移量归零 */
.nickname-input {
  width: 100%;
  height: 100%;
  background: transparent;
  border: none;
  font-size: 36rpx;
  font-weight: 500;
  color: #fff;
  text-align: left;
  padding: 0;
  line-height: 1;
}

/* 授权后 text 保持同盒型 */
.username-text {
  height: 50rpx;
  line-height: 50rpx;
  margin-left: 24rpx;
  flex: 1;
  font-size: 36rpx;
  font-weight: 500;
  color: #fff;
}
/* 禁用状态完全继承样式，无变化 */
.nickname-input:disabled {
  background: transparent;
  border: none;
  font-size: 36rpx;
  font-weight: 500;
  color: #fff;
  opacity: 1;
  pointer-events: none;
  margin-left: 24rpx;
  flex: 1;
  flex-shrink: 0;
  min-width: 0;
  text-align: left !important;   /* 关键：阻止居中 */
  line-height: 50rpx;            /* 与 input 内文字同高 */
}

/* 昵称占位符样式（与原样式一致） */
.nickname-input::placeholder {
  color: rgba(255, 255, 255, 0.7);
}

/* 功能列表和协议列表美化 */
.func-list, .agreement-list {
  background: #fff;
  margin: 30rpx 24rpx;
  border-radius: 24rpx;
  box-shadow: 0 8rpx 32rpx rgba(74, 144, 226, 0.08);
  overflow: hidden;
}

.list-title {
  font-size: 28rpx;
  color: #999;
  padding: 30rpx 32rpx 20rpx;
  border-bottom: 1rpx solid #f0f4f8;
}

/* 列表项美化 */
.func-item, .agreement-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 32rpx;
  border-bottom: 1rpx solid #f0f4f8;
  transition: all 0.3s ease;
  background: #fff;
}

.func-item:active, .agreement-item:active {
  background: #f8fafc;
  transform: translateX(4rpx);
}

.func-item:last-child, .agreement-item:last-child {
  border-bottom: none;
}

.func-left {
  display: flex;
  align-items: center;
  flex: 1;
}

.func-icon {
  width: 60rpx;
  height: 60rpx;
  border-radius: 12rpx;
  background: linear-gradient(135deg, #e8f2ff 0%, #d4e6ff 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 24rpx;
  font-size: 28rpx;
  flex-shrink: 0;
}

.text-content {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.func-text {
  font-size: 32rpx;
  color: #333;
  font-weight: 500;
  margin-bottom: 4rpx;
}

.agreement-desc {
  font-size: 24rpx;
  color: #999;
  line-height: 1.4;
}

/* 退出登录（与原样式一致） */
.tuichu-entry {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 50rpx;
  background-color: hsl(208, 100%, 88%);
  margin: 30rpx 16rpx 16rpx;
  border-radius: 20rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

/* 退出登录文本（与原样式一致） */
.tuichu-entry .title {
  font-size: 30rpx;
  color: #fff;
  font-weight: 500;
}

/* 退出登录图标（与原样式一致） */
.tuichu-entry uni-icons {
  color: #fff;
}
</style>