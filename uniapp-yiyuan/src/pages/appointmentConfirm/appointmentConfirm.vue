<template>
  <view class="confirm-container">
    <view class="page-header">确认预约信息</view>

    <view class="form-card">
      <!-- 预约信息 -->
      <view class="form-section">
        <view class="section-title">预约信息</view>
        <view class="info-item">
          <view class="info-label">就诊医生：</view>
          <view class="info-value">{{ appointmentInfo.doctorName }}</view>
        </view>
        <view class="info-item">
          <view class="info-label">就诊日期：</view>
          <view class="info-value">{{formatYmdToShow(visitDate) }}</view>
        </view>
        <view class="info-item">
          <view class="info-label">就诊时间：</view>
          <view class="info-value">{{ appointmentInfo.timeSlot | formatTimeSlot }}</view>
        </view>
        <view class="info-item">
          <view class="info-label">预约门诊：</view>
          <view class="info-value">{{ appointmentInfo.diseaseName }}</view>
        </view>
      </view>

      <!-- 用户信息 -->
      <view class="form-section">
        <view class="section-title">就诊人信息</view>
        <view class="form-input-item">
          <view class="input-label">姓名</view>
          <input class="input-field" type="text" v-model="patientInfo.real_name" placeholder="请输入就诊人姓名" />
        </view>
        <view class="form-input-item">
          <view class="input-label">身份证号</view>
          <input class="input-field" type="text" v-model="patientInfo.id_card" placeholder="请输入就诊人身份证号" />
        </view>
      </view>

      <!-- 费用信息 -->
      <view class="form-section">
        <view class="fee-item">
          <view class="fee-label">预约费用</view>
          <view class="fee-value">¥{{ appointmentInfo.unit_price }}</view>
        </view>
      </view>
    </view>

    <!-- 底部操作栏 -->
    <view class="action-bar">
      <button class="cancel-btn" @click="handleCancel">取消预约</button>
      <button class="confirm-btn" @click="handleConfirm">确认预约</button>
    </view>
  </view>
</template>

<script>
// 全局配置：后端基础地址（根据实际部署地址修改）
const BASE_URL = 'http://localhost/api';

export default {
  data() {
    return {
      // 从上一页接收的预约信息（字段名对齐后端）
      appointmentInfo: {
        doctorName: '',    // 对应后端 doctor_name
        timeSlot: '',      // 对应后端 time_slot
        diseaseName: '',   // 对应后端 disease_name
        unit_price: 0,     // 对应后端 unit_price（费用字段名对齐）
        deptId: '',
        scheduleId: '',
        visitDate: '',     // 存储标准格式日期（YYYY-MM-DD），用于提交订单
        showDate: ''       // 存储显示格式日期（11月28日），用于页面渲染
      },
      // 就诊人信息（字段名对齐后端必填参数）
      patientInfo: {
        real_name: '',     // 对应后端 real_name
        id_card: ''        // 对应后端 id_card
      },
      // 全局存储的用户 openid（登录后获取，必填）
      openid: '',
      scheduleId: '',
      isLoadingOpenid: false // 标记 openid 获取中，避免重复请求
    };
  },
  onLoad(option) {
    console.log("接收的参数:", option);
    // 1. 解析上一页传递的参数（字段名对齐）
    if (option) {
      this.visitDate = option.visitDate || '';
      this.appointmentInfo.showDate = option.showDate ? decodeURIComponent(option.showDate) : '';
      this.appointmentInfo.doctorName = option.doctorName || '';
      this.appointmentInfo.timeSlot = option.timeSlot || '';
      this.appointmentInfo.diseaseName = decodeURIComponent(option.diseaseName || '');
      this.appointmentInfo.unit_price = Number(option.price || 0); // 费用字段名统一为 unit_price
      this.appointmentInfo.deptId = option.deptId || '';
      this.appointmentInfo.scheduleId = option.scheduleId || '';
    }

    // 2. 调用标准 getOpenid 方法获取用户标识（核心修改）
    this.getOpenid();
  },
  filters: {
    formatTimeSlot(timeSlot) {
      if (!timeSlot) return '';
      return timeSlot.replace(/(\d+)-(\d+)/, '$1:00-$2:00');
    }
  },
  methods: {
    // 核心：标准 getOpenid 方法（复用指定逻辑）
    getOpenid() {
      this.isLoadingOpenid = true;
      // 第一步：优先读取本地缓存的 openid
      const cache = uni.getStorageSync('openid');
      if (cache) {
        this.openid = cache;
        this.isLoadingOpenid = false;
        return;
      }

      // 第二步：缓存无 openid，触发微信登录获取 code
      uni.login({
        provider: 'weixin', // 指定微信登录（uni-app 多平台兼容）
        success: loginRes => {
          if (loginRes.code) {
            // 第三步：用 code 向后端请求 openid
            uni.request({
              url: `${BASE_URL}/user/openid`, // 后端换 openid 接口地址
              method: 'POST',
              data: { code: loginRes.code }, // 传递微信临时 code
              success: res => {
                this.isLoadingOpenid = false;
                if (res.data.code === 200 && res.data.data.openid) {
                  // 第四步：获取成功，存储到本地缓存并赋值
                  this.openid = res.data.data.openid;
                  uni.setStorageSync('openid', this.openid);
                  uni.showToast({ title: '登录成功', icon: 'success', duration: 1000 });
                } else {
                  uni.showToast({ title: '获取用户标识失败，请重试', icon: 'none' });
                }
              },
              fail: () => {
                this.isLoadingOpenid = false;
                uni.showToast({ title: '网络请求失败，请检查网络', icon: 'none' });
              }
            });
          }
        },
        fail: () => {
          this.isLoadingOpenid = false;
          uni.showToast({ title: '微信登录失败，请允许微信授权', icon: 'none' });
        }
      });
    },

    // 取消预约：返回上一页
    handleCancel() {
      uni.navigateBack();
    },

    // 日期格式转换
    formatYmdToShow(ymd) {
      if (!ymd) return '';
      const [year, month, day] = ymd.split('-');
      return `${month}月${day}日`;
    },

    /**
     * 确认预约：调用后端 create 接口创建订单
     */
    handleConfirm() {
      // 1. 先校验 openid 状态
      if (this.isLoadingOpenid) {
        return uni.showToast({ title: '登录中，请稍候...', icon: 'none' });
      }
      if (!this.openid) {
        return uni.showToast({ title: '用户未登录，请先登录', icon: 'none' });
      }

      // 2. 表单校验（对齐后端必填参数）
      const { real_name, id_card } = this.patientInfo;
      const { doctorName, timeSlot, diseaseName, unit_price, scheduleId } = this.appointmentInfo;

      if (!real_name) return uni.showToast({ title: '请输入就诊人姓名', icon: 'none' });
      if (!id_card || !this.validateIdCard(id_card)) return uni.showToast({ title: '请输入正确的身份证号', icon: 'none' });
      if (!doctorName) return uni.showToast({ title: '就诊医生不能为空', icon: 'none' });
      if (!timeSlot) return uni.showToast({ title: '就诊时间不能为空', icon: 'none' });
      if (!diseaseName) return uni.showToast({ title: '预约门诊不能为空', icon: 'none' });
      if (unit_price <= 0) return uni.showToast({ title: '预约费用异常', icon: 'none' });
      if (!scheduleId) return uni.showToast({ title: '排班信息异常', icon: 'none' });

      // 3. 调用后端创建订单接口
      uni.showLoading({ title: '正在创建预约单...' });
      uni.request({
        url: `${BASE_URL}/order/create`, // 后端创建订单接口地址
        method: 'POST',
        data: {
          openid: this.openid,          // 后端必填：用户唯一标识
          real_name: real_name,         // 后端必填：就诊人姓名
          id_card: id_card,             // 后端必填：身份证号
          disease_name: diseaseName,    // 后端必填：预约门诊（疾病名称）
          doctor_name: doctorName,      // 后端必填：医生姓名
          time_slot: timeSlot,          // 后端必填：就诊时间
          unit_price: unit_price,        // 后端必填：单价（预约费用）
          schedule_id: this.appointmentInfo.scheduleId, // 后端必填：排班ID
          visit_date: this.visitDate, // 关键：传递标准格式的就诊日期
        },
        success: (res) => {
          uni.hideLoading();
          const { code, msg, data } = res.data;

          if (code === 200) {
            // 3. 创建订单成功，获取订单号并跳转支付
            this.incrementScheduleUsage(this.appointmentInfo.scheduleId); // 增加排班使用次数
            const orderId = data.order_id;
            uni.showToast({ title: msg, icon: 'success' });
            setTimeout(() => {
              this.showPaymentModal(orderId); // 显示支付弹窗
            }, 1000);
          } else if (code === 404) {
            // 用户不存在：重新触发登录（可能是 openid 失效）
            uni.showToast({ title: '用户未注册，重新登录中...', icon: 'none' });
            setTimeout(() => {
              this.getOpenid(); // 重新获取 openid 并同步后端
            }, 1500);
          } else {
            uni.showToast({ title: msg || '创建预约单失败', icon: 'none' });
          }
        },
        fail: (err) => {
          uni.hideLoading();
          console.error('创建订单接口请求失败:', err);
          uni.showToast({ title: '网络异常，请重试', icon: 'none' });
        }
      });
    },

    /**
     * 显示支付确认弹窗
     * @param {string} orderId - 后端返回的订单ID
     */
    showPaymentModal(orderId) {
      uni.showModal({
        title: '支付确认',
        content: `请支付预约费用：¥${this.appointmentInfo.unit_price}`,
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
     * 增加排班使用次数
     */
    incrementScheduleUsage(scheduleId) {
      uni.request({
        url: `${BASE_URL}/schedule/incrementUsed`,
        method: 'POST',
        data: {
          schedule_id: scheduleId // 排班ID
        },
        success: (res) => {
          console.log('更新排班使用次数:', res.data);
          if (res.data.code !== 200) {
            console.warn('排班使用次数更新失败:', res.data.msg);
          }
        },
        fail: (err) => {
          console.error('更新排班使用次数失败:', err);
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
            // 直接跳转回schedule界面，不传递参数
            this.navigateToSchedule();
          }
        }
      });
    },

    /**
     * 处理支付：这里对接真实支付（如微信支付），支付成功后调用后端更新状态接口
     * @param {string} orderId - 订单ID
     */
    processPayment(orderId) {
      uni.showLoading({ title: '正在处理支付...' });

      // 模拟真实支付流程（实际项目中替换为微信支付/支付宝支付接口）
      setTimeout(() => {
        // 假设支付成功，调用后端更新支付状态接口
        this.updatePayStatus(orderId, 1); // 1=已支付（对应后端 PAY_STATUS_PAID 常量）
      }, 2000);
    },

    /**
     * 调用后端接口更新支付状态
     * @param {string} orderId - 订单ID
     * @param {number} payStatus - 支付状态（1=已支付，0=未支付）
     */
    updatePayStatus(orderId, payStatus) {
      uni.request({
        url: `${BASE_URL}/order/updatePayStatus`, // 后端更新支付状态接口
        method: 'POST',
        data: {
          order_id: orderId,    // 后端必填：订单ID
          pay_status: payStatus // 后端必填：支付状态（1=已支付，0=未支付）
        },
        success: (res) => {
          uni.hideLoading();
          const { code, msg } = res.data;

          if (code === 200) {
            // 支付成功：显示成功弹窗
            this.showSuccessModal();
          } else {
            uni.showToast({ title: msg || '更新支付状态失败', icon: 'none' });
          }
        },
        fail: (err) => {
          uni.hideLoading();
          console.error('更新支付状态接口请求失败:', err);
          uni.showToast({ title: '网络异常，支付状态更新失败', icon: 'none' });
        }
      });
    },

    /**
     * 支付成功弹窗
     */
    showSuccessModal() {
      uni.showModal({
        title: '支付成功',
        content: '支付成功，预约信息可在"我的"界面"挂号记录"查看',
        showCancel: false,
        confirmText: '确定',
        success: (res) => {
          if (res.confirm) {
            // 直接跳转回schedule界面，不传递参数
            this.navigateToSchedule();
          }
        }
      });
    },

    /**
     * 跳转到schedule界面（不传递参数，避免参数问题）
     */
    navigateToSchedule() {
      uni.switchTab({
        url: '/pages/index/index', // tabBar 首页的路径（对应 pages.json 中的配置）
        success: () => {
          console.log('成功跳转到首页');
        },
        fail: (err) => {
          console.error('跳转首页失败:', err);
        }
      });
    },

    // 身份证号校验（保持原逻辑）
    validateIdCard(idCard) {
      const reg = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/;
      return reg.test(idCard);
    }
  }
};
</script>

<style scoped>
/* 样式保持不变，沿用原样式 */
.confirm-container {
  background-color: #f5f5f5;
  min-height: 100vh;
  padding-bottom: 120rpx;
  box-sizing: border-box;
}
.page-header {
  font-size: 32rpx;
  font-weight: 500;
  text-align: center;
  padding: 30rpx 0;
  background-color: #fff;
  border-bottom: 1rpx solid #eee;
}
.form-card {
  margin: 30rpx;
  padding: 30rpx;
  background-color: #fff;
  border-radius: 20rpx;
}
.section-title {
  font-size: 28rpx;
  font-weight: 500;
  color: #333;
  margin-bottom: 20rpx;
  padding-left: 10rpx;
  border-left: 6rpx solid #007aff;
}
.info-item {
  display: flex;
  padding: 15rpx 0;
  border-bottom: 1rpx solid #f0f0f0;
}
.info-item:last-child {
  border-bottom: none;
}
.info-label {
  flex: 0 0 160rpx;
  font-size: 26rpx;
  color: #666;
}
.info-value {
  flex: 1;
  font-size: 26rpx;
  color: #333;
}
.form-input-item {
  display: flex;
  align-items: center;
  padding: 25rpx 0;
  border-bottom: 1rpx solid #f0f0f0;
}
.form-input-item:last-child {
  border-bottom: none;
}
.input-label {
  flex: 0 0 160rpx;
  font-size: 26rpx;
  color: #666;
}
.input-field {
  flex: 1;
  font-size: 26rpx;
  color: #333;
}
.fee-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20rpx 0;
}
.fee-label {
  font-size: 28rpx;
  color: #333;
}
.fee-value {
  font-size: 32rpx;
  color: #ff5722;
  font-weight: bold;
}
.action-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  display: flex;
  height: 100rpx;
  line-height: 100rpx;
  background-color: #fff;
  border-top: 1rpx solid #eee;
}
.cancel-btn, .confirm-btn {
  flex: 1;
  height: 100rpx;
  line-height: 100rpx;
  border-radius: 0;
  font-size: 28rpx;
}
.cancel-btn {
  background-color: #fff;
  color: #333;
}
.confirm-btn {
  background-color: #007aff;
  color: #fff;
}
</style>