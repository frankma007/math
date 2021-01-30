<template>
  <div class="personal-settings">
    <!-- 个人设置内容 -->
    <div class="personal-setting-info">
      <span style="text-align:center">
        <a-avatar :size="64" icon="user" :src="userHead"/>
        <!-- <p class="cur" @click="ModifyAvatar" >修改头像</p> -->
        <p>
          <a-upload name="file" :multiple="true" :action="uploadPictureUrl" :headers="headers" @change="handleChange">
            <a-button>
              <a-icon type="upload" />修改头像
            </a-button>
          </a-upload>
        </p>
      </span>
      <!-- 基本信息 -->
      <div class="basic-info">
        <a-form
          id="components-form-demo-normal-login"
          :form="form"
          class="login-form"
          @submit="handleSubmit">
          <a-form-item>
            <span>真实姓名:</span>
            <a-input
              style="width: 400px;textAlign: center"
              v-decorator="[
                'mobilephone',
              ]"
              :placeholder="userData.RealName"
            >
            </a-input>
          </a-form-item>
          <a-form-item>
            <span>手机号码:</span><span>{{ userName }}</span><span class="cur" @click="handleMobilePhone" style="color: blue">修改</span>
            <a-modal v-model="visible" title="修改手机号" @ok="handleOk" ok-text="保存" centered>
              <a-form-item>
                <a-input
                  v-decorator="[
                    'oldmobilephone',
                    { rules: [{validator: mobilephoneValidate}] },
                  ]"
                  placeholder="原手机号"
                >
                  <a-icon slot="prefix" type="mobile" />
                </a-input>
              </a-form-item>
              <a-form-item>
                <a-input
                  v-decorator="[
                    'newmobilephone',
                    { rules: [{validator: mobilephoneValidate}] },
                  ]"
                  placeholder="新手机号"
                >
                  <a-icon slot="prefix" type="mobile" />
                </a-input>
              </a-form-item>
              <a-form-item>
                <a-input
                  style="width: 50%"
                  v-decorator="[
                    'testing',
                    { rules: [] },
                  ]"
                  placeholder="验证码"
                >
                  <a-icon slot="prefix" type="safety-certificate" />
                </a-input>
                <span class="testing-btn" @click="countDown" :class="{disabled: !this.canClick}">{{ content }}</span>
              </a-form-item>
            </a-modal>
          </a-form-item>
          <a-form-item>
            <span>密码设置:</span><span @click="handlePassWord" class="cur" style="color: blue">修改密码</span>
            <a-modal v-model="visiblePAW" title="修改密码" ok-text="确定" @ok="handleOkPassWord" centered>
              <a-form-item>
                <a-input
                  v-decorator="[
                    'oldPassword',
                    { rules:[{ validator: passwordValidate }]}
                  ]"
                  type="password"
                  placeholder="输入原密码"
                >
                </a-input>
                <p v-show="testEqually" style="textIndent: 5px;color: red;height: 25px;marginBottom: 0">{{ testContent }}</p>
              </a-form-item>
              <a-form-item>
                <a-input
                  v-decorator="[
                    'newPassword',
                    { rules:[{ validator: passwordValidate }]}
                  ]"
                  type="password"
                  placeholder="输入新密码"
                >
                </a-input>
              </a-form-item>
              <a-form-item>
                <a-input
                  v-decorator="[
                    'againNewPassword',
                    { rules:[{ validator: passwordValidate }]}
                  ]"
                  type="password"
                  placeholder="再次输入密码"
                >
                </a-input>
                <p v-show="equally" style="textIndent: 5px;color: red;">密码不一致</p>
              </a-form-item>
            </a-modal>
          </a-form-item>
          <!-- <span class="cur">身份切换</span><a-icon class="cur" type="swap" /> -->
          <a-form-item>
            <span>所属角色:</span><span>校领导</span>
          </a-form-item>
          <a-form-item>
            <span>所属学校:</span><span>{{ this.schoolName }}</span>
          </a-form-item>
          <a-form-item>
            <span>注册日期:</span><span>{{ userData.CreateTime }}</span>
          </a-form-item>
          <a-form-item>
            <a-button type="primary" html-type="submit" class="login-form-button">
              保存个人信息
            </a-button>
          </a-form-item>
        </a-form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  beforeCreate () {
    this.form = this.$form.createForm(this, { name: 'normal_login' })
  },
  created () {
    this.userName = localStorage.getItem('mobilePhone')
    this.userId = localStorage.getItem('UserId')
    this.jwtToken = localStorage.getItem('jwtToken')
    console.log(this.jwtToken)

    // 获取用户基本信息
    this.getUserInfor()
    // 获取教师对应班级
    this.getTeacherClass()
    console.log(this.$rootUrl)
  },
  data () {
    return {
      userName: '',
      // 用户头像
      userHead: '',
      // 用户信息
      userData: {},
      jwtToken: '',
      // 用户ID
      userId: '',
      trueName: '',
      // 教师性别
      sex: '',
      // 教师班级信息
      TeacherClass: [],
      // 修改手机号
      visible: false,
      // 修改密码
      visiblePAW: false,
      // 验证密码一致
      equally: false,
      // 验证原密码是否正确
      testContent: '',
      testEqually: false,
      teacherList: [],
      // 验证码
      content: '发送验证码',
      totalTime: 60,
      canClick: false,
      uploadPictureUrl: '',
      headers: {
        authorization: ''
      },
      schoolName: ''
    }
  },
  methods: {
    // 获取用户基本信息
    getUserInfor () {
      console.log(this.userId)
      this.$http.post('/Base_Manage/Base_User/GetUserInfoByUserName', { userName: this.userName }).then(res => {
        console.log('用户信息---', res)
        this.userData = res.Data.User
        this.trueName = res.Data.User.RealName
        this.userHead = res.Data.User.Photo
        // 教师性别
        this.sex = res.Data.User.Sex
        this.headers = {
          authorization: `Bearer ${this.jwtToken}`
        }
        this.schoolName = res.Data.SchoolName
        console.log(this.headers)
      })
      this.uploadPictureUrl = `http://uwoomobile.doggod.xyz/TeacherUserCenter/TeacherPersonal/SetUserPhoto?UserId=${this.userId}`
    },
    // 修改头像
    handleChange (arg1, label, extra) {
      if (arg1.file.status === 'done') {
        if (arg1.file.response.Success) {
          // debugger
          console.log(arg1.file.response)
          this.userHead = arg1.file.response.Data
          this.$message.success('修改头像成功')
          // this.$router.go(0)
        } else {
          this.$message.error(arg1.file.response.Msg, 2)
        }
      }
    },
    // 获取教师对应班级
    getTeacherClass () {
      this.$http.post('/ClassManage/Exam_Class/GetListByUserId', { userId: this.userId }).then(res => {
        console.log('教师对应班级--', res)
        this.teacherList = res.Data
      })
    },
    // 修改手机号
    handleMobilePhone () {
      this.visible = true
    },
    // 修改手机号模态框确认
    handleOk () {
      this.visible = false
      const values = this.form.getFieldsValue()
      console.log('手机--', values)
      this.$uwonhttp.post('/User/UserManager/UpdatePhoneNum', { Userid: this.userId, PhoneNum: values.newmobilephone, Code: values.testing }).then(res => {
        console.log('修改手机号---', res)
      })
    },
    // 修改密码
    handlePassWord () {
      this.visiblePAW = true
    },
    // 修改密码模态框确认
    handleOkPassWord () {
      const values = this.form.getFieldsValue()
      console.log('修改密码---', values)
      if (values.newPassword !== values.againNewPassword) {
        this.equally = true
        this.testContent = '密码不一致'
      } else if (values.oldPassword.length < 6) {
        return false
      } else {
        this.equally = false
        this.$uwonhttp.post('/TeacherUserCenter/TeacherPersonal/UpdatePwdByOld', { userId: this.userId, oldPwd: values.oldPassword, newPwd: values.againNewPassword }).then(res => {
          console.log(res)
          if (res.data.Success) {
            this.$message.success(res.data.Data)
            this.visiblePAW = false
          } else {
            this.testEqually = true
            this.testContent = res.data.Msg
          }
        })
      }
    },
    // 获取手机验证码
    getPhoneCount (mobilephone) {
      this.$uwonhttp.get('/User/UserManager/GetPhoneNumCode', {
        params: {
          phoneNum: mobilephone
        }
      }
      ).then(res => {
        console.log('手机验证码---', res)
      })
    },
    // 验证码 265071
    countDown () {
      console.log(this.canClick)
      if (!this.canClick) return
      this.canClick = false
      const values = this.form.getFieldsValue()
      console.log(values)

      // 获取手机验证码
      this.getPhoneCount(values.oldmobilephone)
      this.content = `${this.totalTime}s重新获取`
      const clock = window.setInterval(() => {
        this.totalTime--
        this.content = `${this.totalTime}s重新获取`
        if (this.totalTime < 0) {
          window.clearInterval(clock)
          this.content = '重新发送验证码'
          this.totalTime = 60
          this.canClick = true
        }
      }, 1000)
    },
    testMobilephone (str) {
      const regex = /^1[3456789]\d{9}$/
      if (!regex.test(str)) {
        return false
      } else {
        return true
      }
    },
    // 手机校验
    mobilephoneValidate (rule, value, callback) {
      if (!this.testMobilephone(value)) {
        callback(new Error('请输入正确的手机号'))
      } else {
        // 验证码
        this.canClick = true
        callback()
      }
    },
    // 密码校验
    passwordValidate (rule, value, callback) {
      // console.log(value)
      if (value === undefined || value === '') {
        this.equally = false
        this.testEqually = false
        callback(new Error('密码不能为空'))
      } else if (value.length < 6) {
        this.equally = false
        this.testEqually = false
        callback(new Error('密码不少于6位'))
      } else {
        callback()
      }
    },
    // 保存个人设置
    handleSubmit (e) {
      e.preventDefault()
      if (this.sex === 1) {
        this.sex = '男'
      } else {
        this.sex = '女'
      }
      console.log(this.sex)
      debugger
      this.form.validateFields((err, values) => {
        if (!err) {
          console.log('Received values of form: ', values)
          this.$uwonhttp.post('/TeacherUserCenter/TeacherPersonal/SetUserRealName', { UserId: this.userId, RealName: values.mobilephone, Gender: this.sex }).then(res => {
            console.log('保存个人设置', res)
            if (res.data.Success) {
              this.$message.success('修改姓名成功')
              this.$router.go(0)
            }
            this.getUserInfor()
          })
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
  // 上传预览设置
  /deep/.ant-upload-list {
    display: none;
  }
  // 设置内容
  .personal-setting-info {
    display: flex;
    width: 1108px;
    height: 500px;
    margin: 0 auto;
    margin-top: 106px;
    // background-color: #fff;
    // 表单内容
    .basic-info {
      margin: 20px 0 0 150px;
      span {
        margin-right: 30px;
      }
    }
  }
   /deep/.ant-input-affix-wrapper {
        margin-bottom: 15px;
        height: 45px;
    }
  // 验证码按钮
      .testing-btn {
        display: inline-block;
        background-color: #ccc;
        height: 36px;
        padding: 0 10px;
        line-height: 36px;
        margin-left: 30px;
        cursor: pointer;
    }
    // 鼠标变化
    .disabled {
        background-color: #D6D6D6;
        border-color: #D6D6D6;
        color:#fff;
        cursor: not-allowed; // 鼠标变化
      }
</style>
