<template>
  <div class="add-student">
    <div class="search">
      <a-form
        id="components-form-demo-normal-login"
        :form="form"
        class="login-form"
        @submit="handleSubmit"
      >
        <!-- <p v-show="format" class="format">{{ PromptText }}</p> -->
        <!-- <p v-show="handleUnregistered">手机号未注册</p> -->
        <a-form-item >
          <a-input
            @keyup.enter.native="handleOk()"
            style="width: 50%;height: 62px;marginRight: 15px"
            v-decorator="[
              'mobilephone',
              { rules: [{ required: true, message: '请填写手机号'}] },
            ]"
            placeholder="请输入11位教师绑定手机号"
            @blur="mobilephoneValidate"
          >
          </a-input>
          <span class="span" @click="handleOk" @keyup.enter="handleOk">确定</span>
          <span class="span" @click="eliminate">清除</span>
        </a-form-item>

      </a-form>
      <p class="title" v-show="format"><span><img src="@/assets/teacher/警告.png" alt="">{{ PromptText }}</span></p>
    </div>

    <!-- 老师信息 -->
    <div class="student-infor" v-show="studentInfor">
      <p>老师信息</p>
      <div v-for="item in studentList" :key="item.Id">
        <span class="line" >
          <span><img style="width: 32px;border-radius: 18px;" :src="item.Photo" alt=""></span>
          <!-- <span>{{ studentList.Xuehao }}</span> -->
          <span>{{ item.RealName }}</span>
          <!-- <span>绑定手机号</span> -->
          <span>{{ item.PhoneNum }}</span>
          <span v-if="item.IsFriend === 0" class="span add" @click="confirmAdd(item.Id)">确认添加</span>
          <span v-if="item.IsFriend === 1" class="span add">已添加</span>
          <span v-if="item.IsFriend === 2" class="span add">等待同意</span>
        </span>

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
    this.UserId = localStorage.getItem('UserId')
  },
  data () {
    return {
      // 用户id
      UserId: '',
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
      studentList: {},
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
      // console.log('shoujihao', this.mobilephone)
      var values = this.form.getFieldsValue()
      console.log(values)
      if (this.format) {

      } else {
        this.$http.post('/Friend/FriendRelationShip/GetTeacherByPhoneNum', { phone: values.mobilephone, userId: this.UserId }).then(res => {
          console.log('老师的查找---', res)
          if (res.Success) {
            this.studentList = res.Data
            console.log('', this.studentList)
            this.studentInfor = true
            if (res.Data.length === 0) {
              this.$message.warning('没有该教师')
            }
          }
        })
      }
    },
    // 清除
    eliminate () {
      document.getElementById('normal_login_mobilephone').value = ''
    },
    // 确认添加
    confirmAdd (id) {
      this.$http.post('/Friend/FriendRelationShip/AddFriend', { friendsId: id, userId: this.UserId }).then(res => {
        console.log('添加老师---', res)
        if (res.Success) {
          this.$message.success('添加已发送，等待确认')
          this.$router.push({ path: '/SchoolManager/myFriends/indexFriends' })
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
      }
      .add {
         background-color: #68BB97;
      }
    }
  }
</style>
