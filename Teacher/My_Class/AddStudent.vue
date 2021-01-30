<template>
  <div class="add-student">
    <div class="search">

      <!-- <p v-show="format" class="format">{{ PromptText }}</p> -->
      <!-- <p v-show="handleUnregistered">手机号未注册</p> -->
      <a-input
        @keyup.enter.native="handleOk()"
        style="width: 50%;height: 62px;marginRight: 15px"
        v-decorator="[
          'mobilephone',
          { rules: [{ required: true, message: '请填写手机号'}] },
        ]"
        placeholder="请输入11位学生绑定手机号"
        @blur="mobilephoneValidate"
        v-model="mobilephone"
      >
      </a-input>
      <span class="span col" @click="handleOk" >确定</span>
      <span class="span bor" @click="eliminate">清除</span>

      <p class="title" v-show="format"><span><img src="@/assets/teacher/提示编组.png" alt="">{{ PromptText }}</span></p>

    </div>
    <!-- 学生信息 -->
    <div v-if="studentList" class="student-infor">
      <p>学生信息</p>
      <div>
        <span class="line" >
          <span><img :src="studentList.Photo" alt=""></span>
          <span>{{ studentList.Xuehao }}</span>
          <span>{{ studentList.TrueName }}</span>
          <span>绑定手机号</span>
          <span>{{ studentList.PhoneNum }}</span>
        </span>
        <span class="span add" @click="confirmAdd">确认添加</span>
      </div>

    </div>
    <footerLogo></footerLogo>
  </div>
</template>

<script>
import footerLogo from '@/components/XuHuiFooter/FooterLogo'
export default {
  components: {
    footerLogo
  },
  beforeCreate () {
    this.form = this.$form.createForm(this, { name: 'normal_login' })
  },
  created () {
    this.classId = this.$route.query.classId
    // this.userId = localStorage.getItem('UserId')
  },
  data () {
    return {
      // 学生ID
      userId: '',
      // 班级id
      classId: '',
      phone: '',
      PromptText: '',
      format: false,
      // 学生信息
      studentInfor: false,
      // 学生信息
      studentList: '',
      mobilephone: ''
    }
  },
  methods: {
    // 号码校验
    testMobilephone (str) {
      const regex = /^1[3456789]\d{9}$/
      if (!regex.test(str)) {
        return false
      } else {
        return true
      }
    },
    // 手机校验
    // rule, value, callback
    mobilephoneValidate (e) {
      // 主要就是添加一个对undefined和空串的判断
      if (e.target.value === '') {
        this.format = false
        this.studentInfor = false
      } else {
        if (!this.testMobilephone(e.target.value)) {
          this.format = true
          this.PromptText = '手机号格式不正确'
          // 图标
          this.mobileSuc = ''
          this.canClick = false
        } else {
          this.format = false
          // 图标
          this.mobileSuc = 'success'
          // 验证码
          this.canClick = true
          // callback()
        }
      }
    },
    // 确认
    handleOk () {
      console.log('shoujihao', this.mobilephone)
      // var values = this.form.getFieldsValue()
      if (this.format) {

      } else {
        this.$uwonhttp.get('/Class/TeacherClassManager/GetStudentByPhoneNum', {
          params: {
            PhoneNum: this.mobilephone
          }
        }).then(res => {
          console.log('学生Id', res)
          if (res.data.Data.length === 0) {
            this.$message.warning('没有该学生')
          }
          if (res.data.Success) {
            this.studentList = res.data.Data[0]
            console.log('', this.studentList)
            this.studentInfor = true
            this.userId = res.data.Data[0].UserId
          }
        })
      }
    },
    // 清除
    eliminate () {
      document.getElementById('normal_login_mobilephone').value = ''
    },
    // 确认添加
    confirmAdd () {
      this.$uwonhttp.get('/Class/TeacherClassManager/AddStudentToClass', {
        params: {
          UserId: this.userId,
          ClassId: this.classId
        }
      }).then(res => {
        console.log('添加学生', res)
        if (res.data.Success) {
          this.$message.success(res.data.Msg)
          this.$router.push('/Teacher/My_Class/StudentManage')
        } else {
          this.$message.success(res.data.Msg)
        }
      })
    },
    handleSubmit () {

    }
  }
}
</script>

<style lang="less" scoped>
  .col {
    color: #fff;
  }
  .bor {
    color: #BFBFBF;
    border: 1px solid #B5B5B5;
  }
  .add-student {
    width: 80%;
    margin: 0 auto;
  }
  // 提示
  .title {
    margin-top: 20px;
    // padding-left: 50px;
  }
  .search {
    // text-align: center;
    padding-left: 270px;
    margin-top: 160px;

    span:nth-child(2) {
      background-color: #68BB97;
    }
    span:nth-child(3) {
      background-color: #fff;
    }
  }
  .span {
      display: inline-block;
      width: 111px;
      height: 48px;
      line-height: 48px;
      margin-right: 24px;
      text-align: center;
      border-radius: 25px;
      cursor: pointer;
  }
  //
  .student-infor {
    p:nth-child(1) {
      text-align: center;
      margin-top: 50px;
      font-size: 18px;
      // padding-left: 50px;
    }
    div {
      text-align: center;
      margin-top: 56px;
      .line {
        width: 50%;
        display: inline-block;
        border-bottom: 1px dotted #ccc;
        span {
          margin-right: 10%;
        }
        img {
          width: 35px;
        }
      }
      .add {
         background-color: #68BB97;
      }
    }
  }
</style>
