<template>
  <div class="clearfix" style="display: flex">
    <!-- 题目列表 -->
    <div class="subject-list">
      <div class="subject-specific" v-for="item in paperInfo" :key="item.Id">
        <div >
          <!-- <p>{{ item.Name }}</p> -->
          <div v-for="(ite,index) in item.Items" :key="ite.Id" class="sub-info">
            <p class="m-b line" > <span>{{ index +1 }}. </span><span v-html="ite.Title"></span></p>
            <!-- <p class="m-b">算式：____ + ____ =  (台)</p> -->
            <p v-if="ite.ItemTypeId !== 5" class="choice-info m-b">
              <span v-for="val in ite.Options" :key="val.Id" v-show="val.Type === 1">{{ val.Option }}:
                <span v-html="val.Content"></span>
              </span>
              <!-- <span>B: 1, 2, 4, 7, 11, 17, 23</span>
              <span>C: 1, 16, 2, 14, 3, 12, 4, 10, 5, 8, 6, 6</span> -->
            </p>
            <p class="clearfix"><span @click="removeSub(ite.Id)" class="remove-btn fg cur">移除</span></p>
          </div>
        </div>

      </div>
      <!-- <div class="subject-specific">
        <div class="line">
          <p class="m-b">1   一个车间有2排机器，一排有5台，另一排有6台，一共有多少台机器？</p>
          <p class="m-b">算式：____ + ____ =  (台)</p>
          <p class="choice-info m-b">
            <span>A: 1, 4, 7, 10, 13, 15, 19</span>
            <span>B: 1, 2, 4, 7, 11, 17, 23</span>
            <span>C: 1, 16, 2, 14, 3, 12, 4, 10, 5, 8, 6, 6</span>
          </p>
        </div>
        <p class="clearfix"><span class="remove-btn fg cur">移除</span></p>
      </div> -->
    </div>
    <!-- 右侧列表 -->
    <div class="right-list">
      <p class="right-bigbtn">
        <span @click="completeRelea" class="handle-btn cur" style="background-color: #5B7D85;color:#fff;">完成发布</span>
        <a-modal
          v-model="time"
          :header="null"
          :closable="false"
          @ok="handleOkTime"
          centered
        >
          <p>指向发布:&nbsp;&nbsp;
            <a-checkbox-group @change="classChange1">
              <a-checkbox v-for="item in classInfor" :key="item.ClassId" :value="item.ClassId">{{ item.ClassName }}</a-checkbox>
            </a-checkbox-group>
          </p>
          <div class="m-b release-time">
            发布时间:&nbsp;&nbsp;
            <a-radio-group v-model="value">
              <a-radio :value="1" @click="handleImmediately">立即发送</a-radio>
              <br>
              <a-radio :value="2" @click="handleClick">
                自定义时间:
                <span v-if="value == 1">

                  <!-- :disabledDate="disabledDate" -->
                  <a-date-picker
                    :showTime="{ format: 'HH:mm' }"
                    format="YYYY-MM-DD HH:mm"
                    :disabled-time="disabledDateTime"
                    :disabledDate="disabledDate"
                    :show-time="{ defaultValue: moment('00:00:00', 'HH:mm:ss') }"
                    disabled
                    :default-value="moment(PublishDateTime, 'YYYY-MM-DD HH:mm')"
                  >

                  </a-date-picker></span>
                <span v-else>
                  <a-date-picker
                    :showTime="{ format: 'HH:mm' }"
                    format="YYYY-MM-DD HH:mm"
                    @change="StartTimeChange"
                    :disabledDate="disabledDate"
                    :default-value="moment(PublishDateTime, 'YYYY-MM-DD HH:mm')"
                  >
                  </a-date-picker></span>
              </a-radio>
            </a-radio-group>
          </div>
          <div class="m-b">
            作答截止时间:&nbsp;&nbsp;
            <a-date-picker
              :showTime="{ format: 'HH:mm' }"
              format="YYYY-MM-DD HH:mm"
              @change="TimeChange"
              :disabledDate="disabledDate1"
              :default-value="moment(AnswerEndDateTime, 'YYYY-MM-DD HH:mm')" >

            </a-date-picker></div>
          <a-radio-group v-model="value1" @change="onChange">
            <a-radio :style="radioStyle" :value="0">
              本练习保密(仅本人可看)
            </a-radio>
            <a-radio :style="radioStyle" :value="1">
              共享这份练习
            </a-radio>
          </a-radio-group>
          <div v-show="Area" style="padding: 0 27px">
            <!-- <a-checkbox-group @change="onChangeAreaSchool" :default-value="MultipleChoice">
              <a-row>
                <a-col>
                  <a-checkbox :value="2">
                    共享到全区
                  </a-checkbox>
                  <br>
                  <a-checkbox :value="1">
                    共享到全校
                  </a-checkbox>
                </a-col>
              </a-row>
            </a-checkbox-group> -->
            <a-radio-group v-model="valueArea" @change="handleArea">
              <a-radio :style="radioStyle" :value="2">
                共享到全区
              </a-radio>
              <a-radio :style="radioStyle" :value="1">
                共享到全校
              </a-radio>
            </a-radio-group>
          </div>
        </a-modal>
      </p>
      <p class="right-bigbtn btn2"><span @click="returnTrim" class="handle-btn cur" style="background-color: #A7D262;">返回调整</span></p>
      <p class="txt f-s">{{ paperName }}</p>
      <div v-for="(item,index) in paperInfo" :key="item.Id">
        <p class="f-s-n">{{ index + 1 }}. {{ item.Name }}</p>
        <span v-for="(ite,ind) in item.Items" :key="ite.Id">
          <span class="number">{{ ind + 1 }}</span>
        </span>
        <!-- <p class="f-s-n">{{ index + 2 }}. {{ item.Name }}</p> -->
        <!-- <span class="number">1</span> -->
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    this.paperId = this.$route.query.paperId
    this.getPaperInfo()
  },
  data () {
    return {
      paperId: '',
      paperList: [],
      paperInfo: [],
      paperName: '',
      gradeId: '',
      userId: '',
      // 题目总数
      itemCount: 0,
      // 共享
      IsShare: 0,
      // 是否立即发送
      ImmediatelyPublish: true,
      // 选中班级
      classId: [],
      classInfor: [],
      time: false,
      // 自定义开始时间
      PublishDateTime: '',
      comparePublishTime: '',
      // 作答截止时间
      AnswerEndDateTime: '',
      compareAnswerEndTime: '',
      // 发布时间
      value1: 0,
      value: 1,
      valueArea: '全校',
      // 地区
      Area: false,
      moment,
      radioStyle: {
        display: 'block',
        height: '30px',
        lineHeight: '30px'
      }
    }
  },
  methods: {
    // 获取试卷内容’
    getPaperInfo () {
      this.$http.post('/Paper/Exam_PaperItemMapping/GetPaperInfoById', { paperid: this.paperId }).then(res => {
        console.log('试卷内容', res)
        this.paperList = res.Data
        this.paperInfo = res.Data.QuestionTypes
        this.paperName = res.Data.PaperName
        this.gradeId = res.Data.Grade.slice(0, 1)
        console.log('nianji===', this.gradeId)
        this.itemCount = res.Data.ItemCnt
      })
    },
    // 处理当前时间并赋值
    handleTime () {
      const time = new Date()
      const Time = moment(time).format('YYYY-MM-DD HH:mm')
      this.PublishDateTime = Time
      console.log('立即发送时间默认值------' + this.PublishDateTime)
    },
    // 获取当天时间的 23：59
    getNewDayLastTime () {
      const lastTime = new Date(new Date(new Date().toLocaleDateString()).getTime() + 24 * 60 * 60 * 1000 - 1)
      this.AnswerEndDateTime = moment(lastTime).format('YYYY-MM-DD HH:mm')
      this.compareAnswerEndTime = moment(lastTime)
      console.log(moment(lastTime))

      console.log('获取当天时间的最后时间---', this.AnswerEndDateTime)
    },
    // 不可选择时间
    disabledDate (current) {
      // debugger
      // 不可选过去时间add(-1, 'd')  subtract(0, 'day')
      return current < moment().add(-1, 'd')
      // 不可选未来时间
      // return current && current > Date.now()
    },
    disabledDate1 (current) {
      // debugger
      // 不可选过去时间add(-1, 'd')  subtract(0, 'day')
      return current < moment().subtract(0, 'day')
      // 不可选未来时间
      // return current && current > Date.now()
    },
    // 区校选择
    handleArea (e) {
      this.IsShare = e.target.value
      console.log(e.target.value)
    },
    // 多选框，获取选中的班级
    classChange1 (checkedValues) {
      this.classId = checkedValues
      console.log('选中班级----', checkedValues)
    },
    // 默认选择发布时间
    disabledDateTime () {
      return {
        disabledHours: () => this.range(0, 24).splice(4, 20),
        disabledMinutes: () => this.range(30, 60),
        disabledSeconds: () => [55, 56]
      }
    },
    // 获取自定义开始时间
    StartTimeChange (data, dataString) {
      this.PublishDateTime = dataString
      this.comparePublishTime = data._d
      console.log(moment(data._d).valueOf())
      console.log('改变默认值的时间--------' + this.PublishDateTime, data._d)
    },
    // 获取作答截止时间
    TimeChange (data, dataString) {
      this.AnswerEndDateTime = dataString
      this.compareAnswerEndTime = data._d
      // console.log('截止作答时间---------' + this.AnswerEndDateTime, data._d)
    },
    // 选择是否仅本人查看
    onChange (e) {
      console.log(e.target.value)
      this.IsShare = e.target.value
      if (e.target.value === 1) {
        this.Area = true
      } else {
        this.Area = false
      }
    },
    // 切换自定义时间
    handleClick () {
      this.ImmediatelyPublish = false
      console.log('是否立即发送---', this.ImmediatelyPublish)
    },
    // 立即发送
    handleImmediately () {
      this.ImmediatelyPublish = true
      console.log('是否立即发送---', this.ImmediatelyPublish)
    },
    // 完成发布
    completeRelea () {
      this.time = true
      // 获取老师对应班级
      this.$http.post('/HighFrequency/HighFrequency/GetClassByUserGrade', { userId: this.userId, grade: this.gradeId }).then(res => {
        this.classInfor = res.Data
        console.log('老师对应班级---', this.classInfor)
      })
      // 当前时间
      this.handleTime()
      // 当天最后时间
      this.getNewDayLastTime()
    },
    // 发布时间完成
    handleOkTime () {
      if (this.classId.length === 0) {
        return this.$message.error('请选择需要发布的班级')
      }
      if (this.PublishDateTime === '' || this.AnswerEndDateTime === '') {
        return this.$message.error('请注意发布时间的选择')
      }
      if (moment(this.comparePublishTime).valueOf() > moment(this.compareAnswerEndTime).valueOf()) {
        return this.$message.error('截止时间不能小于发布时间')
      }
      // 难度   年级
      this.$http.post('/Paper/Exam_Paper/PublishPaper', { PaperId: this.paperId, ClassIds: this.classId, ImmediatelyPublish: this.ImmediatelyPublish, PublishDateTime: this.PublishDateTime, AnswerEndDateTime: this.AnswerEndDateTime, IsShare: this.IsShare, DifficultyStar: 0, ItemCount: this.itemCount, ItemInfo: '' }).then(res => {
        console.log('完成发布', res)
        this.time = false
        this.$router.push({ path: '/Teacher/My_Task/HighWrong/ThisRelease',
          query: {
            paperName: this.paperName
          } })
      })
    },
    // 移除题目
    removeSub (Id) {
      this.$http.post('/Paper/Exam_PaperItemMapping/Remove', { id: Id }).then(res => {
        console.log('移除题目', res)
        this.getPaperInfo()
      })
    },
    // 返回调整
    returnTrim () {
      this.$router.go(-1)
    }
  }
}
</script>

<style lang="less" scoped>
  .m-b {
    margin-bottom: 20px;
  }
  .txt {
    margin-bottom: 32px;
    text-align: center;
  }
  .f-s {
    font-size: 20px;
  }
  .f-s-n {
    font-size: 18px;
    margin-bottom: 15px;
  }
  .sub-info {
    margin-bottom: 16px;
    padding: 16px;
    background-color: #fff;
    border-radius: 5px;
  }
  // 题目列表
  .subject-list {
    width: 71%;
    height: 800px;
    margin-right: 15px;
    font-size: 12px;
    overflow: scroll;
    background-color: rgba(240, 242, 245);
    border-radius: 5px;
    .subject-specific {
      margin-bottom: 20px;
      padding: 16px 16px 20px 12px;
      font-size: 14px;
      // background-color: #fff;
      border-radius: 5px;
        // 下划线
        .line {
          margin-bottom: 10px;
          padding-bottom: 18px;
          border-bottom: 1px solid #ddd;
        }
      //   p {
      //     margin-bottom: 20px;
      // }
      // 移除按钮
      .remove-btn {
        margin-right: 30px;
        padding: 5px 20px;
        color: #fff;
        background-color: #A8D261;
        border-radius: 5px;
      }
    }
    // 选择题
    .choice-info {
      span {
        margin-right: 60px;
      }
    }
  }
  // 右侧列表
  .right-list {
    width: 25%;
    height: 800px;
    padding: 0 20px;
    background-color: #fff;
    border-radius: 5px;
    .right-bigbtn {
      text-align: center;
    }
    .handle-btn {
      display: inline-block;
      margin: 20px 0;
      padding: 10px 50px;
      background-color: #ddd;
      border-radius: 5px;
    }
    .btn2 {
      margin-bottom: 60px;
    }
    .number {
      display: inline-block;
      width: 41px;
      height: 40px;
      line-height: 40px;
      margin-right: 24px;
      margin-bottom: 24px;
      // padding: 5px;
      font-size: 14px;
      text-align: center;
      color: #5B7D85;
      border: 1px solid #839398;
      border-radius: 5px;
    }
  }
</style>
