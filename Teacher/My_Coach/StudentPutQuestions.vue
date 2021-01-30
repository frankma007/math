<template>
  <div>
    <h1>学生提问</h1>
    <!-- 学生基本信息 -->
    <div class="clearfix student-basic-data">
      <a-avatar size="large" icon="user" class="fl"/>
      <div class="fl">
        <p>刘毅清</p>
        <p>高小一中</p>
      </div>
      <div class="fl">
        <a-switch default-checked @change="onChange" />隐藏已回答
      </div>
      <!-- 提问信息基本 -->
      <div class="fg question-data">
        <span>
          <p>3</p>
          <p>近七天学生提问数</p>
        </span>
        <span>
          <p>156</p>
          <p>学生总提问数</p>
        </span>
        <span>
          <p>5</p>
          <p>近七天回答数</p>
        </span>
        <span>
          <p>100</p>
          <p>累计回答数</p>
        </span>
      </div>
    </div>
    <!-- 学生提问搜索 -->
    <div class="seacher-data clearfix">
      <div class="fl">
        <a-cascader
          :options="options"
          @change="onChange"
          placeholder="全部练习"

        />
        <a-select style="width: 220px; marginRight: 20px;" placeholder="所有日期" show-search @change="handleChange">
          <a-select-option value="近一周">
            近一周
          </a-select-option>
          <a-select-option value="近一个月">
            近一个月
          </a-select-option>
        </a-select>
      </div>
      <!-- 直接搜索内容 -->
      <div class="seacher-data-info fg">
        <input class="input" type="text" placeholder="搜索提问内容" v-model="content" @keyup.enter="getIuputContent">
        <span @click="getIuputContent">搜索</span>
        <span @click="handleReset">重置</span>
      </div>
    </div>
    <!-- 学生提问题目 -->
    <div class="stu-question-data clearfix">
      <!-- <div class="stu-question-subject">
        <span>1. &nbsp;</span>
        <span v-html="subject"></span>
      </div>
      <span v-for="item in option" :key="item.Id">
        <span>{{ item.Option }},</span>
        <span v-html="item.Content"></span>
      </span> -->
      <!--  -->
      <div class="clearfix student-info-data fl">
        <a-avatar size="large" icon="user" class="fl"/>
        <div class="fl">
          <p>王小清</p>
          <p>一（1）班</p>
        </div>
      </div>
      <div class="seize-seat"></div>
      <div class="student-question">
        <span class="question-limit">这道题我看了解析，我还是不明白怎么弄，例如xxxxxxxxxxxxxxxxxxxxxxxxxx…</span><span class="">查看详情</span>
      </div>
      <div class="student-question-infor">
        <p>1,小朋友排队买票，小军从前边数排第3，后边有5个人，这一队共有几人？（    ）</p>
        <p><span>A, 7</span><span>A, 7</span><span>A, 7</span></p>
        <p class="chapter-icon"><img src="@/assets/finemicro/link.png" alt="">一.20以内加减法</p>
        <p>正确答案：<span style="color:#4D5753">C</span></p>
        <p>ta的答案：<span style="color:#FF682C">A</span></p>
        <p style="color:#68BB97">正确答案：<span>平均正确率95%</span></p>
        <span class="to-answer" @click="toAnswer">去回答</span>
      </div>
    </div>
    <!--  -->

  </div>
</template>

<script>
import '@/utils/utils.less'
export default {
  created () {
    // 获取题目题干
    // this.getTextSubject()
    // 获取题干对应选项
    this.getTextOption()
    // this.getTest()
  },
  mounted () {
  },
  data () {
    return {
      options: [
        {
          value: '一年级',
          label: '一年级',
          children: [
            {
              label: '本学期',
              value: '本学期'
            },
            {
              value: '一10以内的数',
              label: '一10以内的数'
            },
            {
              value: '二10以内的数及加减法',
              label: '二10以内的数及加减法'
            }
          ]
        },
        {
          value: '二年级',
          label: '二年级',
          children: [
            {
              value: '一20以内的数及加减法',
              label: '一20以内的数及加减法'
            },
            {
              value: '二识别图形',
              label: '二识别图形'
            }
          ]
        }
      ],
      // 搜索框内容
      content: '',
      // 题干内容
      subject: '',
      // 题目对应选项
      option: [],
      // 隐藏时是否显示DOM结构
      forceRender: true
    }
  },
  methods: {
    // 搜索框内容
    getIuputContent () {
      console.log('搜索input==值', this.content)
    },
    // 重置
    handleReset () {
      this.content = ''
    },
    // 选择框
    handleChange (value) {
      console.log('选择---', value)
    },
    // 联级选择框
    onChange (value) {
      console.log(value)
    },
    // 获取题目题干
    getTextSubject () {
      this.$http.post('/Paper/Exam_Item/GetTheData', {
        id: '4a5bda6f-8505-4982-8563-2d4cdb48d89f'
      }).then(res => {
        console.log(res)
        this.subject = res.Data.Title
        console.log('题干内容---', this.subject)
      })
    },
    // 获取题目选项
    getTextOption () {
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetOptionsByItemId', {
        itemId: '4a5bda6f-8505-4982-8563-2d4cdb48d89f'
      }).then(res => {
        this.option = res.Data
        console.log('题干对应选项---', this.option)
      })
    },
    // 切换tabs面板的回调
    callbreak (key) {
      console.log('切换面板的回调', key)
    },
    toAnswer () {
      this.$router.push({ path: '/Teacher/My_Coach/StudentQuestionDetails' })
    }
  }
}
</script>

<style lang="less" scoped>
  // 学生基本信息
  .ant-switch-checked {
    text-align: center;
    background-color: #E0E7EA;
  }
  .ant-switch-checked::after {
    text-align: center;
    background-color: #68BB97;
  }
  // 搜索框
  // /deep/.ant-cascader-input.ant-input {
  //   width: 195px;
  //   height: 65px;
  //   border-radius: 40px;
  // }
  .student-basic-data {
    height: 127px;
    margin-bottom: 20px;
    padding: 40px 70px 0 50px;
    background-color: #fff;
    border-radius: 5px;
    div:nth-child(2) {
      p {
        margin: 0 10px;
      }
    }
    // 提问信息
  .question-data {
    span {
      display: inline-block;
      margin: 0 35px;
      text-align: center;
      }
    }
  }
   // 搜索内容
  .seacher-data {
    margin-bottom: 25px;
    .seacher-data-info {
      margin-right: 80px;
      span {
        display: inline-block;
        width: 50px;
        height: 35px;
        line-height: 35px;
        margin-left: 10px;
        margin-right: 10px;
        text-align: center;
        border-radius: 5px;
        // background-color: #ddd;
        cursor: pointer;
      }
      span:nth-child(2) {
        background-color: #68BB97;
      }
      span:nth-child(3) {
        background-color: #A2C240;
      }
      .input {
        width: 240px;
        height: 40px;
        text-indent: 20px;
        border-radius: 5px;
        border: 1px solid #ccc;
      }
      .input:focus {
        border-radius: 5px;
        border: 1px solid #00aaff;
        outline:none;
      }
    }
  }
  // 学生提问
  .stu-question-data {
    // display: flex;
    // background-color: #E0EEEC;
    // 占位
    .seize-seat {
      width: 30px;
      background-color: rgba(240, 242, 245);
    }
   .student-info-data {
      width: 200px;
      background-color: #fff;
      padding: 16px 32px 11px 32px;
      border-radius: 5px;
      span {
        margin-right: 10px;
      }
      p {
        margin-bottom: 3px;
      }
    }
    .student-question {
      // width: 80%;
      // height: 71px;
      line-height: 75px;
      margin-left: 225px;
      text-indent: 15px;
      background-color: #E0EEEC;
      border-radius: 5px;
      .question-limit {
        width: 560px;
        margin-bottom: 0;
        white-space: nowrap;
        overflow:hidden;
        text-overflow:ellipsis;
      }
    }
  }
  .student-question-infor {
    position: relative;
    margin-top: 20px;
    padding: 15px 20px;
    background-color: #EAEAEA;
    border-radius: 5px;
    .chapter-icon {
      font-size: 12px;
      img {
        width: 20px;
        margin-right: 8px;
      }
    }
    .to-answer {
      position: absolute;
      top: 21px;
      right: 44px;
      display: inline-block;
      width: 74px;
      height: 32px;
      line-height: 32px;
      text-align: center;
      color: #fff;
      background-color: #68BB97;
      border-radius: 5px;
      cursor: pointer;
    }
  }
  // 散点图
  .situationAnalysis {
    height: 400px;
  }
  // 折线图
  .brokenLine {
    height: 450px;
  }
  .line {
    height: 450px;
  }
  // 饼图
  .pie {
    height: 450px;
  }
</style>
