<template>
  <div>
    <p class="fn-col">编辑推送</p>
    <!-- 试题列 -->
    <!-- <div class="high-wrong">
      <p class="high-p">高频错题（1）</p>
      <div class="process">
        <div style="text-align:center;">
          <a-progress type="circle" :percent="80" :width="50" :format="(percent) =>{if(percent=='100'){return '100%'}else{return percent+'%'}}"/>
        </div>
        <div class="number">
          1
        </div>
      </div>
    </div> -->
    <!-- 选入题目 -->
    <div class="selected-sub">
      <p style="font-size:20px;" class="selec-p clearfix">
        <span class="fl">已选入题目</span>
        <span class="fg" v-show="bigTitleDefault">{{ paperName }}<img id="myTitleBtn" @click="editBigTile" class="cur" src="@/assets/teacher/布置作业编辑.png" alt=""></span>
        <span><span @click="editTitle" v-if="BigTitle" class="edit-title fg cur">确定</span><input class="sel-input fg" type="text" v-if="BigTitle" v-model="paperName"></span>
      </p>
      <p class="selec-p2">试题总量（{{ paperInfo.length }}）</p>
      <p class="selec-type">
        <span class="type-flex" v-for="item in paperInfo" :key="item.Id">
          <span>{{ item.Name }}</span>
          <span class="fg"><span class="sel-col">{{ item.Items.length }}</span>题</span>
        </span>
        <!-- <span class="type-flex">
          <span>多选题</span>
          <span class="fg"><span class="sel-col">1</span>题</span>
        </span>
        <span class="type-flex">
          <span>填空题</span>
          <span class="fg"><span class="sel-col">1</span>题</span>
        </span> -->
        <span @click="toAssign" class="to-assign cur">
          去布置
        </span>
      </p>
    </div>
    <!-- 试题内容 -->
    <div v-for="(item) in paperInfo" :key="item.Id">
      <p style="font-size:16px;margin-top: 23px">{{ item.Name }}</p>
      <div class="sub-infor" v-for="(val, num) in item.Items" :key="val.Id">
        <p>{{ num + 1 }}. <span v-html="val.Title"></span></p>
        <p class="choice-info" v-show="val.ItemTypeId !== 5" >
          <span v-show="ite.Type === 1" v-for="ite in val.Options" :key="ite.Id">{{ ite.Option }}: <span v-html="ite.Content"></span></span>
          <!-- <span>B: 1, 2, 4, 7, 11, 17, 23</span>
          <span>C: 1, 16, 2, 14, 3, 12, 4, 10, 5, 8, 6, 6</span> -->
        </p>
        <div v-for="(ite, ind) in val.Options" :key="ind">
          <p class="ana-info" v-show="ite.Type === 8">【答案】：<span v-html="ite.Content"></span></p>
        </div>
        <div v-for="(ite) in val.Options" :key="ite.Id">
          <p v-show="ite.Type === 10" >【解析】：<span v-html="ite.Content"></span></p>
        </div>
        <p class="clearfix hanle-btn">
          <span v-show="val.IsOnce === 'yes'" class="fl cur">曾布置过</span>
          <span @click="removeSub(val.Id)" class="fg cur">移除</span>
          <span @click="singleEditSubject(val.ItemId,item.TypeId,item.Type,item.Id)" class="fg cur">编辑</span>
        </p>
      </div>
    </div>
    <a-drawer
      title=""
      placement="right"
      :visible="MakeTopic"
      @close="onCloseMakeTopic"
      width="60%"
      height="100%"
      :drawerStyle="{ backgroundColor: '#FAF7F8' }"
      :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.1)'}"
      destroyOnClose
    >
      <div class="make-topic clearfix">
        <div class="make-title" >
          <!-- 编写的提示 -->
          <p class="f-c m-b">
            <span class="make-sub-title sub-style">题干</span>
            <span style="color: #FF8D60;">
              <span v-if="titleCheck">
                <img src="@/assets/teacher/提示编组.png" alt="">&nbsp;
                <span style="vertical-align: middle;">题干不能为空</span>
              </span>
            </span>
          </p>
          <!-- {{ subTitle }} -->
          <TinyMceEditor
            @keyup.enter="keyUp"
            ref="edit"
            id="edi"
            :height="268"
            :subTitle="subTitle"
            :catchData="catchData"
            v-model="subTitle"
          ></TinyMceEditor>

        </div>

        <!-- 选择题 -->
        <div v-if="questionId === 2 || questionId === 10 || questionId === 11" class="make-select fl">
          <p class="f-c sub-style" >答案选项  <span v-if="questionId === 2">单选题</span>
            <span v-if="questionId === 10">多选题</span>
            <span v-if="questionId === 11">判断题</span>
          </p>

          <!-- <a-radio-group @change="onChangeSlect" v-model="selectValue">
                      <a-radio :value="2">单选</a-radio>
                      <a-radio :value="10">多选</a-radio>
                      <a-radio :value="11">判断题</a-radio>
                    </a-radio-group> -->
          <div>
            <p v-for="subOption in subOptions" :key="subOption.id" v-show="questionId === 2">
              <span class="options" >

                <a-radio-group @change="onChangeSlect" v-model="alonevalue">
                  <a-radio :value="subOption.Option">{{ subOption.Option }} 选项</a-radio>
                </a-radio-group>
                <span v-if="subOption.Content === '' && selectCheck" ><img src="@/assets/teacher/提示编组.png" alt="">选项不能为空</span>
                <!-- {{ subOption.Option }}选项 -->
              </span>
              <!-- <span v-if="subOption.Content !== ''" style="color:#FF682C">选项不能为空</span> -->
              <TinyMceEditor v-model="subOption.Content" :height="100"></TinyMceEditor>
            </p>
            <p v-show="questionId === 10">
              <!-- v-if="MultipleChoice.length !== 0" -->
              <span class="options" >
                <a-checkbox-group @change="onChangeduo" :defaultValue="MultipleChoice">
                  <a-row>
                    <a-col v-for="subOption in subOptions" :key="subOption.id" :span="24">
                      <a-checkbox :value=" subOption.Option">
                        {{ subOption.Option }}选项
                      </a-checkbox>
                      <span v-if="subOption.Content === '' && selectCheck"><img src="@/assets/teacher/提示编组.png" alt="">选项或答案不能为空</span>
                      <TinyMceEditor v-model="subOption.Content" :height="100"></TinyMceEditor>
                    </a-col>
                  </a-row>
                </a-checkbox-group>
                <!-- {{ subOption.Option }}选项 -->
              </span>

            </p>
            <p v-for="subOption in subJudgeOptions" :key="subOption.id" v-show="questionId === 11">
              <span class="options" >
                <a-radio-group @change="onChangeSlect" v-model="judge">
                  <a-radio :value="subOption.Option">{{ subOption.Option }} {{ subOption.Content }}</a-radio>
                </a-radio-group>
                <!-- {{ subOption.Option }}选项 -->
              </span>
            </p>
          </div>

          <p v-if="questionId === 10 || questionId === 2" style="float: right;font-size: 18px;margin-bottom: 0;">
            <span class="f-c">选项</span>
            <a-icon type="plus-circle" @click="AddOptions"/>&nbsp;&nbsp;&nbsp;&nbsp;
            <a-icon type="minus-circle" @click="ReduceOptions"/>
          </p>
        </div>
        <!-- 填空题 -->

        <div class="make-analysis" style="float:right">
          <p class="f-c m-b">
            <span class="m-r sub-style">解析</span>
            <span v-if="analysisCheck">
              <img src="@/assets/teacher/提示编组.png" alt="">&nbsp;
              <span style="vertical-align: middle;">解析不能为空</span>
            </span>
          </p>
          <TinyMceEditor :height="160" v-model="subAnalysis"></TinyMceEditor>
        </div>
      </div>
      <div class="edit-subject-operation">
        <span @click="cancelEdit">取消</span>
        <a-modal
          :visible="EditCance"
          v-model="EditCance"
          title="温馨提示"
          width="630px"
          @ok="handleOkEdit"
          @cancel="handleCancelEdit"
          centered
        >
          <p>关闭后，正在编辑的题目将不做任何变更！是否保存</p>
        </a-modal>
        <span @click="handleSave">保存</span>

      </div>
    </a-drawer>
    <!-- <div class="sub-infor">
      <p>1 选择正确的算式（ ）</p>
      <p class="choice-info">
        <span>A: 1, 4, 7, 10, 13, 15, 19</span>
        <span>B: 1, 2, 4, 7, 11, 17, 23</span>
        <span>C: 1, 16, 2, 14, 3, 12, 4, 10, 5, 8, 6, 6</span>
      </p>
      <p>【答案】：B</p>
      <p class="ana-info">【解析】：把每个算式的得数都计算出来，只有7+5=12，所以选B。</p>
      <p class="clearfix hanle-btn">
        <span class="fg cur">编辑</span>

      </p>
    </div> -->
  </div>
</template>

<script>
import TinyMceEditor from '@/components/TinyMCE/tinyMce'
export default {
  components: {
    TinyMceEditor
  },
  watch: {
    $route () {
      // this.paperId = this.$route.query.paperId
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/My_Task/HighWrong/EditPush?' + ChangId) {
        // this.getPaperInfo()
        this.$router.go(0)
      }
    }
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    this.paperId = this.$route.query.paperId
    this.knowledge = this.$route.query.chapterId
    this.getPaperInfo()
  },
  data () {
    return {
      userId: '',
      // 章节id
      knowledge: '',
      // 试卷内容
      paperList: [],
      paperInfo: [],
      bigTitleDefault: true,
      BigTitle: false,
      paperName: '',
      // 题型ID
      questionId: '',
      // 编辑弹窗
      MakeTopic: false,
      // 取消编辑弹窗
      EditCance: false,
      // 题干内容
      subTitle: '',
      // 编辑解析
      subAnalysis: '',
      // 题干校验
      titleCheck: false,
      // 解析校验
      analysisCheck: false,
      catchData: '',
      // 年级
      grade: '',
      // 默认选择空
      subOptions: [ { Option: 'A' }, { Option: 'B' }, { Option: 'C' } ],
      subJudgeOptions: [{ Option: 'A', Content: '对' }, { Option: 'B', Content: '错' }],
      MultipleChoice: [],
      alonevalue: '',
      judge: ''
    }
  },
  methods: {
    // 修改试卷名
    editBigTile () {
      this.bigTitleDefault = false
      this.BigTitle = true
    },
    editTitle () {
      this.$http.post('/Paper/Exam_Paper/ChangePaperTitle', { paperid: this.paperId, paperName: this.paperName }).then(res => {
        console.log('修改试卷名称', res)
        // this.paperList.PaperName = res.Data
      })
      this.bigTitleDefault = true
      this.BigTitle = false
      console.log('试卷名称', this.paperName)
    },
    // 获取试卷内容’
    getPaperInfo () {
      this.$http.post('/Paper/Exam_PaperItemMapping/GetPaperInfoById', { paperid: this.paperId }).then(res => {
        console.log('试卷内容', res)
        this.paperList = res.Data
        this.paperInfo = res.Data.QuestionTypes
        // var options = res.Data.QuestionTypes
        this.paperName = res.Data.PaperName
        this.grade = res.Data.Grade
      })
    },
    onChangeSlect (e) {
      console.log(e.target.value)
      this.answer = e.target.value
      console.log('答案---', this.answer)
    },
    onChangeduo (checkedValues) {
      console.log('checked = ', checkedValues)
      this.answer = checkedValues
    },
    // 保存编辑题目
    handleSave () {
      console.log('题干---', this.subTitle)
      console.log('选项类型---', this.selectValue)
      console.log('解析---', this.subAnalysis)
      console.log('选项---', this.subOptions)
      console.log('填空---', this.completion)
      console.log('答案---', this.answer)
      console.log('题型ItemId---', this.questionItemType)
      console.log('自身类型id---', this.myTypeQuestionId)
      console.log('编辑单个题目id--', this.singleEditQuestionId)
      console.log('同意答案---', this.allSynonymAnswer)
      console.log('引用的id---', this.IsQuote)
      console.log('年级---', this.grade)

      // console.log(this.answer.split(''))
      // 答案的格式编写
      this.saveAnswerWrite()
      // 保存提示
      this.checkSaveRule()

      this.$http.post('/Paper/Exam_Item/UpdateQuoteItem', {
        UserId: this.userId,
        analysis: this.subAnalysis,
        itemType: this.questionItemType,
        title: this.subTitle,
        options: this.subOptions,
        gradeTerm: this.grade,
        typeId: this.selectValue,
        answer: this.answer,
        parentId: this.myTypeQuestionId,
        paperId: this.paperId,
        // 知识点id
        chapter: this.knowledge,
        // 单题的id（用于修改）
        id: this.singleEditQuestionId,
        // 同意答案
        OtherAnswer: this.allSynonymAnswer
        // VoiceTitle: this.subVoiceTitle
      }).then(res => {
        console.log('保存---', res)
        if (res.Success) {
          this.$message.success('保存成功')
          this.getPaperInfo()
          this.MakeTopic = false
        } else {
          this.$message.success(res.Msg)
        }
      })
    },
    // 移除题目
    removeSub (Id) {
      this.$http.post('/Paper/Exam_PaperItemMapping/Remove', { id: Id }).then(res => {
        console.log('移除题目', res)
        this.getPaperInfo()
      })
    },
    // 保存时答案格式编写
    saveAnswerWrite () {
      if (this.selectValue === 5 || this.selectValue === 23) {
        // const blanks = this.completion.substring(heade + 1, end).split('').join('|')
        // const blanks = this.completion.map(item => {
        //   return item.replace(/<[^<>]+>/g, '')
        // })
        // const subBlanks = blanks.join('|')
        // console.log(subBlanks)
        // this.answer = subBlanks
      }
      console.log('填空修改答案---', this.answer)
      // debugger
      if (this.answer.length > 1 && this.selectValue !== 5 && this.selectValue === 10) {
        if (this.answer === '' || typeof this.answer === 'string') {
          return
        } else {
          this.answer = this.answer.join('|')
        }
      }
      if (this.selectValue === 11) {
        this.subOptions = this.subJudgeOptions
      }
      console.log('修改的答案--', this.answer)
    },
    // 编辑题目
    editSubject () {
      this.MakeTopic = true
    },
    // 关闭编辑试题
    onCloseMakeTopic () {
      this.MakeTopic = false
    },
    // 取消题目编辑
    cancelEdit () {
      this.EditCance = true
      // this.MakeTopic = false
    },
    // 取消弹框题目编辑
    handleCancelEdit () {
      this.EditCance = false
      setTimeout(() => { this.MakeTopic = false }, 300)
    },
    // 确定题目编辑
    handleOkEdit () {
      if (this.subTitle === '' || this.subAnalysis === '' || this.answer === '') {
        this.$message.error('请填写完整内容')
        this.EditCance = false
      } else {
        this.handleSave()
        this.EditCance = false
        setTimeout(() => { this.MakeTopic = false }, 300)
      }
    },
    // 校验保存规则
    checkSaveRule () {
      this.selectCheck = true
      if (this.subTitle === '') {
        this.titleCheck = true
      } else {
        this.titleCheck = false
      }
      if (this.subAnalysis === '') {
        this.analysisCheck = true
      } else {
        this.analysisCheck = false
      }
    },
    // 单个题目编辑
    singleEditSubject (id, TypeId, itemType, parentId, IsQuote) {
      debugger
      // 回显单个题目
      this.$nextTick(() => {
        this.handleEchoSubject(id)
      })
      this.MakeTopic = true
      this.questionId = TypeId
      console.log('当前题目id--', id)
      this.singleEditQuestionId = id
      // parentId
      this.myTypeQuestionId = parentId
      // itemType
      this.questionItemType = itemType
      // itemtype保存
      // 默认类型
      this.selectValue = +TypeId
      // 是否是引用的题目
      this.IsQuote = IsQuote
      console.log('是否是引用的题目--', this.IsQuote)

      // this.afterVisibleChange(id)
    },
    // 回显单个题目
    handleEchoSubject (id) {
      // debugger
      this.$http.post('/Paper/Exam_Item/GetTheData', { id }).then(res => {
        console.log('题干---', res)
        // this.subTitle = ''
        const reg = /(#&\d+@)/g
        const stem = res.Data.Title.replace(reg, ' ')
        this.subTitle = stem
        // this.subVoiceTitle = res.Data.VoiceTitle
        console.log('题干---', this.subTitle)
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetAnalysisDataByItemId', { itemId: id }).then(res => {
        console.log('解析---', res)
        // this.subAnalysis = ''
        this.subAnalysis = res.Data.Content
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetAnswerDataByItemId', { itemId: id }).then(res => {
        console.log('答案---', res)
        // this.answer = ''
        // this.alonevalue = ''
        // this.judge = ''
        this.answer = res.Data.Content
        this.alonevalue = res.Data.Content
        this.judge = res.Data.Content
        // debugger
        if (res.Data.Content.length > 1) {
          if (res.Data.Content.indexOf(',') !== -1) {
            this.MultipleChoice = res.Data.Content.split(',')
          } else {
            this.MultipleChoice = res.Data.Content.split('|')
          }
        }
        console.log('多选答案---', this.MultipleChoice)
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetOptionsByItemId', { itemId: id }).then(res => {
        console.log('选项---', res)
        this.subOptions = ''
        this.subOptions = res.Data
      })
      this.$http.post('/Paper/Exam_Item/GetMoreItemAnswer', { itemId: id }).then(res => {
        console.log('同义答案---', res)
        // this.subOptions = res.Data
        // this.EchoSynonymousAnswer = ''
        this.EchoSynonymousAnswer = res.Data
      })
    },
    // 单选题增加
    AddOptions () {
      switch (this.subOptions.length) {
        case 0:
          this.subOptions.push({
            Option: 'A',
            Content: ''
          })
          break
        case 1:
          this.subOptions.push({
            Option: 'B',
            Content: ''
          })
          break
        case 2:
          this.subOptions.push({
            Option: 'C',
            Content: ''
          })
          break
        case 3:
          this.subOptions.push({
            Option: 'D',
            Content: ''
          })
          break
        case 4:
          this.subOptions.push({
            Option: 'E',
            Content: ''
          })
          break
        case 5:
          this.subOptions.push({
            Option: 'F',
            Content: ''
          })
          break
        case 6:
          this.subOptions.push({
            Option: 'G',
            Content: ''
          })
          break
        default:
          this.$message.success('最多加到G哦')
      }
      // 填空增加
      this.completion.push('')
    },
    // 减少选项
    ReduceOptions () {
      // 选择减少
      this.subOptions.pop()
      // 填空减少
      this.completion.pop('')
    },
    // 去布置
    toAssign () {
      this.$router.push({ path: '/Teacher/My_Task/HighWrong/AssignWrong',
        query: {
          paperId: this.paperId
        } })
    }
  }
}
</script>

<style lang="less" scoped>
  .fn-col {
    font-size: 18px;
    color: #000;
  }
  // 选中题号颜色
  .sel-col {
    font-weight: 700;
    color: #68BB97;
  }
  // 题号
  .process{
    position: relative;
    width: 120px;
    // height: 110px;
    display: inline-block;
    // margin: 10px 0px;
}
  .number{
    width: 30%;
    height: 30%;
    text-align: center;
    margin: 0 auto;
}
.high-wrong {
  margin-top: 20px;
  margin-bottom: 16px;
  padding: 8px 10px;
  background-color: #fff;
  border-radius: 5px;
  .high-p {
    font-size: 15px;
    margin-bottom: 15px;
  }
}
  // 选入题目
  .selected-sub {
    padding: 5px 10px;
    background-color: #fff;
    border-radius: 5px;
    .selec-p {
      font-size: 15px;
      margin-bottom: 15px;
      .sel-input {
        border: 1px solid #ccc;
        border-radius: 15px;
      }
    }
    .edit-title {
      margin-top: 2px;
      margin-left: 15px;
      padding: 3px 10px;
      font-size: 15px;
      color: #fff;
      background-color: #68BB97;
      border-radius: 5px;
    }
    .selec-p2 {
      margin-bottom: 8px;
      color:#68BB97;
    }
    .selec-type {
      display: flex;
      margin-bottom: 20px;
    }
    .type-flex {
      // flex: 1;
      width: 30%;
      height: 46px;
      line-height: 46px;
      margin-right: 20px;
      padding: 0 16px;
      background-color: #F4F4F4;
      border-radius: 5px;
    }
    // 去布置
    .to-assign {
      width: 110px;
      height: 46px;
      line-height: 46px;
      text-align: center;
      color: #fff;
      background-color: #68BB97;
      border-radius: 8px;
    }
  }
  // 试卷内容
  .sub-infor {
    // margin-top: 30px;
    margin-bottom: 30px;
    padding: 16px 0 16px 16px;
    font-size: 15px;
    background-color: #fff;
    border-radius: 5px;
    p {
      margin-bottom: 15px;
    }
    // 选择题
    .choice-info {
      span {
        margin-right: 60px;
      }
    }
    // 解析
    .ana-info {
      padding-bottom: 20px;
      border-bottom: 1px solid #DADADA;
    }
    // 操作按钮
    .hanle-btn {
      span {
        width: 74px;
        height: 32px;
        line-height: 32px;
        margin-right: 20px;
        text-align: center;
      }
      span:nth-child(1) {
        color: #FF682C;
        border: 1px solid #FF682C;
        border-radius: 5px;
      }
      span:nth-child(2) {
        color: #fff;
        background-color: #A8D261;
        border-radius: 5px;
      }
      span:nth-child(3) {
        border: 1px solid #ccc;
        border-radius: 5px;
      }
    }
  }
  // 编辑弹窗
  .make-topic {
    // display: flex;
    padding: 20px;

    .make-title {
      // width: 49%;
      height: 326px;
      margin-right: 10px;
      padding: 20px;
      background-color: #fff;
      border-radius: 5px;
      .make-sub-title {
        margin-right: 30px;
      }
    }
    // 选择
    .make-select {
      width: 100%;
      min-height: 326px;
      padding: 20px;
      background-color: #fff;
      border-radius: 5px;
      /deep/.w-e-text img {
          height: 30px;
      }
      // 同意答案的增加
      .sy-blocks {
        font-size: 18px;
      }
      // 增加填空
      .fill-blocks {
        font-size: 18px;
      }
      // 添加同意按钮
      .agreen-answer {
        cursor: pointer;
        margin-top: 10px;
        padding: 12px;
        background-color: #68BB9A;
        border-radius: 5px;
      }
    }
    // 解析
    .make-analysis {
      width: 100%;
      height: 230px;
      padding: 20px;
      margin-top: 30px;
      background-color: #fff;
      /deep/.ant-editor-wang {
        height: 90%;
      }
    }
  }
  // 题目操作按钮
  .edit-subject-operation {
    margin-right: 50px;
    span {
      display: inline-block;
      width: 114px;
      height: 60px;
      line-height: 60px;
      text-align: center;
      margin-left: 48px;
      margin-top: 40px;
      border-radius: 5px;
      cursor: pointer;
    }
    span:nth-child(1) {
      background-color: #fff;
    }
    span:nth-child(2) {
      color: #fff;
      background-color: #68BB97;
    }
  }
  /deep/.ant-progress-text{
    font-size: 16px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #fd7b47;
}
</style>
