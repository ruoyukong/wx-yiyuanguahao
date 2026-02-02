<template>
  <view class="agreement-container">
    <!-- 顶部标题栏 -->
    <view class="agreement-header">
      <text class="header-title">{{ title }}</text>
    </view>
    
    <!-- 协议内容滚动区 -->
    <scroll-view class="agreement-content" scroll-y>
      <!-- 根据类型显示对应协议文本 -->
      <view v-if="type === 'user'" class="content-text">
        <h3>用户授权协议</h3>
        <p>一、授权范围</p>
        <p>1. 您同意授权本医院平台获取您的微信昵称、头像等公开信息，用于用户身份识别及服务提供。</p>
        <p>2. 平台仅在提供医疗服务范围内使用您的个人信息，不会用于其他商业用途。</p>
        
        <p>二、用户权利</p>
        <p>1. 您有权查看、修改个人信息，或申请注销账号。</p>
        <p>2. 您有权拒绝提供非必要的个人信息，但可能影响部分服务的使用。</p>
        
        <p>三、用户义务</p>
        <p>1. 您应提供真实、准确的个人信息，若信息变更需及时更新。</p>
        <p>2. 您应妥善保管账号信息，不得转借他人使用，对账号操作承担相应责任。</p>
        
        <p>四、协议变更</p>
        <p>平台有权根据业务调整更新本协议，更新后将通过公告形式通知您，继续使用服务即视为同意变更后的协议。</p>
      </view>

      <view v-if="type === 'privacy'" class="content-text">
        <h3>隐私政策</h3>
        <p>一、信息收集范围</p>
        <p>1. 基础信息：微信昵称、头像、openid等用于身份验证的信息。</p>
        <p>2. 服务相关信息：挂号记录、缴费记录、就诊信息等医疗服务产生的信息。</p>
        <p>3. 设备信息：手机型号、系统版本等用于优化服务的设备相关信息。</p>
        
        <p>二、信息使用方式</p>
        <p>1. 用于提供医疗服务：如挂号预约、缴费结算、就诊提醒等。</p>
        <p>2. 用于服务优化：根据使用数据改善平台功能和用户体验。</p>
        <p>3. 法律法规要求：在法律规定范围内披露或使用相关信息。</p>
        
        <p>三、信息保护措施</p>
        <p>1. 采用加密技术存储和传输个人信息，防止信息泄露。</p>
        <p>2. 严格限制信息访问权限，仅授权人员可接触相关信息。</p>
        <p>3. 定期进行安全检测，及时修复潜在安全漏洞。</p>
        
        <p>四、信息留存期限</p>
        <p>个人信息将在服务需要的范围内留存，账号注销后将删除或匿名化处理相关信息。</p>
      </view>

      <view v-if="type === 'service'" class="content-text">
        <h3>服务协议</h3>
        <p>一、服务内容</p>
        <p>1. 挂号服务：提供医院科室、医生的在线预约挂号服务。</p>
        <p>2. 缴费服务：提供门诊、住院等费用的在线支付服务。</p>
        <p>3. 记录查询：提供挂号记录、缴费记录等个人医疗相关记录的查询服务。</p>
        <p>4. 信息通知：提供就诊提醒、缴费通知等服务相关信息推送。</p>
        
        <p>二、服务使用规则</p>
        <p>1. 您应按照平台提示正确操作，不得通过非法手段获取或使用服务。</p>
        <p>2. 挂号成功后应按时就诊，若无法按时就诊需提前取消预约，多次爽约可能影响后续挂号资格。</p>
        <p>3. 在线缴费后，费用将直接转入医院账户，退款将按医院相关规定处理。</p>
        
        <p>三、服务免责声明</p>
        <p>1. 平台仅提供技术服务，医疗诊断和治疗由医院专业人员负责。</p>
        <p>2. 因网络故障、系统维护等不可抗力因素导致服务中断，平台不承担相应责任。</p>
        <p>3. 因您提供错误信息或操作不当导致的损失，平台不承担相应责任。</p>
        
        <p>四、服务终止</p>
        <p>平台有权根据业务调整终止部分或全部服务，终止前将提前通知您。</p>
      </view>

      <view v-if="type === 'about'" class="content-text">
        <h3>关于我们</h3>
        <p>一、医院简介</p>
        <p>本医院是一家集医疗、教学、科研、预防、保健为一体的综合性医院，拥有专业的医疗团队和先进的医疗设备，致力于为患者提供优质、高效、便捷的医疗服务。</p>
        
        <p>二、医院地址</p>
        <p>地址：XX省XX市XX区XX路XX号</p>
        <p>联系电话：0123-4567890</p>
        <p>工作时间：周一至周日 8:00-18:00（节假日除外）</p>
        
        <p>三、服务理念</p>
        <p>以患者为中心，以质量为核心，坚持以人为本、诚信服务的理念，不断提升医疗服务水平，为患者的健康保驾护航。</p>
        
        <p>四、平台说明</p>
        <p>本平台是医院官方指定的在线服务平台，旨在为患者提供便捷的挂号、缴费、查询等服务，简化就医流程，提升就医体验。</p>
        
        <p>五、意见反馈</p>
        <p>若您在使用平台过程中有任何意见或建议，可通过医院服务热线或平台反馈渠道与我们联系，我们将及时为您处理。</p>
      </view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      title: '', // 协议标题
      type: ''  // 协议类型
    };
  },
  onLoad(options) {
    // 接收路由参数，设置标题和协议类型
    this.title = options.title || '协议详情';
    this.type = options.type || '';
  }
};
</script>

<style scoped>
/* 页面容器 */
.agreement-container {
  min-height: 100vh;
  background-color: #f5f5f5;
}

/* 标题栏 */
.agreement-header {
  padding: 20rpx 30rpx;
  background-color: #fff;
  border-bottom: 1rpx solid #eee;
  position: sticky;
  top: 0;
  z-index: 10;
}

.header-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #333;
}

/* 内容滚动区 */
.agreement-content {
  background-color: #fff;
  padding: 30rpx 24rpx;
  height: calc(100vh - 88rpx);
}

/* 文本内容样式 */
.content-text {

  line-height: 1.8;
  font-size: 28rpx;
  color: lab(97.73% 0.67 0.22);
  padding: 0 30rpx; /* 左右各留30rpx距离，上下0 */
}

.content-text h3 {
  font-size: 34rpx;
  color: #222;
  margin-bottom: 30rpx;
  text-align: center;
}

.content-text p {
  margin-bottom: 20rpx;
  text-indent: 56rpx; /* 首行缩进2字符 */
}
</style>