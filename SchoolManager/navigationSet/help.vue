<template>
  <div class="help-feed">
    <!-- 反馈内容 -->
    <div class="feed-back-info">
      <p><span style="color: red">*</span>请写下您的问题或者意见反馈，我们会尽快解决并改进!<span class="fg my-feedback cur" @click="toMyFeedback">我的反馈</span></p>
      <textarea v-model="textarea" name="" id="" cols="30" rows="10"></textarea>
      <p v-show="tip" class="tips"><img style="vertical-align: middle;" src="@/assets/teacher/警告.png" alt=""><span>未填写内容</span></p>
      <p style="margin-top: 40px;">请留下您的联系方式，方便后续为您更好的服务！（非必填）</p>
      <input type="text" placeholder="QQ / 微信号 / 邮箱 / 手机号" v-model="contact">
      <p class="submit"><span @click="submit">提交反馈</span></p>
    </div>
    <!-- 专科专练数据 -->
    <div class="customer-service">
      <p class="contact">联系我们</p>
      <p><span>客服QQ</span><span class="fg">3355177030</span></p>
      <p><span>客服电话</span><span class="fg">021-54486816</span></p>
      <p><span>微信公众号</span><span class="fg">有我更精彩</span></p>
      <div class="cus-code">
        <img src="@/assets/teacher/位图.png" alt="">
        <p>专课专练客服号</p>
      </div>
    </div>
  </div>
</template>

<script>
import '@/utils/utils.less'
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
  },
  data () {
    return {
      userId: '',
      // 反馈内容
      textarea: '',
      // 联系方式
      contact: '',
      tip: false
    }
  },
  methods: {
    submit () {
      console.log(this.textarea)
      console.log(this.contact)
      if (!this.textarea) {
        this.tip = true
      } else {
        this.tip = false
        this.$uwonhttp.post('/Customer/Suggestion/SaveData', { CreatorId: this.userId, FaultContent: this.textarea, Phone: this.contact }).then(res => {
          console.log('反馈0--', res)
          if (res.data.Success) {
            this.$message.success('提交成功')
            this.$router.push({ path: '/SchoolManager/navigationSet/helpComplete' })
          }
        })
      }
    },
    // 我的反馈
    toMyFeedback () {
      this.$router.push({ path: '/SchoolManager/navigationSet/myHelpList' })
    }
  }
}
</script>

<style lang="less" scoped>
  .help-feed {
    display: flex;
  }
  .tips {
    color: #FF682C;
    margin-bottom: 0;
  }
  // 反馈内容
  .feed-back-info {
    flex: 1;
    font-size: 16px;
    p {
      span {
        // font-size: 18px;
      }
    }
    textarea {
      resize: none;
      width: 100%;
      height: 300px;
      // margin-bottom: 50px;
      border-radius: 5px;
    }
    input {
      width: 100%;
      height: 44px;
      text-indent: 20px;
      border-radius: 5px;
      border: none;
      border: 1px solid #ccc;
    }
    .submit {
      margin-top: 30px;
      text-align: center;
      span {
        display: inline-block;
        width: 280px;
        height: 45px;
        line-height: 45px;
        border-radius: 5px;
        background-color: #68BB97;
        cursor: pointer;
      }
    }
  }
  // 客服
  .customer-service {
    flex: 1;
    margin-left: 50px;
    padding: 0 130px;
    // 客服二维码
    .cus-code {
      text-align: center;
      margin-top: 35px;
      p {
        margin-top: 15px;
      }
    }
    .contact {
      font-size: 25px;
      text-align: center;
    }
    p {
      span {
        font-size: 18px;
      }
    }
  }
  // 我的反馈
  .my-feedback {
    // width: 64px;
    // height: 24px;
    // line-height: 24px;
    padding: 8px;
    text-align: center;
    font-size: 14px;
    color: #fff;
    background-color: #68BB97;
    border-radius: 13px;
  }
</style>
