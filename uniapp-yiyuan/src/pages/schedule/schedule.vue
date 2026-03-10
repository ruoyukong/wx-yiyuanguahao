<template>
  <view class="schedule-container">
    <!-- 顶部导航栏标题 -->
    <view class="page-header">选择就诊{{diseaseName}}时间</view>

    <!-- 横向日期选择器 -->
    <scroll-view class="date-scroll" scroll-x="true">
      <view 
        class="date-item" 
        v-for="(date, index) in dateList" 
        :key="index"
        :class="{ 'active': selectedDateIndex === index, 'past': date.isPast }"
        @click="selectDate(index)"
      >
        <view class="week-day">{{ date.weekDay }}</view>
        <view class="date">{{ date.date }}</view>
        <view class="next-week-tag" v-if="date.isNextWeek">下周</view>
      </view>
    </scroll-view>

    <!-- 主内容区：显示选中日期的排班信息 -->
    <view class="content">
      <!-- 加载中状态 -->
      <view class="loading" v-if="isLoading">
        <uni-icons type="loading" size="40" color="#007aff"></uni-icons>
        <view class="loading-text">正在加载排班信息...</view>
      </view>

      <!-- 排班列表 (无论有无数据，都会显示这3个时间段) -->
      <view class="schedule-list" v-if="!isLoading">
        <view class="time-slot" v-for="(slot, slotIndex) in schedules" :key="slotIndex">
          <view class="time-slot-title">{{ slot.time_slot }}</view>
          
          <view class="doctor-list">
            <!-- 时间段内无医生时显示 -->
            <view class="no-doctor" v-if="!slot.doctors || slot.doctors.length === 0">
              <uni-icons type="none" size="40" color="#cccccc"></uni-icons>
              <view class="no-doctor-text">该时段暂无医生出诊</view>
            </view>

            <!-- 医生列表 -->
            <view class="doctor-item" v-for="(item, itemIndex) in slot.doctors" :key="itemIndex">
              <view class="doctor-info">
                <view class="doctor-details">
                  <view class="doctor-name">{{ item.doctor.name }} <text class="doctor-title">{{ item.doctor.title }}</text></view>
                  <view class="doctor-good-at">擅长: {{ item.doctor.good_at }}</view>
                  <view class="price">¥{{ item.price }}</view>
                </view>
              </view>
              <view class="appointment-status">
                <view class="remaining">{{ item.remaining }}/{{ item.total }}个号</view>
                <button 
                  class="appointment-btn" 
                  :disabled="item.remaining <= 0"
                  :class="{ 'disabled': item.remaining <= 0 }"
                  @click="handleAppointment(item)"
                >
                  {{ item.remaining > 0 ? '预约' : '已约满' }}
                </button>
              </view>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      deptId: null,
      dateList: [], // 日期列表
      selectedDateIndex: 0, // 当前选中的日期索引
      schedules: [], // 选中日期的排班信息
      isLoading: false, // 加载状态
      diseaseName: null,
      // 1. 定义3个前端标准时段 (value用于匹配后端, label用于显示)
      standardTimeSlots: [
        { value: '8-10', label: '8:00-10:00' },
        { value: '10-12', label: '10:00-12:00' },
        { value: '14-17', label: '14:00-17:00' }
      ]
    };
  },
  onLoad(option) {
    // 从URL获取科室ID
    const diseaseName = decodeURIComponent(option.diseaseName);
    this.diseaseName = diseaseName;
    if (option.deptId) {
      this.deptId = option.deptId;
      this.init();
    } else {
      uni.showToast({ title: '参数错误', icon: 'none' });
      setTimeout(() => {
        uni.navigateBack();
      }, 1500);
    }
  },
  methods: {
    /**
     * 页面初始化
     */
    init() {
      this.generateDateList();
      this.fetchScheduleData();
    },

    /**
     * 格式化日期为 YYYY-MM-DD 标准格式
     */
    formatDateToYmd(dateObj) {
      const year = dateObj.getFullYear();
      const month = String(dateObj.getMonth() + 1).padStart(2, '0'); // 月份补0（1→01）
      const day = String(dateObj.getDate()).padStart(2, '0'); // 日期补0（5→05）
      return `${year}-${month}-${day}`;
    },

    // 生成日期列表（修正下周判定+当天不可预约）
   // 修改 generateDateList 方法：
generateDateList() {
  const weekDays = ['周一', '周二', '周三', '周四', '周五', '周六', '周日'];
  const today = new Date();
  const todayYear = today.getFullYear();
  const todayMonth = today.getMonth();
  const todayDate = today.getDate();
  
  // 创建今天的0点时间戳
  const todayZero = new Date(todayYear, todayMonth, todayDate).getTime();
  
  // 获取今天是周几（0=周日，1=周一...6=周六）
  const todayDayOfWeek = today.getDay();
  
  this.dateList = [];
  
  // 从明天开始，只显示未来6天（总共6个选项）
  for (let i = 1; i <= 6; i++) {
    const date = new Date(todayYear, todayMonth, todayDate + i);
    const dateTimestamp = date.getTime();
    const dayOfWeek = date.getDay(); // 该日期是周几
    const weekDayText = weekDays[dayOfWeek === 0 ? 6 : dayOfWeek - 1]; // 转换为周一到周日的显示
    
    // 计算日期显示格式
    const displayDate = `${date.getMonth() + 1}月${date.getDate()}日`;
    const ymd = this.formatDateToYmd(date);
    
    // 判断是否是下周
    let isNextWeek = false;
    
    // 规则：下周从下周一算起
    // 如果今天是周日（0），那么明天就是周一，应该算下周
    if (todayDayOfWeek === 0 && i === 1) {
      isNextWeek = true; // 周一的明天是下周一
    } 
    // 如果今天不是周日，检查日期是否在下周一或之后
    else if (todayDayOfWeek !== 0) {
      // 计算距离下周一还有多少天
      const daysUntilNextMonday = todayDayOfWeek === 1 ? 7 : 8 - todayDayOfWeek;
      isNextWeek = i >= daysUntilNextMonday;
    }
    
    this.dateList.push({
      weekDay: weekDayText,
      date: displayDate,
      ymd: ymd,
      timestamp: dateTimestamp,
      isPast: false, // 未来日期不会过去
      isToday: false, // 不是今天（我们从明天开始）
      isCurrentWeek: !isNextWeek,
      isNextWeek: isNextWeek
    });
  }
  
  // 默认选中第一个（明天）
  this.selectedDateIndex = 0;
},

// 修改 selectDate 方法，移除过去和当天的判断（因为列表里已经没有过去和当天的日期了）
selectDate(index) {
  if (this.selectedDateIndex !== index) {
    this.selectedDateIndex = index;
    this.fetchScheduleData();
  }
},
    /**
     * 根据职称计算价格
     */
    calculatePrice(title) {
      if (!title) return 30;
      if (title.includes('主任医师')) return 100;
      if (title.includes('副主任医师')) return 60;
      return 30;
    },

    /**
     * 请求排班数据
     */
    fetchScheduleData() {
      this.isLoading = true;
      this.schedules = []; // 清空旧数据

      // --------------------------
      // 使用真实网络请求
      // --------------------------
      wx.request({
        url: `http://localhost/api/schedule/list/:dept/${this.deptId}`, // 假设URL已修正
        method: 'GET',
        success: (res) => {
          if (res.data.code === 200) {
            this.processScheduleData(res.data.data);
          } else {
            // 如果获取数据失败，也初始化schedules，保证3个时间段正常显示
            this.processScheduleData([]);
            uni.showToast({ title: res.data.msg || '获取排班失败', icon: 'none' });
          }
        },
        fail: () => {
          // 如果请求失败，同样初始化schedules
          this.processScheduleData([]);
          uni.showToast({ title: '网络请求失败，请重试', icon: 'none' });
        },
        complete: () => {
          this.isLoading = false;
        }
      });
    },

    /**
     * 处理并格式化排班数据
     * 核心逻辑：遍历前端标准时段，用后端数据填充
     */
    processScheduleData(rawData) {
      const selectedWeekDay = this.dateList[this.selectedDateIndex].weekDay;
      
      // 1. 先对后端返回的数据进行预处理，按 [week_day][time_slot] 分组，方便后续查找
      const scheduleMap = {};
      rawData.forEach(item => {
        // 只处理当前选中星期几的数据
        if (item.week_day === selectedWeekDay) {
          if (!scheduleMap[item.time_slot]) {
            scheduleMap[item.time_slot] = [];
          }
          const remaining = Math.max(0, item.total - item.used);
          const price = this.calculatePrice(item.doctor?.title);
          scheduleMap[item.time_slot].push({
            ...item,
            remaining,
            price,
            scheduleId: item.id // 保存排班id
          });
        }
      });

      // 2. 遍历前端定义的标准时段，构建最终的 schedules 数组
      this.schedules = this.standardTimeSlots.map(slot => {
        // 从预处理好的 map 中查找对应的数据，如果找不到则为空数组
        const doctors = scheduleMap[slot.value] || [];
        
        return {
          time_slot: slot.label, // 显示用的标签
          doctors: doctors       // 填充的医生数据（包含scheduleId）
        };
      });
    },

    /**
     * 点击预约按钮
     */
    handleAppointment(item) {
      if (item.remaining > 0) {
        // 获取选中日期的 标准格式（YYYY-MM-DD）和 显示格式（11月28日）
        const selectedDateObj = this.dateList[this.selectedDateIndex];
        const visitDate = selectedDateObj.ymd; // 如 2025-11-28（用于存储）
        const showDate = selectedDateObj.date; // 如 11月28日（用于前端显示，可选传递）
        
        // 处理其他参数编码（保留原有逻辑）
        let diseaseNameToPass = this.diseaseName || '';
        const isEncoded = /%[0-9A-Fa-f]{2}/.test(diseaseNameToPass) && !/[\u4e00-\u9fa5]/.test(diseaseNameToPass);
        if (!isEncoded && diseaseNameToPass) {
          diseaseNameToPass = encodeURIComponent(diseaseNameToPass);
        }
        
        // 拼接 URL，增加 visitDate（必传）和 showDate（可选，用于确认页显示）
        uni.navigateTo({
          url: `/pages/appointmentConfirm/appointmentConfirm?scheduleId=${item.scheduleId}&doctorName=${item.doctor.name}&timeSlot=${encodeURIComponent(item.time_slot)}&price=${item.price}&deptId=${encodeURIComponent(this.deptId)}&diseaseName=${diseaseNameToPass}&visitDate=${visitDate}&showDate=${encodeURIComponent(showDate)}`
        });
      } else {
        uni.showToast({ title: '该时段已约满', icon: 'none' });
      }
    }
  }
}
</script>

<style scoped>
/* 页面整体背景 */
.schedule-container {
  background-color: #f5f5f5;
  min-height: 100vh;
  padding-bottom: 20rpx;
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

/* 日期滚动条 */
.date-scroll {
  white-space: nowrap;
  background-color: #fff;
  padding: 20rpx 0;
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

/* 单个日期项 */
.date-item {
  display: inline-block;
  width: 120rpx;
  text-align: center;
  margin: 0 10rpx;
  padding: 15rpx 0;
  border-radius: 15rpx;
  background-color: #f0f0f0;
}

/* 选中的日期项 */
.date-item.active {
  background-color: #007aff;
  color: #fff;
}

/* 过去的日期项 */
.date-item.past {
  opacity: 0.5;
}

.date-item .week-day {
  font-size: 28rpx;
  font-weight: 500;
}

.date-item .date {
  font-size: 24rpx;
  margin-top: 5rpx;
}

/* 下周标签 */
.next-week-tag {
  font-size: 20rpx;
  color: #ff5722;
  margin-top: 5rpx;
  font-weight: bold;
}

/* 内容区通用样式 */
.content {
  padding: 20rpx;
}

/* 加载中样式 */
.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 100rpx 0;
}
.loading-text {
  margin-top: 20rpx;
  font-size: 26rpx;
  color: #999;
}

/* 排班列表样式 */
.schedule-list {
  background-color: #fff;
  border-radius: 20rpx;
  overflow: hidden;
}

/* 时间段标题样式 */
.time-slot-title {
  font-size: 28rpx;
  font-weight: 500;
  padding: 25rpx 30rpx;
  border-bottom: 1rpx solid #f0f0f0;
  background-color: #fafafa;
}

/* 医生列表容器 */
.doctor-list {
  padding: 20rpx;
}

/* 时间段内无医生时的样式 */
.no-doctor {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60rpx 0;
  color: #999;
  font-size: 26rpx;
}
.no-doctor-text {
  margin-top: 15rpx;
}

/* 单个医生项 */
.doctor-item {
  display: flex;
  align-items: center;
  padding: 15rpx 0;
  border-bottom: 1rpx solid #f5f5f5;
}
.doctor-item:last-child {
  border-bottom: none;
}

/* 医生详情 */
.doctor-details {
  flex-grow: 1;
  margin-left: 40rpx;
}
.doctor-info {
  flex-grow: 1;
}
.doctor-name {
  font-size: 28rpx;
  font-weight: 500;
}
.doctor-title {
  font-size: 24rpx;
  color: #007aff;
  background-color: #e8f4ff;
  padding: 2rpx 10rpx;
  border-radius: 15rpx;
  margin-left: 10rpx;
}
.doctor-good-at {
  font-size: 24rpx;
  color: #666;
  margin-top: 5rpx;
}
.price {
  font-size: 26rpx;
  color: #ff5722;
  font-weight: bold;
  margin-top: 8rpx;
}

/* 预约状态区域 */
.appointment-status {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}
.remaining {
  font-size: 22rpx;
  color: #999;
  margin-bottom: 10rpx;
}
.appointment-btn {
  width: 140rpx;
  height: 56rpx;
  line-height: 56rpx;
  font-size: 24rpx;
  border-radius: 30rpx;
  background-color: #007aff;
  color: #fff;
}
.appointment-btn.disabled {
  background-color: #cccccc;
  color: #fff;
}
</style>