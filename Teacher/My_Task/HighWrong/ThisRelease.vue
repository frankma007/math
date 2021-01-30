<template>
  <div>
    <p class="f-s-tip">本次发布练习</p>
    <div class="release-info">
      <p class="f-s"><img src="@/assets/teacher/file.png" alt=""><span>{{ paperName }}</span></p>
      <p><img src="@/assets/teacher/correct.png" alt=""></p>
      <p>练习发布成功，可在<span @click="toClassPractive" class="col cur">"班级练习"</span>中查看试卷</p>
    </div>
  </div>
</template>

<script>
export default {
  created () {
    this.paperName = this.$route.query.paperName
  },
  data () {
    return {
      paperName: ''
    }
  },
  methods: {
    toClassPractive () {
      this.$router.push({ path: '/Teacher/My_Task/MyClassTask' })
    }
  }
}
</script>

<style lang="less" scoped>
  .f-s-tip {
    font-size: 18px;
  }
  .f-s {
    margin-bottom: 25px;
    font-size: 20px;
    span {
      margin-left: 15px;
    }
  }
  .col {
    color: #68BB97;
  }
  .release-info {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    text-align: center;
    p:nth-child(2) {
      margin-bottom: 10px;
    }
    p:nth-child(3) {
      font-size: 14px;
    }
  }
</style>
