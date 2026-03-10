<template>
  <view class="department-container">
    <view class="page-header">
      <!-- 医生列表页显示返回按钮 -->
      <uni-icons 
        v-if="isShowDoctorList" 
        type="back" 
        size="20" 
        color="#333" 
        @click="backToDeptList()"
      />
      <text class="header-title">
        {{ isShowDoctorList ? (currentDept.name + ' - 医生列表') : '科室信息' }}
      </text>
    </view>

    <!-- 选择医院（仅科室列表页显示） -->
    <view class="hospital-selector-container" v-if="!isShowDoctorList">
      <view class="hospital-selector">
        <picker 
          mode="selector" 
          :range="hospitalList" 
          :value="selectedHospitalIndex"   
          range-key="name"  
          @change="handleHospitalChange"
        >
          <view class="picker-display">
            <text class="picker-label">选择医院</text>
            <text class="picker-value">{{ hospitalList[selectedHospitalIndex].name }}</text>
            <uni-icons class="picker-icon" type="arrow-right" size="16" color="#666" />
          </view>
        </picker>
      </view>
    </view>

    <!-- 加载状态（通用） -->
    <view class="loading" v-if="isLoading">
      <view class="loading-spinner"></view>
      <text>{{ isShowDoctorList ? '加载医生信息中...' : '加载中...' }}</text>
    </view>

    <!-- 错误提示（通用） -->
    <view class="error" v-if="errorMsg && !isLoading">
      <text class="error-icon">❌</text>
      <text class="error-text">{{ errorMsg }}</text>
    </view>

    <!-- 1. 科室列表视图 -->
    <view class="department-list" v-if="!isShowDoctorList && departmentList.length > 0 && !isLoading && !errorMsg">
      <view 
        class="department-item" 
        v-for="dept in departmentList" 
        :key="dept.id"
        @click="handleDeptClick(dept)"
      >
        <view class="dept-header">
          <text class="dept-name">{{ dept.name }}</text>
          <view class="dept-location">
            <uni-icons type="map-pin" size="14" color="#1890ff" />
            <text class="location-text">{{ dept.location ? dept.location : '暂无位置信息' }}</text>
          </view>
        </view>
       
        <view class="dept-intro">
          <text class="intro-label">简介：</text>
          <text class="intro-content">{{ dept.intro ? dept.intro : '暂无简介' }}</text>
        </view>

        <view class="view-doctor-btn">
          <text>查看科室医生</text>
          <uni-icons type="right" size="14" color="#1890ff" />
        </view>
      </view>
    </view>

    <!-- 无科室数据提示 -->
    <view class="no-data" v-if="!isShowDoctorList && departmentList.length === 0 && !isLoading && !errorMsg">
      <uni-icons type="empty" size="40" color="#ccc" />
      <text class="no-data-text">该医院暂无科室数据</text>
    </view>

    <!-- 2. 医生列表视图（点击科室后显示） -->
    <view class="doctor-list-container" v-if="isShowDoctorList && !isLoading && !errorMsg">
      <scroll-view class="doctor-list" scroll-y>
        <view class="doctor-item" v-for="doctor in doctorList" :key="doctor.id">
          <view class="doctor-info">
            <view class="doctor-base">
              <text class="doctor-name">{{ doctor.name ? doctor.name : '未知医生' }}</text>
              <text class="doctor-title">{{ doctor.title ? doctor.title : '主治医师' }}</text>
            </view>
            <text class="doctor-department">科室：{{ currentDept ? currentDept.name : '未知科室' }}</text>
          </view>
          <view class="doctor-desc">
            <text class="desc-label">擅长：</text>
            <text class="desc-content">{{ doctor.good_at ? doctor.good_at : '暂无擅长领域信息' }}</text>
          </view>
        </view>

        <!-- 无医生数据提示 -->
        <view class="no-doctor" v-if="doctorList.length === 0 && !isLoading">
          <uni-icons type="empty" size="36" color="#ccc" />
          <text class="no-doctor-text">该科室暂无医生信息</text>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      selectedHospitalId: '1',
      selectedHospitalIndex: 0,
      departmentList: [],
      isLoading: false,
      errorMsg: '',
      hospitalList: [
        { name: 'dgut市第一人民医院', id: 1 },
        { name: 'dgut市中医院', id: 2 }
      ],
      // 医生相关数据
      currentDept: null, // 当前选中的科室
      doctorList: [], // 医生列表
      isShowDoctorList: false, // 控制显示「科室列表」还是「医生列表」
    };
  },
  onLoad(options) {
    if (options.hospitalId) {
      this.selectedHospitalId = options.hospitalId;
      this.selectedHospitalIndex = this.hospitalList.findIndex(
        item => item.id == options.hospitalId
      );
    }
    this.fetchDepartmentList();
  },
  methods: {
    // 获取科室列表
    fetchDepartmentList() {
      this.isLoading = true;
      this.errorMsg = '';
      
      wx.request({
        url: `http://localhost/api/department/list/hosp/${this.selectedHospitalId}`,
        method: 'GET',
        success: (res) => {
          const { code, msg, data } = res.data;
          if (code === 200) {
            this.departmentList = data;
          } else {
            this.errorMsg = `请求失败：${msg}`;
          }
        },
        fail: (err) => {
          console.error("科室接口请求失败:", err);
          this.errorMsg = err.errMsg.includes('404')
            ? '接口地址错误，请检查后端配置'
            : '网络错误或后端未启动，请重试';
        },
        complete: () => {
          this.isLoading = false;
        }
      });
    },

    // 切换医院
    handleHospitalChange(e) {
      const index = e.detail.value;
      this.selectedHospitalIndex = index;
      this.selectedHospitalId = this.hospitalList[index].id;
      this.fetchDepartmentList();
    },

    // 点击科室：切换到医生列表视图
    handleDeptClick(dept) {
      this.currentDept = dept;
      this.isShowDoctorList = true; // 隐藏科室列表，显示医生列表
      this.fetchDoctorList(dept.id); // 加载该科室医生
    },

    // 返回科室列表
    backToDeptList() {
      this.isShowDoctorList = false; // 显示科室列表，隐藏医生列表
      this.doctorList = []; // 清空医生列表
      this.errorMsg = ''; // 清空错误提示
    },

    // 获取科室医生列表（修复 URL 和参数）
    fetchDoctorList(deptId) {
      this.isLoading = true;
      this.errorMsg = '';
      this.doctorList = [];
      
      wx.request({
        // 正确 URL：后端路由是 /api/doctor/index/科室ID
        url: `http://localhost/api/doctor/index/$dept/${deptId}`, 
        method: 'GET',
        success: (res) => {
          const { code, msg, data } = res.data;
          if (code === 200) {
            this.doctorList = data; // 后端已返回正确医生数据
          } else {
            this.errorMsg = `医生数据获取失败：${msg}`;
          }
        },
        fail: (err) => {
          console.error("医生接口请求失败:", err);
          this.errorMsg = '医生信息加载失败，请重试';
        },
        complete: () => {
          this.isLoading = false;
        }
      });
    }
  }
};
</script>

<style scoped>
/* 页面整体背景 */
page {
  background-color: #f5f5f5;
}

.department-container {
  padding: 16px;
}

/* 页面标题（兼容返回按钮） */
.page-header {
  font-size: 20px;
  font-weight: 600;
  text-align: center;
  margin-bottom: 20px;
  color: #333;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}
.header-title {
  margin: 0 20px;
}
.page-header::after {
  content: '';
  display: block;
  width: 40px;
  height: 3px;
  background-color: #1890ff;
  margin: 8px auto 0;
  border-radius: 3px;
}

/* 医院选择器样式 */
.hospital-selector-container {
  padding: 0 24rpx;
  margin-bottom: 30rpx;
}
.hospital-selector {
  border: 1rpx solid #dcdfe6;
  border-radius: 12rpx;
  overflow: hidden;
  background-color: #fff;
  box-shadow: 0 2rpx 12rpx 0 rgba(0, 0, 0, 0.05);
}
.picker-display {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 24rpx;
  font-size: 30rpx;
}
.picker-label {
  color: #666;
  margin-right: 16rpx;
}
.picker-value {
  flex: 1;
  color: #333;
  font-size: 32rpx;
  font-weight: 500;
}
.picker-icon {
  color: #999;
}

/* 加载样式 */
.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80rpx 0;
}
.loading-spinner {
  width: 30px;
  height: 30px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #1890ff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 10px;
}
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* 错误提示样式 */
.error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80rpx 0;
  color: #ff4d4f;
}
.error-icon {
  font-size: 40px;
  margin-bottom: 16rpx;
}
.error-text {
  font-size: 28rpx;
}

/* 科室列表样式 */
.department-list {
  display: flex;
  flex-direction: column;
  gap: 16rpx;
  padding: 0 8rpx;
}
.department-item {
  background-color: #fff;
  padding: 24rpx;
  border-radius: 12rpx;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.04);
  transition: all 0.3s ease;
  cursor: pointer;
}
.department-item:active {
  transform: translateY(-2rpx);
  box-shadow: 0 6rpx 20rpx rgba(0, 0, 0, 0.08);
}

.dept-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 16rpx;
}
.dept-name {
  font-size: 32rpx;
  font-weight: 600;
  color: #1890ff;
}
.dept-location {
  display: flex;
  align-items: center;
  color: #666;
  font-size: 24rpx;
  background-color: #f0f7ff;
  padding: 4rpx 12rpx;
  border-radius: 16rpx;
}
.location-text {
  margin-left: 6rpx;
}

.dept-intro {
  font-size: 26rpx;
  color: #666;
  line-height: 1.6;
  margin-bottom: 20rpx;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.intro-label {
  color: #333;
  font-weight: 500;
}

.view-doctor-btn {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  color: #1890ff;
  font-size: 24rpx;
  padding-top: 12rpx;
  border-top: 1rpx solid #f0f0f0;
}
.view-doctor-btn text {
  margin-right: 6rpx;
}

/* 无数据样式 */
.no-data {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 120rpx 0;
}
.no-data-text {
  margin-top: 20rpx;
  color: #999;
  font-size: 28rpx;
}

/* 医生列表容器样式 */
.doctor-list-container {
  background-color: #fff;
  border-radius: 12rpx;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.04);
  padding: 16rpx;
  margin-top: 16rpx;
}
.doctor-list {
  height: calc(100vh - 200rpx); /* 占满剩余屏幕高度 */
}

/* 医生项样式 */
.doctor-item {
  padding: 24rpx 0;
  border-bottom: 1rpx solid #f5f5f5;
}
.doctor-item:last-child {
  border-bottom: none;
}
.doctor-info {
  margin-bottom: 16rpx;
}
.doctor-base {
  display: flex;
  align-items: center;
  gap: 16rpx;
  margin-bottom: 8rpx;
}
.doctor-name {
  font-size: 30rpx;
  font-weight: 600;
  color: #333;
}
.doctor-title {
  font-size: 24rpx;
  color: #666;
  background-color: #f0f7ff;
  padding: 4rpx 12rpx;
  border-radius: 14rpx;
}
.doctor-department {
  font-size: 22rpx;
  color: #999;
}
.doctor-desc {
  font-size: 26rpx;
  color: #666;
  line-height: 1.6;
}
.desc-label {
  color: #333;
  font-weight: 500;
}

/* 无医生数据样式 */
.no-doctor {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80rpx 0;
  color: #999;
}
.no-doctor-text {
  margin-top: 20rpx;
  font-size: 28rpx;
}
</style>

