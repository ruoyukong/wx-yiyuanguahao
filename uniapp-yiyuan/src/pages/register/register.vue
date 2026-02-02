<template>
  <view class="register-container">
    <!-- 页面标题 -->
    <view class="page-title">预约挂号</view>
  <!-- 新增：全局搜索区域 -->
    <view class="global-search-area" v-if="hospitalSelected">
      <view class="search-bar">
        <input 
          class="search-input" 
          type="text" 
          placeholder="搜索门诊..." 
          v-model="searchKeyword"
          @input="handleSearchInput"
          @focus="handleSearchFocus"
         
        />
        <view class="search-action-btn" @click="handleSearchAction">
          {{ showClearBtn ? '清空' : '搜索' }}
        </view>
      </view>

      <!-- 新增：搜索结果展示区域 -->
      <view class="search-results" v-if="showSearchResults && searchedDiseases.length > 0">
        <view class="search-results-title">搜索结果 ({{ searchedDiseases.length }})</view>
        <view class="disease-item" v-for="(disease, index) in searchedDiseases" :key="index">
          <view class="disease-header">
            <view class="disease-name">{{ disease.disease_name }}</view>
            <view class="appoint-btn" @click="handleDiseaseAppointment(disease)">
              预约挂号
            </view>
          </view>
          <view class="doctor-group">
            <view class="doctor-item" v-for="(doctor, docIndex) in disease.doctors" :key="docIndex">
              <view class="doctor-info">
                <view class="doctor-name">{{ doctor.name }} <text class="doctor-title">{{ doctor.title }}</text></view>
                <view class="doctor-good-at">擅长: {{ doctor.good_at }}</view>
              </view>
            </view>
          </view>
        </view>
      </view>
      <view class="search-results" v-if="showSearchResults && searchedDiseases.length === 0 && searchKeyword.trim() !== ''">
        <view class="no-results">未找到匹配的疾病</view>
      </view>
    </view>
    <!-- 初始状态：选择医院 -->
    <view class="hospital-selection" v-if="!hospitalSelected">
      <view class="selection-prompt">请选择您要预约的医院</view>
      <view class="hospital-buttons">
        <view 
          class="hospital-btn" 
          v-for="hospital in hospitalList" 
          :key="hospital.id"
          @click="selectHospital(hospital.id, hospital.index)"
        >
          {{ hospital.name }}
        </view>
      </view>
    </view>

    <!-- 选择后状态：科室导航 -->
    <view class="main-content" v-else>
      
      <!-- 左侧：科室导航栏 -->
      <view class="department-nav">
        <view class="nav-header">科室列表</view>
        <scroll-view scroll-y class="department-scroll">
          <view 
            class="department-item" 
            v-for="dept in departmentList" 
            :key="dept.id"
            :class="{ 'active': selectedDepartmentId === dept.id }"
            @click="handleDepartmentClick(dept.id)"
          >
            {{ dept.name }}
          </view>
        </scroll-view>
      </view>

      <!-- 右侧：疾病与医生列表 (修改后) -->
      <view class="disease-list">
       
       
        <!-- 正在加载提示 -->
        <view class="loading-state" v-if="isLoadingDisease">
          <uni-icons type="loading" size="40" color="#007aff"></uni-icons>
          <text>正在加载信息...</text>
        </view>

        <!-- 疾病列表 (无表头) -->
        <view v-else-if="diseaseList.length > 0">
          <view class="disease-item" v-for="(disease, diseaseIndex) in diseaseList" :key="diseaseIndex">
           <view class="disease-header">
              <view class="disease-name">{{ disease.disease_name }}</view>
              <view class="appoint-btn" @click="handleDiseaseAppointment(disease)">
                                     预约挂号
              </view>
            </view>
            <!-- 对应医生列表 -->
            <view class="doctor-group">
              <view class="doctor-item" v-for="(doctor, doctorIndex) in disease.doctors" :key="doctorIndex">
                
                <view class="doctor-info">
                  <view class="doctor-name">{{ doctor.name }} <text class="doctor-title">{{ doctor.title }}</text></view>
                  <view class="doctor-good-at">擅长: {{ doctor.good_at }}</view>
                </view>
                
              </view>
            </view>
          </view>
        </view>

        <!-- 无数据提示 -->
        <view class="empty-state" v-else-if="selectedDepartmentId && !isLoadingDisease">
          <text>该科室暂无数据</text>
        </view>
         <!-- 无数据提示 (修改：根据过滤结果显示) -->
         <view class="empty-state" v-else-if="!isLoadingDisease">
          <text>
          {{ searchKeyword ? '未找到匹配的门诊' : '请先从左侧选择一个科室' }}
         </text>
        </view>
      </view>
    </view>
  </view>
</template>
<script>
export default {
  data() {
    return {
      // 医院相关
      hospitalList: [
        { name: 'dgut市第一人民医院', id: 1, index: 0 },
        { name: 'dgut市中医院', id: 2, index: 1 }
      ],
      selectedHospitalId: null,
      selectedHospitalIndex: null,
      hospitalSelected: false,

      // 科室相关
      departmentList: [],
      selectedDepartmentId: null,

      // 疾病与医生相关
      diseaseList: [],         // 当前科室的疾病列表
      allDiseases: [],         // 新增：存储所有科室的疾病列表
      isLoadingDisease: false, // 加载当前科室疾病的状态
      isLoadingAllDiseases: false, // 新增：加载所有科室疾病的状态

      // 搜索相关
      searchKeyword: '',
      showClearBtn: false,
      showSearchResults: false,
    };
  },
  computed: {
    searchedDiseases() {
      // 如果没有搜索关键字，直接返回空数组
      if (!this.searchKeyword.trim()) {
        return [];
      }

      const keyword = this.searchKeyword.toLowerCase();

      // 关键修改：从 allDiseases 而不是 diseaseList 中搜索
      return this.allDiseases.filter(disease => {
        return disease && disease.disease_name && disease.disease_name.toLowerCase().includes(keyword);
      });
    }
  },
  methods: {
    /**
     * 用户点击医院选择按钮时触发
     * @param {number} hospitalId - 选择的医院ID
     * @param {number} index - 选择的医院在数组中的索引
     */
    selectHospital(hospitalId, index) {
      this.selectedHospitalId = hospitalId;
      this.selectedHospitalIndex = index;
      this.hospitalSelected = true;

      // 重置数据状态
      this.departmentList = [];
      this.diseaseList = [];
      this.allDiseases = [];
      this.searchKeyword = '';
      this.showSearchResults = false;

      // 开始加载所有数据，并显示全局加载提示
      this.isLoadingAllDiseases = true;
      uni.showLoading({
        title: '正在加载所有科室数据...',
        mask: true // 添加遮罩，防止用户操作
      });

      // 第一步：加载科室列表
      this.fetchDepartmentList().then(() => {
        // 第二步：遍历科室，并行加载所有疾病数据
        if (this.departmentList.length === 0) {
          this.isLoadingAllDiseases = false;
          uni.hideLoading();
          return;
        }

        const diseasePromises = this.departmentList.map(dept => {
          // 为每个科室ID创建一个Promise
          return new Promise((resolve) => {
            this.fetchDiseaseList(dept.id, resolve);
          });
        });

        // 等待所有科室的疾病数据加载完成
        Promise.all(diseasePromises).then(() => {
          // 所有数据加载完毕
          this.isLoadingAllDiseases = false;
          uni.hideLoading();
        });

      }).catch(() => {
        // 如果科室列表加载失败，也隐藏加载提示
        this.isLoadingAllDiseases = false;
        uni.hideLoading();
      });
    },

    /**
     * 请求后端，获取指定医院的科室列表
     * @returns {Promise} 返回一个Promise，用于串联操作
     */
    fetchDepartmentList() {
      return new Promise((resolve, reject) => {
        wx.request({
          url: `http://localhost/api/department/list/hosp/${this.selectedHospitalId}`,
          method: 'GET',
          success: (res) => {
            const { code, msg, data } = res.data;
            if (code === 200) {
              this.departmentList = data || [];
              resolve(); // 科室列表加载成功，resolve Promise
            } else {
              uni.showToast({
                title: `获取科室失败: ${msg}`,
                icon: 'none'
              });
              reject(); // 科室列表加载失败，reject Promise
            }
          },
          fail: (err) => {
            console.error("获取科室接口请求失败:", err);
            uni.showToast({
              title: '网络错误，无法获取科室',
              icon: 'none'
            });
            reject(); // 网络错误，reject Promise
          }
        });
      });
    },

    /**
     * 请求后端，获取指定科室的疾病列表
     * @param {number} deptId - 科室ID
     * @param {Function} callback - 回调函数，用于通知Promise任务完成
     */
    fetchDiseaseList(deptId, callback) {
      // 如果是为当前选中的科室加载数据，则显示局部加载状态
      if (deptId === this.selectedDepartmentId) {
        this.isLoadingDisease = true;
      }

      wx.request({
        url: `http://localhost/api/disease/list/:dept/${deptId}`, // 保持你原来的URL
        method: 'GET',
        success: (res) => {
          if (res.data.code === 200) {
            const diseaseData = res.data.data || [];

            // 将新获取的疾病数据合并到 allDiseases 中
            this.allDiseases = [...this.allDiseases, ...diseaseData];

            // 如果是当前选中的科室，则同时更新 diseaseList
            if (deptId === this.selectedDepartmentId) {
              this.diseaseList = diseaseData;
            }
          } else {
            // 如果是当前选中的科室加载失败，则清空 diseaseList
            if (deptId === this.selectedDepartmentId) {
              this.diseaseList = [];
            }
          }
        },
        fail: () => {
          console.error(`获取科室 ${deptId} 疾病数据失败`);
          // 如果是当前选中的科室加载失败，则清空 diseaseList
          if (deptId === this.selectedDepartmentId) {
            this.diseaseList = [];
            uni.showToast({ title: '网络错误，无法获取疾病数据', icon: 'none' });
          }
        },
        complete: () => {
          // 如果是为当前选中的科室加载数据，则隐藏局部加载状态
          if (deptId === this.selectedDepartmentId) {
            this.isLoadingDisease = false;
          }
          // 无论成功失败，都调用回调函数通知Promise任务完成
          if (callback) {
            callback();
          }
        },
      });
    },

    /**
     * 用户点击科室时触发
     * @param {number} deptId - 科室ID
     */
    handleDepartmentClick(deptId) {
      this.selectedDepartmentId = deptId;
      this.showSearchResults = false; // 切换科室时隐藏搜索结果

      // 先检查 allDiseases 中是否已经有该科室的数据
      const cachedDiseases = this.allDiseases.filter(disease => disease.dept_id === deptId);

      if (cachedDiseases.length > 0) {
        // 如果有缓存，则直接使用缓存数据，无需再次请求
        this.diseaseList = cachedDiseases;
      } else {
        // 如果没有缓存，则显示加载状态并请求数据
        this.fetchDiseaseList(deptId);
      }
    },

    // --- 以下是搜索相关的方法，无需修改 ---
    handleSearchInput() {
      this.showClearBtn = this.searchKeyword.trim() !== '';
    },
    handleSearchFocus() {
      this.showClearBtn = this.searchKeyword.trim() !== '';
      this.showSearchResults = true;
    },
    handleSearchAction() {
      if (this.showClearBtn) {
        this.searchKeyword = '';
        this.showClearBtn = false;
        this.showSearchResults = false;
      } else {
        if (this.searchKeyword.trim()) {
          this.showSearchResults = true;
        }
      }
    },
    handleDiseaseAppointment(disease) {
      const firstDoctorId = disease.doctors?.[0]?.id;
      const diseaseName = disease.disease_name;
      const deptId = disease.dept_id;
     if (diseaseName&& deptId) {
    // 将两个参数都传递给 goToSchedule 方法
    this.goToSchedule(diseaseName, deptId);
  } else {
    uni.showToast({ title: '缺少疾病或科室信息，无法预约', icon: 'none' });
  }
    },
    goToSchedule(diseaseName,deptId) {
  uni.navigateTo({
    // 使用模板字符串拼接 URL 参数
   url: `/pages/schedule/schedule?diseaseName=${diseaseName}&deptId=${deptId}`
  });
}

},
};
</script>

<style scoped>
/* 页面整体背景
   注意：我们将在 .hospital-selection 中控制选择页面的背景，这里保持中性 */
.register-container {
  background-color: #f5f5f5;
  min-height: 100vh;
  padding: 30rpx;
  box-sizing: border-box; /* 确保 padding 不会撑大页面 */
}

/* 页面标题 - 放大字体 */
.page-title {
  font-size: 44rpx; /* 放大 */
  font-weight: bold;
  text-align: center;
  margin-bottom: 50rpx;
  color: #333;
}

/* --- 初始状态：选择医院 --- */
/* 整个选择页面的容器，设置底部为纯白 */
.hospital-selection {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start; /* 从顶部开始布局 */
  min-height: calc(100vh - 60rpx); /* 填满剩余高度 */
  background: linear-gradient(to bottom, #f5f5f5 0%, #ffffff 100%);
  border-radius: 30rpx; /* 给整个区域添加一个圆角，视觉更柔和 */
  padding-top: 100rpx; /* 顶部留出一些空间 */
}

/* 选择提示文本 - 放大字体 */
.selection-prompt {
  font-size: 36rpx; /* 放大 */
  color: #333;
  margin-bottom: 60rpx;
  font-weight: 500;
}

/* 医院按钮容器 */
.hospital-buttons {
  display: flex;
  flex-direction: column;
  gap: 30rpx; /* 增大按钮间距 */
  width: 85%;
  max-width: 600rpx;
}

/* 医院按钮 - 透明灰色立体阴影样式 */
.hospital-btn {
  /* 透明灰色背景 */
  background-color: hwb(0 100% 0% / 0.905);
  /* 黑色字体 */
  color: #000000;
  font-size: 38rpx; /* 放大字体 */
  font-weight: 500;
  text-align: center;
  padding: 40rpx 30rpx; /* 增大内边距，让按钮更大 */
  border-radius: 20rpx; /* 圆润的圆角 */
  
  /* 立体阴影效果：使用多个阴影叠加 */
  box-shadow: 
    8rpx 8rpx 15rpx rgba(0, 0, 0, 0.1), /* 右下阴影，营造立体感 */
    -8rpx -8rpx 15rpx rgba(255, 255, 255, 0.8); /* 左上高光，增强立体感 */
    
  transition: all 0.2s ease-in-out;
  border: 1rpx solid rgba(255, 255, 255, 0.5); /* 细边框增强质感 */
}

/* 按钮点击/激活状态 */
.hospital-btn:active {
  /* 点击时，阴影反向，营造凹陷感 */
  box-shadow: 
    inset 5rpx 5rpx 10rpx rgba(0, 0, 0, 0.1),
    inset -5rpx -5rpx 10rpx rgba(255, 255, 255, 0.7);
  /* 文字略微缩小，增强反馈 */
  transform: scale(0.98);
}

/* --- 选择后状态：科室导航 --- */
.main-content {
  display: flex;
  height: calc(100vh - 150rpx);
  background-color: #fff; /* 右侧内容区背景设为白色 */
  border-radius: 30rpx;
  overflow: hidden; /* 确保内部元素不会超出圆角 */
  box-shadow: 0 5rpx 20rpx rgba(0,0,0,0.05);
}

/* 左侧科室导航 */
.department-nav {
  width: 220rpx; /* 适当增宽 */
  background-color: #f8f8f8;
  border-right: 1rpx solid #f0f0f0;
  display: flex;
  flex-direction: column;
}

/* 科室导航标题 - 放大字体 */
.nav-header {
  padding: 25rpx;
  font-size: 34rpx; /* 放大 */
  font-weight: bold;
  color: #333;
  border-bottom: 1rpx solid #f0f0f0;
  text-align: center;
}

.department-scroll {
  flex: 1;
  height: 100%;
}

/* 科室选项 - 放大字体 */
.department-item {
  padding: 30rpx 20rpx; /* 增大内边距 */
  font-size: 30rpx; /* 放大 */
  color: #555;
  border-bottom: 1rpx solid #f0f0f0;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.department-item.active {
  background-color: #fff;
  color: #007aff;
  font-weight: 500;
  border-left: 5rpx solid #007aff;
}

/* 右侧医生列表 */
.doctor-list {
  flex: 1;
  padding: 40rpx; /* 增大内边距 */
  overflow-y: auto;
}



/* 空状态提示 - 放大字体 */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 80%;
  color: #999;
  font-size: 32rpx; /* 放大 */
}

/* 空状态图标 - 放大 */
.empty-state uni-icons {
  font-size: 100rpx;
  margin-bottom: 20rpx;
}
/* --- 右侧：疾病与医生列表 (新增) --- */
.disease-list {
  flex: 1;
  padding: 30rpx;
  overflow-y: auto;
  background-color: #fafafa;
}

/* 加载状态 */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 300rpx;
  color: #999;
  font-size: 28rpx;
}
.loading-state uni-icons {
  margin-bottom: 15rpx;
  animation: loading 1s linear infinite;
}
@keyframes loading {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* 疾病项容器 */
.disease-item {
  background-color: #fff;
  border-radius: 20rpx;
  padding: 30rpx;
  margin-bottom: 25rpx;
  box-shadow: 0 2rpx 15rpx rgba(0, 0, 0, 0.05);
}

/* 疾病标题栏（包含名称和预约按钮）(新增) */
.disease-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
}

/* 疾病名称 (修改) */
.disease-name {
  font-size: 34rpx;
  font-weight: bold;
  color: #333;
  padding-left: 10rpx;
  border-left: 6rpx solid #007aff;
}

/* 医生组容器 */
.doctor-group {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}

/* 单个医生项 (修改) */
.doctor-item {
  display: flex;
  align-items: center;
  padding: 15rpx;
  border-bottom: 1rpx solid #f0f0f0;
}
.doctor-item:last-child {
  border-bottom: none;
}
/* 医生信息 (修改：因为没有了头像，让信息容器占满一行) */
.doctor-info {
  flex: 1;
  /* 为了让布局更美观，可以添加一些左边距，模拟原来头像的位置 */
  padding-left: 10rpx; 
}

/* 医生信息 */
.doctor-info {
  flex: 1;
}
.doctor-name {
  font-size: 30rpx;
  color: #333;
  font-weight: 500;
}
.doctor-title {
  font-size: 24rpx;
  color: #999;
  margin-left: 10rpx;
}
.doctor-good-at {
  font-size: 26rpx;
  color: #666;
  margin-top: 5rpx;
}

/* 预约按钮 */
.appoint-btn {
  font-size: 26rpx;
  color: #007aff;
  background-color: rgba(0, 122, 255, 0.1);
  padding: 12rpx 25rpx;
  border-radius: 40rpx;
  font-weight: 500;
}
.appoint-btn:active {
  background-color: rgba(0, 122, 255, 0.2);
}

/* 空状态提示 (微调) */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  /* 保持原高度，或根据需要调整 */
  min-height: 400rpx; 
  color: #999;
  font-size: 32rpx;
}
/* 新增：全局搜索区域样式 */
.global-search-area {
  padding: 0 30rpx 20rpx; /* 左右和下方留出边距 */
}
.search-bar {
  display: flex;
  align-items: center;
  background-color: #ffffff;
  border-radius: 60rpx; /* 圆角输入框 */
  padding: 0 20rpx;
  box-shadow: 0 2rpx 15rpx rgba(0, 0, 0, 0.05);
}
.search-input {
  flex: 1; /* 让输入框占满剩余空间 */
  height: 72rpx;
  padding: 0 20rpx;
  font-size: 28rpx;
  color: #333;
}
.search-input:focus {
  outline: none;
}
.search-action-btn {
  font-size: 28rpx;
  font-weight: 500;
  color: #007aff;
  padding: 0 20rpx;
  height: 100%;
  display: flex;
  align-items: center;
}

/* 正在加载提示 */
.loading-state {
  /* 需要给 loading-state 一个高度，或者让它占据 flex 空间 */
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  /* 让 loading 区域占满剩余空间，使其垂直居中 */
  flex-grow: 1;
  color: #999;
  font-size: 28rpx;
}
/* 空状态提示 (修改：同样让它占满空间) */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  flex-grow: 1; /* 占满剩余空间 */
  color: #999;
  font-size: 32rpx;
}
/* 新增：搜索结果区域样式 */
.search-results {
  margin-top: 20rpx;
  background-color: #fff;
  border-radius: 20rpx;
  padding: 20rpx;
  box-shadow: 0 2rpx 15rpx rgba(0, 0, 0, 0.05);
}
.search-results-title {
  font-size: 26rpx;
  color: #999;
  margin-bottom: 15rpx;
  padding-left: 10rpx;
}
.no-results {
    text-align: center;
    padding: 40rpx;
    color: #999;
    font-size: 28rpx;
}

/* 新增：内容容器样式 */
.content-container {
  padding: 0 30rpx 30rpx;
}

/* 调整主内容区域的顶部边距，因为搜索结果在上面了 */
.main-content {
  display: flex;
  height: calc(100vh - 250rpx); /* 根据实际情况调整高度 */
  background-color: #fff;
  border-radius: 30rpx;
  overflow: hidden;
  box-shadow: 0 5rpx 20rpx rgba(0,0,0,0.05);
}
</style>