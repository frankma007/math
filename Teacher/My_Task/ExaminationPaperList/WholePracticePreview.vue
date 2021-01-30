<template>
  <div style="display: flex;">
    <div class="subject" >
      <p class="sub-title">{{ paperName }}</p>
      <div class="sub-info" v-for="(item, index) in paperData" :key="item.ItemId" :id="item.ItemId">
        <span style="font-size: 18px;color: #000">{{ index + 1 }}. </span>
        <span class="sub-data" v-html="item.Title"></span>
        <!-- 试题选项 -->
        <div style="padding-left: 14px">
          <!-- @change="SelectOptions" -->
          <a-radio-group @change="SelectOptions">
            <a-radio :style="radioStyle" :value="item.ItemId + '|' + k.Option" v-for="(k, j) in item.AuditPaperItemAnswers" :key="j" v-show="k.Type === 1">
              <span
                v-if="k.Type === 1"
                style="fontSize: 20px;"
              >{{ k.Option }}:&nbsp;&nbsp;</span>
              <span class="p-inline" v-if="k.Type === 1" v-html="k.Content" style="fontSize: 18px;display:inline-block"></span>
            </a-radio>
          </a-radio-group>
        </div>
        <div class="ana-show" v-for="(ite,ind) in item.AuditPaperItemAnswers" :key="ind">
          <p @click="ViewResolution(item.ItemId)" class="cur" style="color: #68BB97" v-if="ite.Type === 10">查看解析</p>
          <div v-if="ite.Type === 10 && anaShow === item.ItemId" v-html="ite.Content"></div>
        </div>
      </div>

    </div>
    <!-- <div class="sub-btn">
      <p class="clearfix"><span class="fl">{{ newSubject }}/{{ paperNum }}题</span><span class="fg">{{ callinTime }}</span></p>
      <div class="title-btn" @scroll="onScroll">
        <a
          v-for="(i, t) in paperData"
          :key="t"
          class="title-btn-whole"
          :class="{ 'color': color === i.ItemId }"
          @click="switchTitle(t, i.ItemId, i.TypeId)"
        >
          {{ t + 1 }}
        </a>
      </div>
      <p class="submit"><a-button
        @click="viewsSubject"
      >
        提交
      </a-button></p>
    </div> -->
  </div>
</template>

<script>
export default {
  created () {
    this.paperId = this.$route.query.id
    // 获取试卷基本信息
    this.getTitleList()
    // 获取试卷的名称
    this.getTestPaperName()
    this.getAnswer()
    // this.getSubjectInfo()
  },
  mounted () {
    this.timeRecord(true)
    window.addEventListener('scroll', this.getScrollPosition, false)
  },
  destroyed () {
    window.removeEventListener('scroll', this.getScrollPosition, false)
  },
  data () {
    return {
      // 试卷id
      paperId: '',
      // 计时器
      callinTime: '',
      // 试卷信息
      paperData: [],
      // 试卷名称
      paperName: '',
      // 试卷题数
      paperNum: '',
      // 当前题目标号
      newSubject: '1',
      // 切换题目颜色
      color: '',
      //  // 题目切换
      key: 1,
      // 填空题答案
      testAnswer: [],
      // 单选题答案
      testChoiceAnswer: [],
      radioStyle: {
        display: 'block',
        lineHeight: '30px'
      },
      // 查看解析
      anaShow: ''
    }
  },
  watch: {
    $route () {
      this.paperId = this.$route.query.id
      this.anaShow = ''
      if (this.$route.fullPath === '/Teacher/My_Task/ExaminationPaperList/WholePracticePreview?id=' + this.paperId) {
        // this.timeRecord(false)
        this.timeRecord(true)
        window.addEventListener('scroll', this.getScrollPosition, false)
        this.getTitleList()
        this.getTestPaperName()
      } else {
        this.timeRecord(false)
        window.removeEventListener('scroll', this.getScrollPosition, false)
      }
    }
  },
  methods: {
    onScroll () {
      // const scrollItems = document.querySelectorAll('.sub-info')
    },
    // 计时器
    timeRecord (bolean) {
      const _this = this
      let hour, minute, second
      hour = minute = second = 0
      if (bolean === true) {
        _this.timer = setInterval(() => {
          if (second >= 0) {
            second = second + 1
          }
          if (second >= 60) {
            second = 0
            minute = minute + 1
          }
          if (minute >= 60) {
            minute = 0
            hour = hour + 1
          }
          _this.callinTime = hour + ':' + minute + ':' + second + ''
        }, 1000)
      } else {
        window.clearInterval(_this.timer)
      }
    },
    // 监听浏览器滚动事件
    // getScrollPosition () {
    //   // 滚动条距顶部距离
    //   const top = document.documentElement.scrollTop || document.body.scrollTop
    //   if (top > 115) {
    //     document.getElementsByClassName('sub-btn')[0].style.top = '0'
    //   }
    //   if (top < 115) {
    //     document.getElementsByClassName('sub-btn')[0].style.top = '130px'
    //   }
    //   // this.onScroll()
    // },
    // 获取试卷基本信息
    getTitleList () {
      this.$http.get('/Paper/Exam_Item/GetAuditPaperItemsList', {
        params: {
          paperId: this.paperId
        }
      }).then(res => {
        this.paperData = res.Data
        console.log('试卷基本信息---', res)
        this.paperNum = res.Data.length
        this.color = res.Data[0].ItemId
        console.log(this.paperData)
        this.paperData.forEach((item, index) => {
          // debugger
          const reg = /(#&\d+@)/g
          // const reg = /(#@)/g
          // const id = item.ItemId
          // console.log(id)
          const inputele = '<input  type="text" itemId= "' + item.ItemId + '" style="width: 100px;border: 0px; BACKGROUND-COLOR: transparent; text-align:center; border-bottom:1px solid grey;" name="' + 'input' + '"/>'
          const stem = item.Title.replace(reg, inputele)
          item.Title = stem
          // debugger

          // console.log('刚进页面')
          // console.log(this.itemId)
        })
      })
    },
    // 获取试卷的信息 重新渲染
    // getSubjectInfo () {
    //   // debugger
    //   this.$http.post('/Paper/Exam_PaperItemMapping/GetPaperInfoById', {
    //     paperid: '1292634508184522752'
    //   }).then(res => {
    //     console.log('保存后的试卷题目---', res)
    //   })
    // },
    // 获取试卷的名称
    getTestPaperName () {
      this.$http.post('/Paper/Exam_Paper/GetTheData', { id: this.paperId }).then(res => {
        this.paperName = res.Data.Title
        console.log(this.paperName)
      })
    },
    // 切换题目
    switchTitle (index, ItemId, typeId) {
      // console.log(index)
      document.getElementById(ItemId).scrollIntoView({ behavior: 'smooth' })
      this.newItemId = ItemId
      this.newSubject = index + 1
      // 切换题目(当前题目对应下标)
      this.key = index
      // 切换题目颜色 (当前题目的id)
      this.color = ItemId
      // 切换题目时的类型ID
      this.newTextType = typeId
      // 处理试卷切换题目选项
      this.switchTitleOption(ItemId)
      // this.updateFormat(this.paperData)
    },
    // 处理切换题目时选项
    switchTitleOption (ItemId) {
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetOptionsByItemId', {
        itemId: ItemId
      }).then(res => {
        this.option = res.Data
        // console.log('当前题目选项---', this.option)
      })
    },
    // 选择题的选项变化
    SelectOptions (e) {
      const choiceAnswer = e.target.value
      console.log(choiceAnswer)
      const choiceArray = choiceAnswer.split('|')
      console.log(choiceArray)

      const itemid = choiceArray[0]
      const selectOption = choiceArray[1]

      const exist = this.testChoiceAnswer.some(item => {
        if (item.ItemId === itemid) {
          return true
        }
      })

      if (!exist) {
        this.testChoiceAnswer.push({
          ItemId: itemid,
          Answer: selectOption
        })
        this.selectOOption = selectOption
        // console.log('添加选择题答案：' + selectOption)
        // console.log(this.selectOption)
      } else {
        this.testChoiceAnswer.forEach(item => {
          if (item.ItemId === itemid) {
            item.Answer = selectOption
            this.selectOption = selectOption

            // console.log('更新选择题答案：' + selectOption)
          }
        })
      }
      console.log(this.testChoiceAnswer)
    },
    // 查看解析
    ViewResolution (id) {
      this.anaShow = id
    },
    // 提交试卷
    viewsSubject () {
      const that = this
      // this.updateFormat()
      const modal = this.$confirm({
        title: '温馨提示',
        content: '是否确认提交练习',
        okText: '确认',
        cancelText: '取消',
        // 确定按钮回调
        onOk (e) {
          // console.log(e)
          // 停止计时
          that.timeRecord(false)
          // 清除提示框
          modal.destroy()
          const inputs = document.getElementsByTagName('input')

          for (var i = 0; i < inputs.length; i++) {
            const itemId = inputs[i].getAttribute('itemid')
            const each = inputs[i].value

            const exist = that.testAnswer.some(item => {
              if (item.ItemId === itemId) {
                return true
              }
            })

            if (!exist) {
              that.testAnswer.push({
                ItemId: itemId,
                AnswerArray: [each],
                Answer: each
              })
              console.log('添加填空题答案：' + each)
            } else {
              that.testAnswer.forEach(item => {
                // console.log(item)
                if (item.ItemId === itemId) {
                  item.AnswerArray.push(each)
                  console.log('更新填空题答案：' + item.Answer)
                }
              })
            }
          }

          for (var x = 0; x < that.testAnswer.length; x++) {
            that.testAnswer[x].Answer = that.testAnswer[x].AnswerArray.join('|')
            console.log(that.testAnswer[x].Answer)
          }
          that.eachAnswer = [...that.testChoiceAnswer, ...that.testAnswer]
          const EachAnswer = that.eachAnswer.filter(item =>
            item.ItemId !== null
          )
          that.eachAnswer = EachAnswer
          console.log('提交的完整答案----', that.eachAnswer)
          if (that.eachAnswer.length !== that.paperData.length) {
            return that.$message.error('请提交完整的答案')
          } else {
            that.$message.success('提交成功')
            that.$router.push({ path: '/Paper/TeacherAuditPaper/TestExamine',
              query: {
                paperId: that.paperId,
                answerJson: JSON.stringify(that.eachAnswer)
              }
            })
          }
        }
      })
    },
    getAnswer (id) {
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetAnswerDataByItemId', { itemId: '11177' }).then(res => {
        this.answer = res.Data.Content

        this.completion = this.answer.split('|')
        console.log('答案----', res)
        console.log('答案数组---', this.completion)

        this.mulAnswer = res.Data.Content.split('')
        // console.log(res.Data.Content)
        // console.log(this.mulAnswer)
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .color {
    background-color: #66BC9A;
  }
  // 题目
  .subject {
    width: 100%;
    // height: 500px;
    margin-right: 10px;
    // padding: 50px;
    // background-color: #fff;
    border-radius: 5px;
    // 试卷名称
    .sub-title {
      font-size: 20px;
      color: #49534E;
      text-align: center;
    }
    // 试卷主体
    .sub-info {
      margin-bottom: 50px;

      padding: 10px 15px;
      padding-top: 30px;
      border-radius: 5px;
      background-color: #fff;
      border-bottom: 1px dotted #ddd;
      // padding-bottom: 50px;
      p {
        display: inline-block;
      }
      /deep/.ant-radio-wrapper {
        height: 100%;
      }
    }
  }
  // 题目对应按钮
  .sub-btn {
    position: fixed;
    right: 0;
    top: 130px;
    height: 100%;
    width: 25%;
    // height: 300px;
    padding: 20px;
    background-color: #fff;
    border-radius: 5px;
    // 对应按钮
    .title-btn-whole {
      display: inline-block;
      width: 41px;
      height: 40px;
      line-height: 40px;
      margin: 16px 16px 0 0;
      text-align: center;
      border: 1px solid #ccc;
      border-radius: 5px;
      cursor: pointer;
    }
  }
  // 提交
  .submit {
    margin-top: 75px;
    text-align: center;
    button {
      display: inline-block;
      width: 114px;
      height: 50px;
      line-height: 50px;
      color: #fff;
      background-color: #61BB96;
      border-radius: 25px;
    }
  }

  .p-inline {
    p {
      display: inline-block;
    }
  }
  .sub-data {
    // display: inline-block;
    font-size: 20px;
    color: #000;
    p {
      display: inline-block;
    }
  }
  .ana-show {
    padding-left: 20px;
    // padding-top: 15px;
  }
</style>
