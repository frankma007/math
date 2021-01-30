<template>
  <div>
    <div class="stu-infor" id="stu-infor">
      <p><img style="width: 56px;border-radius:50px" :src="studentList.photo" alt=""></p>
      <p class="p"><span>学生信息</span><span>{{ studentList.UserNo }}</span><span><img @click="handleStudentInfor" class="cur" src="@/assets/teacher/编辑按钮.png" alt=""></span></p>
      <a-modal
        v-model="visible"
        title="修改学号"
        @ok="handleOk"
        width="630px"
        :dialogStyle="{ padding: '40px',backgroundColor: '#68BB97'}"
        centered
      >
        <p><img src="@/assets/teacher/学士帽.png" alt=""> &nbsp;&nbsp;&nbsp;&nbsp;<a-input v-model="StudentNumber" style="width: 268px;"></a-input></p>
      </a-modal>
      <p class="p"><span>学生姓名</span> <span>{{ studentList.TrueName }}</span><span><img @click="handleStudentName" class="cur" src="@/assets/teacher/编辑按钮.png" alt=""></span></p>
      <a-modal
        v-model="modify"
        title="修改姓名"
        @ok="handleOkName"
        width="630px"
        centered
      >
        <p>
          <img style="width: 37px" src="@/assets/teacher/学士帽2.png" alt=""> &nbsp;&nbsp;&nbsp;&nbsp;
          <a-input v-model="studentName" style="width: 268px"></a-input>
        </p>
      </a-modal>
      <p class="make-sub">做卷信息</p>
      <div class="infor">
        <p v-for="(item,index) in studentMakePaper" :key="index">{{ item }}</p>
      </div>
      <p @click="removeStudent" class="remove">移出该学生</p>
      <a-modal
        v-model="removeStu"
        title="温馨提示"
        @ok="handleRemove"
        width="630px"
        centered
      >
        <p>确认要移出 {{ studentList.TrueName }}学生吗？</p>
      </a-modal>
    </div>
    <footerLogo :heightPage="heightPage"></footerLogo>
  </div>
</template>

<script>
import footerLogo from '@/components/newFooterLogo/newFooterLogo'
export default {
  components: {
    footerLogo
  },
  created () {
    this.studentId = this.$route.query.id
    this.teacherId = localStorage.getItem('UserId')
    console.log('学生个人----', this.studentId)
    this.getStudentInfor()
  },
  mounted () {
    this.heightPage = document.getElementById('stu-infor').offsetHeight
  },
  watch: {
    $route () {
      const ChangId = window.location.href.split('?')[1]
      this.studentId = this.$route.query.id
      if (this.$route.fullPath === '/Teacher/My_Class/StudentInfor?' + ChangId) {
        this.getStudentInfor()
      }
    }
  },
  data () {
    return {
      heightPage: 0,
      // 学生ID
      studentId: '',
      // 学生个人信息
      studentList: {},
      // 教师id
      teacherId: '',
      // 个人做卷信息
      studentMakePaper: [],
      // 修改学号
      visible: false,
      // 学号
      StudentNumber: '',
      // 修改姓名
      modify: false,
      // 姓名
      studentName: '',
      // 移出学生
      removeStu: false
    }
  },
  methods: {
    // 获取学生个人信息
    getStudentInfor () {
      this.$uwonhttp.post('/Student/User/GetStudentDetail', { StudentId: this.studentId }).then(res => {
        console.log('学生个人信息', res)
        this.studentList = res.data.Data
        this.studentMakePaper = res.data.Data.Paper
      })
    },
    //  打开处理学生学号
    handleStudentInfor () {
      this.visible = true
    },
    // 处理学生学号确定
    handleOk () {
      console.log('学号---', this.StudentNumber)
      this.$uwonhttp.post('/Student/User/GetUserNoForEdit', { StuNo: this.StudentNumber, StudentId: this.studentId, TeacherId: this.teacherId }).then(res => {
        console.log('修改学号--', res)
        this.visible = false
        this.getStudentInfor()
      })
    },
    // 打开处理学生姓名
    handleStudentName () {
      this.modify = true
    },
    // 处理学生姓名确定
    handleOkName () {
      console.log('学生姓名--', this.studentName)
      this.$uwonhttp.post('/Student/User/GetTrueNameForEdit', { Name: this.studentName, StudentId: this.studentId, TeacherId: this.teacherId }).then(res => {
        console.log('修改姓名--', res)
        this.modify = false
        this.getStudentInfor()
      })
    },
    // 移出该学生
    removeStudent () {
      this.removeStu = true
    },
    // 确认移出该学生
    handleRemove () {
      this.$uwonhttp.post('/Student/User/DeleteStudent', { StudentId: this.studentId, TeacherId: this.teacherId }).then(res => {
        console.log('移出学生---', res)
        this.removeStu = false
        if (res.data.Success) {
          this.$message.success('成功移除')
          this.$router.push({ path: '/Teacher/My_Class/StudentManage' })
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .cur {
    cursor: pointer;
  }
   /deep/.ant-modal-title {
    color: #fff;
  }
  .ant-modal-close-x {
    color: #fff;
  }
  .stu-infor {
    text-align: center;
    margin: 0 auto;
    margin-top: 168px;

  }
  .p {
    width: 500px;
    height: 45px;
    text-align: center;
    margin: 0 auto;
    display: flex;
    justify-content: center;
    align-items: center;
    justify-content: space-between;
    border-bottom: 1px solid #ccc;
    margin-bottom: 30px;
}
  .make-sub {
    width: 500px;
    margin: 0 auto;
    text-align: left;
    margin-bottom: 30px;
  }
  .infor {
    width: 500px;
    height: 110px;
    margin: 0 auto;
    margin-bottom: 30px;
     padding-left: 25px;
    padding-top: 15px;
    border-radius: 5px;
    text-align: left;
    background-color: #EAEAEA;
  }
  .remove {
    width: 500px;
    margin: 0 auto;
    height: 45px;
    line-height: 45px;
    border-radius: 25px;
    color: #fff;
    background-color: #68BB97;
    cursor: pointer;
  }
  // 模态框
      /deep/.ant-modal-header {
        text-align: center;
        background-color: #68BB97;
      }
      /deep/.ant-modal-footer button + button {
        background-color: #68BB97;
        border: none;
      }
</style>
