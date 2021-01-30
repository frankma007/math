<template>
  <div class="task-details">
    <!-- 作业形式选择 -->
    <!-- <img @click="handlePractice" src="@/assets/teacher/日常作业@2x.png" alt="">
    <img @click="handleCustom" src="@/assets/teacher/日常作业备份@2x.png" alt=""> -->
    <span class="task-info cur" @click="handleCustom">
      <div>
        <p class="f-z"><img class="m-r" src="@/assets/teacher/custom.png" alt="">自定义练习</p>
        <p class="m-l">自由布置，支持拍照智能批改</p>
      </div>
    </span>
    <span class="task-info cur" @click="handlePractice">
      <p class="f-z"><img class="m-r" src="@/assets/teacher/practice.png" alt="">检查练习</p>
      <p class="m-l">智能批改协助，高效率批阅</p>
    </span>
    <footerLogo></footerLogo>
  </div>
</template>

<script>
import footerLogo from '@/components/XuHuiFooter/FooterLogo'
export default {
  components: {
    footerLogo
  },
  methods: {
    // handlePractice () {
    //   this.$router.push({ path: '/Teacher/My_Task/ArrangmentPracticeAttribute' })
    // },
    handlePractice () {
      this.$router.push({ path: '/Teacher/IntelligentCorrection/MindPractice' })
    },

    handleCustom () {
      this.$router.push({ path: '/Teacher/My_Task/CustomPractice' })
    }
  }
}
</script>

<style lang="less" scoped>
  .task-details {
    margin-top: 100px;
    text-align: center;
    // img {
    //   width: 307px;
    //   margin-top: 100px;
    //   margin-right: 110px;
    //   cursor: pointer;
    // }
  }
  .m-r {
    margin-right: 12px;
  }
  .f-z {
    margin-top: 50px;
    margin-bottom: 12px;
    font-size: 24px;
  }
  .m-l {
    margin-left: 80px;
    color: #C7C7C7;
  }
  .task-info {
    width: 307px;
    height: 163px;
    margin-right: 108px;
    display: inline-block;
    text-align: center;
    background-color: #fff;
    border-radius: 8px;
    img {
      width: 32px;
    }
  }
</style>
