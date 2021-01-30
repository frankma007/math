<template>
  <div>
    <!--  题目题干 -->
    <div class="testQuestion">
      <span>{{ testNum }},&nbsp;</span>
      <span v-html="testQuestion.Title"></span>
    </div>
    <!-- 题目选项 -->
    <span class="test-option" v-for="item in options" :key="item.Id">
      <span>{{ item.Option }}.&nbsp;</span>
      <span v-html="item.Content"></span>
    </span>
    <!-- 试卷章节名称 -->
    <p class="chapter-name"><img src="@/assets/finemicro/link.png" alt=""> <span @click="toSubjectAnalysis">{{ paperChapterName }}</span></p>
    <!-- 正确率 -->
    <div class="accuracy">
      <a-table
        :columns="columns"
        :data-source="data"
        bordered
        :pagination="pagination">
      </a-table>
    </div>
    <!-- 详情统计 -->
    <div></div>
    <!-- 辅导讲解 -->
    <div class="coach-explain">
      <div style="position: relative">
        <p>辅导讲解</p>
        <textarea></textarea>
        <div></div>
        <div>
          <p @click="playSound" v-if="soundTime !== '' && soundTime > 0" class="sound-show" :class="{'active': active === '1'}">{{ soundTime }}</p>
        </div>
        <!-- 操作按钮 -->
        <div class="operation-btn">
          <span>取消</span>
          <span>完成</span>
        </div>
        <!-- 录音弹窗 -->
        <div v-if="soundPopup" class="sound-popup">
          <img @click="soundShow" v-show="stopItHide" src="@/assets/correcting/录音弹窗.png" alt="">
          <img @click="hideSound" v-show="stopIt" src="@/assets/correcting/点击停止.png" alt="">
          <p v-show="stopItHide">点击说话</p>
          <p v-show="stopIt">点击停止</p>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import Recorder from 'js-audio-recorder'
const columns = [
  {
    title: '统计',
    dataIndex: 'name',
    key: 'name',
    align: 'center'
  },
  {
    title: '我班',
    dataIndex: 'age',
    key: 'age',
    align: 'center'
  },
  {
    title: '我校',
    dataIndex: 'address',
    key: 'address',
    align: 'center'
  },
  {
    title: '全区',
    dataIndex: 'area',
    key: 'area',
    align: 'center'
  }
]
const data = [
  {
    key: '1',
    name: '平均正确率',
    age: '85%',
    address: '65%',
    area: '75%'
  },
  {
    key: '2',
    name: '平均用时',
    age: '95%',
    address: '72%',
    area: '86%'
  }
]
const recorder = new Recorder()

export default {
  created () {
    // 当前书卷ID
    this.paperId = this.$route.query.paperId
    // 当前试题ID
    this.testPaperId = this.$route.query.id
    // 当前试题编号
    this.testNum = this.$route.query.subject
    // 获取单个题目的题干
    this.getStem()
    // 获取题目对应选项
    this.getTestOption()
    // 获取章节名称
    this.getChapterName()
  },
  watch: {
    $route () {
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/My_Coach/AccurateCoach?' + ChangId) {
        this.getStem()
        this.getTestOption()
      }
    }
  },
  data () {
    return {
      soundTime: '',
      // 录音动画
      active: '',
      // 录音弹窗
      soundPopup: false,
      stopIt: false,
      stopItHide: false,
      // 当前试卷ID
      paperId: '',
      // 试卷章节名称
      paperChapterName: '',
      // 当前试题编号
      testNum: '',
      // 当前试题ID
      testPaperId: '',
      // 单个题目题干
      testQuestion: [],
      // 题目对应的选项
      options: [],
      // 正确率行
      columns,
      // 正确率列
      data,
      // 表格的分页
      pagination: false
    }
  },
  methods: {
    // 录音
    soundRecording () {
      console.log('111')
      this.soundPopup = true
      this.stopItHide = true
      this.stopIt = false
    },
    // 显示停止/ 开始录音
    soundShow () {
      this.stopItHide = false
      this.stopIt = true

      recorder.start().then(() => {

      }, (error) => {
        // 出错了
        console.log(`${error.name} : ${error.message}`)
        alert('无法启动音频源')
      })
    },
    // 停止录音
    hideSound () {
      this.soundPopup = false
      recorder.stop()
      this.soundTime = parseInt(recorder.duration)
      // recorder.downloadWAV()
      recorder.onplayend = this.playend
      // this.soundTime.splice(0, 2)
      console.log(this.soundTime)
      if (this.soundTime < 1) {
        this.$message.error('说话时间太短')
      }
    },
    // 播放
    playSound () {
      console.log('11111')
      this.active = '1'
      recorder.play()
    },
    playend () {
      console.log('播放完成')
      this.active = ''
    },
    // 获取单个题目的题干
    getStem () {
      this.$http.post('/Paper/Exam_Item/GetTheData', {
        id: this.testPaperId
      }).then(res => {
        this.testQuestion = res.Data
        // console.log('题干信息', this.testQuestion)
      })
    },
    // 获取题目对应选项
    getTestOption () {
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetOptionsByItemId', {
        itemId: this.testPaperId
      }).then(res => {
        this.options = res.Data
        // console.log('题目对应选项---', this.options)
      })
    },
    // 获取章节名称
    getChapterName () {
      this.$http.post('/Paper/Exam_Paper/GetTheData', {
        id: this.paperId
      }).then(res => {
        console.log('章节名称---', res)
        this.paperChapterName = res.Data.ChapterName
        console.log('试卷的基本信息---', this.paperChapterName)
      })
    },
    // 跳转逐题分析
    toSubjectAnalysis () {
      this.$router.push({ path: '/Teacher/My_Task/InspectionWorkData',
        query: {
          id: '2'
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
  // 表格
  /deep/.ant-table-thead {
    /deep/ .ant-table-align-center{
      color: #fff;
      background-color: #68BB97;
    }
  }
   /deep/.ant-table-tbody tr:nth-child(odd) {
      background-color: #F4F4F4;
  }
  // 题目题干
  .testQuestion {
    margin-bottom: 20px;
    span {
      font-size: 18px;
    }
  }
  // 题目选项
  .test-option {
    display: inline-block;
    margin-right: 35px;
    margin-bottom: 25px;
    span {
      margin-right: 8px;
    }
    span:nth-child(1) {
      font-size: 20px;
    }
    // 选项图片
    span /deep/ img {
      // width: 30px;
    }
  }
  // 试卷章节名称
  .chapter-name {
    span {
      display: inline-block;
      width: 137px;
      height: 25px;
      line-height: 25px;
      text-align: center;
      vertical-align: middle;
      border-radius: 5px;
      background-color: #ccc;
      cursor: pointer;
    }
    img {
      width: 20px;
      margin-right: 5px;
    }
  }
  // 正确率
  .accuracy {
    width: 600px;
    margin-top: 20px;
  }
   // 辅导讲解
  .coach-explain {
    display: flex;
    margin-top: 30px;
    div:nth-child(1) {
      width: 100%;
      // height: 600px;
      margin-right: 15px;
      padding: 30px 50px;
      background-color: #fff;
      border-radius: 5px;
      textarea {
        width: 100%;
        height: 200px;
        border: 1px solid #ccc;
        border-radius: 5px;
        resize: none;
      }
    }
    div:nth-child(2) {
      width: 38%;
      // height: 600px;
      padding: 50px;
      background-color: #fff;
      border-radius: 5px;
      p {
        margin-top: 20px;
        text-align: center;
        img {
          cursor: pointer;
        }
      }
    }
  }
   @keyframes soundShow
    {
    0%   {background:  url('../../../assets/correcting/播放1.png');}
    25%  {background:  url('../../../assets/correcting/播放2.png');}
    50% {background:  url('../../../assets/correcting/播放3.png');}
    100% {background:  url('../../../assets/correcting/播放3.png');}
    }
    .active {
      animation: soundShow 2s linear infinite;
    }
    .sound-show {
    // animation: soundShow 2s linear infinite;
    width: 201px;
    height: 39px;
    line-height: 39px;
    text-align: center;
    background: url('../../../assets/correcting/录音show.png');
    cursor: pointer;
  }
   // 录音弹窗
  .sound-popup {
    width: 206px;
    height: 206px;
    position: absolute;
    left: 50%;
    top: 50%;
    padding-top: 61px;
    transform: translate(-109px, -154px);
    border-radius: 5px;
    text-align: center;
    background-color: #959595;
    cursor: pointer;
  }

</style>
