<template>
  <div>
    <p class="title">本次发布练习</p>
    <div class="whole-pic">
      <p><img src="@/assets/teacher/file.png" alt="">{{ picName }}</p>
      <p><img src="@/assets/teacher/icon-输入正确 2.png" alt=""></p>
      <p class="complete">发布成功</p>
      <p class="mind-p">可在<span @click="toMindCorrect" class="mind-cor cur">“智能批改-检查练习”</span>中查看学生提交情况</p>
    </div>
  </div>
</template>

<script>
export default {
  created () {
    this.picName = this.$route.query.prcName
  },
  data () {
    return {
      picName: ''
    }
  },
  watch: {
    $route: {
      handler (val) {
        const ChangId = window.location.href.split('?')[1]
        debugger
        if (val.fullPath === '/Teacher/IntelligentCorrection/CustomComplete?' + ChangId) {
          this.picName = this.$route.query.prcName
        }
      }
    }
  },
  methods: {
    toMindCorrect () {
      this.$router.push({ path: '/Teacher/IntelligentCorrection/MindPractice' })
    }
  }
}
</script>

<style lang="less" scoped>
  .title {
    font-size: 20px;
  }
  .complete {
    margin-bottom: 16px;
    font-size: 18px;
  }
  .mind-p {
    font-size: 18px;
    .mind-cor {
      color: #68BB97;
    }
  }

  .whole-pic {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    p {
      text-align: center;
    }
    p:nth-child(1) {
      font-size: 28px;
      margin-bottom: 43px;
      img {
        width: 40px;
      }
    }
    p:nth-child(2) {
      img {
        width: 40px;
      }
    }
  }
</style>
