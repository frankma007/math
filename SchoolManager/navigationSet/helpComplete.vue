<template>
  <div>
    <p class="help">帮助与反馈</p>
    <div class="help-list">
      <p><img src="@/assets/teacher/icon-输入正确 2.png" alt=""></p>
      <p class="name">提交成功！您的反馈已记录，感谢您的支持！</p>
    </div>
    <footerLogo></footerLogo>
  </div>
</template>

<script>
import footerLogo from '@/components/XuHuiFooter/FooterLogo'
export default {
  components: {
    footerLogo
  }
}
</script>

<style lang="less" scoped>
  .help {
    font-size: 16px;
    color: #49534E;
  }
  .name {
    font-size: 20px;
  }
  .help-list {
    text-align: center;
  }
</style>
