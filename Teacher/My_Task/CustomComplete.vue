<template>
  <div>
    <div class="cus-complete">
      <p>发布班级:&nbsp;&nbsp;&nbsp; {{ className }}</p>
      <p>发布类型:&nbsp;&nbsp;&nbsp;{{ release }}</p>
      <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;名称:&nbsp;&nbsp;&nbsp;{{ data.name }}</p>
      <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内容:&nbsp;&nbsp;&nbsp;{{ data.content }}</p>
      <p class="release-time">发布时间: {{ time }}</p>
    </div>
    <p class="handle-btn">
      <span @click="handleBack">返回</span>
    </p>
    <footerLogo></footerLogo>
  </div>
</template>

<script>
import footerLogo from '@/components/XuHuiFooter/FooterLogo'
import moment from 'moment'
export default {
  components: {
    footerLogo
  },
  created () {
    this.data = JSON.parse(this.$route.query.data)
    console.log(this.data)
    this.time = moment(new Date()).format('YYYY-MM-DD HH:mm')
    this.classRelease = this.$route.query.clre
    const arr = this.classRelease.split(',')
    console.log(arr)
    this.className = arr[1]
    this.release = arr[0]
  },
  data () {
    return {
      data: {},
      time: '',
      classRelease: '',
      className: '',
      release: ''
    }
  },
  methods: {
    handleBack () {
      this.$router.push({ path: '/Home/Introduce' })
    }
  }
}
</script>

<style lang="less" scoped>
  // 操作按钮
  .handle-btn {
    position: absolute;
    right: 200px;
    bottom: 150px;
    span {
      display: inline-block;
      width: 114px;
      height: 60px;
      font-size: 16px;
      line-height: 60px;
      text-align: center;
      border-radius: 5px;
      background-color: #fff;
      cursor: pointer;
    }
}
  .cus-complete {
    margin-top: 88px;
    margin-left: 100px;
    color: #4D5753;
    p {
      font-size: 18px;
    }
  }
  // 发布时间
  // .release-time {
  //   margin-top: 120px;
  //   font-size: 14px;
  //   text-align: center;
  //   color: #B5B5B5;
  // }
</style>
