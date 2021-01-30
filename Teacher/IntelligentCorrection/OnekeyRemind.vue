<template>
  <div>
    <div v-if="showLack" class="show-lack">
      <img src="@/assets/lack/错题收纳为空.png" alt="">
      <p>暂无未做学生</p>
    </div>
    <div>
      <p class="clearfix">
        <span class="fl f-s">未做 共{{ allStudentList.length }}位学生</span>
        <span @click="onekeyRemind" class="fg one-remind cur">一键提醒</span>
      </p>
      <div class="all-stu-infor">
        <ul class="clearfix">
          <li v-for="(item,index) in allStudentList" :key="item.UserId">
            <!-- <img @click="removeStudent(item.UserId,$event)" v-show="checkStudent" class="img" :src="checkIcon" alt=""> -->
            <p><img class="classStu-img" :src="item.Photo" alt=""></p>
            <p class="cur" style="margin-bottom: 5px;" >{{ item.StudentNo }}</p>
            <p class="cur">{{ item.StudentName }}</p>
          </li>
        <!-- <li>
          <img class="img" src="@/assets/teacher/未勾选.png" alt="">
          <p><img src="@/assets/teacher/老师头像.png" alt=""></p>
          <p>01</p>
          <p>杨思琦</p>
        </li> -->
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    this.classId = this.$route.query.classId
    this.id = this.$route.query.paperId
    this.getAllStudents()
  },
  watch: {
    $route (to, from) {
      // console.log('单题详情加载', to, from)
      this.classId = this.$route.query.classId
      this.id = this.$route.query.paperId
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/IntelligentCorrection/OnekeyRemind?' + ChangId) {
        this.getAllStudents()
      }
    }
  },
  data () {
    return {
      showLack: false, // 缺省
      userId: '',
      // 自定义练习id
      id: '',
      classId: '',
      // 未作学生
      allStudentList: []

    }
  },
  methods: {
    // 获取全部学生信息
    getAllStudents (id) {
      this.$uwonhttp.post('/Customer/CustomExercise/UnDoStudent', { classId: this.classId, customExerciseId: this.id }).then(res => {
        console.log('全部学生信息--', res)
        if (res.data.Data.length === 0) {
          this.showLack = true
        } else {
          this.showLack = false
        }
        this.allStudentList = res.data.Data
        // this.classNum = res.data.Data.length
        console.log('全部学生信息--', this.allStudentList)
      })
    },
    onekeyRemind () {
      this.$uwonhttp.post('/Customer/CustomExercise/SendPushToNotDoStudent', { classId: this.classId, teacherId: this.userId, paperId: this.id }).then(res => {
        console.log('一键提醒', res)
        this.$message.success('已提醒')
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .f-s {
    font-size: 14px;
  }
  // 缺省
  .show-lack {
    margin-top: 150px;
    text-align: center;
    color: #ccc;
  }
  // 全部学生信息
  .all-stu-infor {
    margin-top: 16px;
    background-color: #fff;
    border-radius: 5px;
    ul {
      padding: 32px 0 0 30px;
      background-color: #fff;
      border-radius: 5px;
      li {
        position: relative;
        float: left;
        width: 83px;
        height: 130px;
        margin-right: 60px;
        margin-bottom: 15px;
        text-align: center;
        // cursor: pointer;
        .img {
          position: absolute;
          top: -1px;
          left: 54px;
          cursor: pointer;
        }
      }
    }
  }
  // 头像
  .classStu-img {
    width: 56px;
    height:56px;
    border-radius: 30px;
  }
  .one-remind {
    display: inline-block;
    width: 88px;
    height: 30px;
    line-height: 30px;
    text-align: center;
    color: #fff;
    background-color: #68BB97;
    border-radius: 15px;
  }
</style>
