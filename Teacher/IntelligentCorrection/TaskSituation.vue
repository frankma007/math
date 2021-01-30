<template>
  <div>
    <p class="tit">{{ studentName }}的练习 <span v-show="waitRevision === '0'" class="repulse-btn cur">打回等待学生订正中</span></p>
    <div class="whole-info">
      <!-- 学生 -->
      <span class="dis m-r" v-for="item in studentList" :key="item.Id">
        <!-- <p class="f-s">试卷：{{ item.PaperTitle }}</p> -->
        <span class="dis m-b"><img class="img" src="@/assets/teacher/学生头像.png" alt=""></span>
        <span class="dis midd">
          <p class="m-b-p">{{ studentName }}</p>
          <p>答案首次提交时间：{{ item.SubmitDateTime }}</p>
        </span>
        <p>
          <img
            class="ans-img"
            v-for="(ite,ind) in item.ImgUrls"
            :key="ind"
            :src="ite"
            alt=""
            v-image-preview="ite">
        </p>
      </span>
      <!-- 教师 -->
      <span class="dis" v-for="item in teacherList" :key="item.TeacherCorrectId">
        <span class="dis m-b"><img class="img" :src="item.Photo" alt=""></span>
        <span class="dis midd">
          <p class="m-b-p">{{ item.TeacherName }} <span class="">第一次订正批改时间：{{ item.DateTime }}</span></p>
          <p>评价等第：{{ item.Epilogue }}</p>
        </span>
        <p>
          <img
            v-for="(ite,ind) in item.ImgUrls"
            :key="ind"
            class="ans-img"
            :src="ite"
            alt=""
            v-image-preview="ite"
          >
        </p>
      </span>
    </div>
  </div>
</template>

<script>
// import Recorder from 'js-audio-recorder'
// const recorder = new Recorder()
export default {
  created () {
    this.studentId = this.$route.query.studentId
    this.studentName = this.$route.query.studentName
    this.customId = this.$route.query.customId
    this.waitRevision = this.$route.query.waitRevision
    this.getSituationList()
  },
  watch: {
    $route () {
      this.studentId = this.$route.query.studentId
      this.studentName = this.$route.query.studentName
      this.customId = this.$route.query.customId
      this.waitRevision = this.$route.query.waitRevision
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/IntelligentCorrection/TaskSituation?' + ChangId) {
        this.getSituationList()
      }
    }
  },
  data () {
    return {
      studentId: '',
      studentName: '',
      customId: '', // 自定义练习id
      studentList: [], // 学生信息
      teacherList: [], // 教师信息
      waitRevision: '' // 等待学生订正中
    }
  },
  methods: {
    // 获取作业，老师批改详情
    getSituationList () {
      this.$uwonhttp.post('/Customer/CustomExercise/GetStudentDoPaperDetail', {
        customExerciseId: this.customId,
        studentid: this.studentId
      }).then(res => {
        console.log('作业详情', res)
        this.studentList = res.data.Data.StudentDatas
        this.teacherList = res.data.Data.TeacherDatas
      })
    }
  }
}
</script>

<style lang="less">
  .m-b {
    margin-right: 10px;
    margin-bottom: 15px;
  }
  .m-r {
    margin-right: 84px;
  }
  .f-s {
    font-size: 18px;
  }
  .m-b-p {
    margin-bottom: 8px;
  }
  .tit {
    margin-bottom: 24px;
    font-size: 18px;
  }
  .dis {
    display: inline-block;
  }
  .img {
    width: 40px;
    border-radius: 20px;
  }
  .ans-img {
    width: 128px;
    height: 96px;
    margin-right: 10px;
  }
  // 打回订正按钮
  .repulse-btn {
    display: inline-block;
    width: 125px;
    height: 25px;
    line-height: 25px;
    margin-left: 10px;
    text-align: center;
    font-size: 12px;
    color: #fff;
    background-color: #DADADA;
    border-radius: 15px;
  }
  // 整体
  .whole-info {
    padding: 24px 0 32px 24px;
    background-color: #fff;
    border-radius: 5px;
  }
  // 学生信息
  .midd {
    vertical-align: middle;
    font-size: 14px;
  }
</style>
