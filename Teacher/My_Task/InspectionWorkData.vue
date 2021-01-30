<template>
  <div class="data">
    <span class="col-btn" :class="{ 'color': color === 1}" @click="change" >班级数据</span>
    <span class="col-btn" :class="{ 'color': color == 2}" @click="change1" >逐题分析</span>
    <span class="col-btn" :class="{ 'color': color === 3}" @click="change2" >微课视频</span>
    <!-- <span v-for="(i, j) in 20" :key="j" :class="{ 'color': color === j}">{{ i }}逐题分析</span> -->
    <!-- <span class="fg" @click="add">右</span> -->
    <ClassData v-show="color === 1" :id="paperId" :classId="classId" :key="color"></ClassData>
    <SubjectAnalysis v-show="color === 2" :key="timer" :isShowSub="isShowSub"></SubjectAnalysis>
    <MicroVideo v-show="color === 3" :key="timer1"></MicroVideo>
  </div>
</template>

<script>
import ClassData from '@/components/ClassData/ClassData'
import SubjectAnalysis from '@/components/SubjectAnalysis/SubjectAnalysis'
import MicroVideo from '@/components/MicroVideo/MicroVideo'
export default {
  components: {
    ClassData,
    SubjectAnalysis,
    MicroVideo
  },
  created () {
    // 设置精准辅导跳转默认显示
    // this.defaultId = this.$route.query.id
    // if (this.defaultId !== '') {
    //   this.color = this.defaultId
    // }
    // this.color = this.$route.query.id
    this.paperId = this.$route.query.paperId
    this.classId = this.$route.query.classId
  },
  mounted () {
    if (this.$route.query.color == '2') {
      this.change1()
    }
  },
  data () {
    return {
      isShowSub: 0,
      // 显示组件
      color: 1,
      // 默认显示ID
      defaultId: '',
      // 添加key
      timer: '',
      timer1: 0,
      // 试卷id
      paperId: '',
      // 班级id
      classId: ''
    }
  },
  watch: {
    $route (newVal, oldVal) {
      debugger
      console.log(newVal)
      const ChangId = window.location.href.split('?')[1]

      if (this.$route.fullPath === '/Teacher/My_Task/InspectionWorkData?' + ChangId) {
        this.color = 1
      }
      if (this.$route.query.color == '2') {
        this.change1()
      }
    },

    '$route' (to, from) {
      if (from.path == '/Teacher/TeachingEvaluation/Weekly') {
        // 从哪里来
        debugger
        this.color = 2
        this.isShowSub = 1
      }
    }

  },
  methods: {
    change () {
      this.color = 1
    },
    change1 () {
      this.color = 2
      // 处理key值，重新加载vue组件
      this.timer = new Date().getTime()
    },
    change2 () {
      this.color = 3
      this.timer1 = 3
    }
    // add () {
    //   this.color = ++this.color
    // }
  }
}
</script>

<style lang="less" scoped>
  .data {
    .color {
    background-color: #6FBD9C;
    color:#fff;
  }
  }

  .col-btn {
    color: #BBBBBB;
  }
  span {
    display: inline-block;
    width: 100px;
    height: 40px;
    line-height: 40px;
    margin: 20px 35px 20px 0;
    text-align: center;
    border-radius: 20px;
    background-color: #fff;
    // border:1px #aaa solid;
    cursor: pointer;
  }
</style>
