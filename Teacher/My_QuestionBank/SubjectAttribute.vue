<template>
  <div>
    <a-form :form="form" :label-col="{ span: 5 }" :wrapper-col="{ span: 8 }" @submit="handleSubmit">
      <a-row :gutter="12">
        <a-col :span="10">
          <a-form-item label="所属年级">
            <!-- 默认选项  initialValue: '5五年级' -->
            <!-- /ClassManage/Exam_Class/GetGradeByUserId -->
            <a-select
              v-decorator="[
                'grade',
                { rules: [{ required: true, message: '必选' }],initialValue: gradeInfo },
              ]"
              placeholder="年级选择"
              @change="handleSelectChangeGrade"
            >
              <a-select-option v-for="item in gradeInfor" :key="item.Id" :value="item.Id + '' + item.Gradename">
                {{ item.Gradename }}
              </a-select-option>
            </a-select>
          </a-form-item>
          <a-form-item class="level" label="题型">
            <a-select
              @change="SubjectChange"
              style="height: 32px"
              v-decorator="['subType', { rules: [{ required: true, message: '必选' }] }]"
              placeholder="题型选择">
              <a-select-option v-for="item in subjectType" :key="item.Id" :value="item.Id + ',' + item.ItemType " >
                {{ item.ItemTypeName }}
              </a-select-option>
            </a-select>
          </a-form-item>
        </a-col>
        <a-col :span="10">
          <a-form-item label="章节">
            <a-cascader
              :field-names="{ label: 'ChapterName', value: 'ChapterId', children: 'Firsts' }"
              :options="options"
              change-on-select
              @change="onChangeChapter"
              placeholder="章节选择"
              v-decorator="['chapter', { rules: [{ required: true, message: '必选' }]}]"/>
            <!-- <a-tree-select
              allowClear
              showSearch

              :tree-data="options"
              treeNodeFilterProp="title"
              @change="onChange"
              placeholder="章节选择"
            ></a-tree-select> -->
          </a-form-item>
          <a-form-item label="难度" class="level-star">
            <a-rate @change="starChange" style="height: 32px" v-decorator="['Level', { rules: [{ required: true, message: '必选' }] }]"></a-rate>
          </a-form-item>
        </a-col>

      </a-row>
    </a-form>
    <div @click="handleSubmit" class="start-input">开始录入</div>
    <footerLogo></footerLogo>
    <a-drawer
      title=""
      placement="top"
      :visible="MakeTopic"
      @close="onCloseMakeTopic"
      height="100%"
      :drawerStyle="{ backgroundColor: '#FAF7F8' }"
      :destroyOnClose="true"
    >
      <div class="title-attribute" style="padding-left: 30px">
        <p>题目属性</p>
        <p>年级: <span>{{ gradeNameAttribute }}</span></p>
        <p>知识点: <span>{{ KnowledgePoints }} </span></p>
        <p>题型: <span>{{ QuestionType }}</span></p>
        <p>难度: <span>
          <a-rate v-model="valueStar" disabled></a-rate>
        </span></p>
      </div>
      <div class="make-topic clearfix">
        <div class="make-title fl">
          <p class="f-c"><span>题干</span></p>
          <span style="color: #FF8D60;">
            <span v-if="titleCheck">
              <img src="@/assets/teacher/提示编组.png" alt="">&nbsp;
              <span style="vertical-align: middle;">题干不能为空</span>
            </span>
          </span>
          <wang-edit @keyup.native="keyUp" style="height: 90%" ref="edit" v-model="subTitle"></wang-edit>
          <!-- <wang-edit v-if="aa === '2'" style="height: 90%" ref="edit" v-model="subjectListTitle"></wang-edit> -->
        </div>
        <!-- 选择题 -->
        <div v-if="questionId === '2,1' || questionId === '10,2' || questionId === '11,3'" class="make-select fl">
          <p class="f-c">答案选项</p>
          <span v-if="questionId === 2">单选</span>
          <span v-if="questionId === 10">多选</span>
          <span v-if="questionId === 11">判断题</span>
          <span v-if="questionId === 5">填空题</span>
          <!-- <a-radio-group @change="onChangeSlect" v-model="selectValue">
                      <a-radio :value="2">单选</a-radio>
                      <a-radio :value="10">多选</a-radio>
                      <a-radio :value="11">判断题</a-radio>
                    </a-radio-group> -->
          <div style="margin-top:50px">
            <p v-for="subOption in subOptions" :key="subOption.id" v-show="questionId === '2,1'">
              <span class="options" >
                <!-- v-model="alonevalue" -->
                <a-radio-group @change="onChangeSlect" >
                  <a-radio :value="subOption.Option">{{ subOption.Option }} 选项</a-radio>
                </a-radio-group>
                <span v-if="subOption.Content === '' && selectCheck" ><img src="@/assets/teacher/警告.png" alt=""><span style="vertical-align: middle;">选项或答案不能为空</span></span>
                <!-- {{ subOption.Option }}选项 -->
              </span>
              <!-- <span v-if="subOption.Content !== ''" style="color:#FF682C">选项不能为空</span> -->
              <wang-edit v-model="subOption.Content"></wang-edit>
            </p>
            <p v-show="questionId === '10,2'">
              <span class="options" >
                <a-checkbox-group @change="onChangeduo" >
                  <a-row>
                    <a-col v-for="subOption in subOptions" :key="subOption.id" :span="24">
                      <a-checkbox :value=" subOption.Option">
                        {{ subOption.Option }}选项
                      </a-checkbox>
                      <span v-if="subOption.Content === '' && selectCheck"><img src="@/assets/teacher/提示编组.png" alt=""><span style="vertical-align: middle;">选项或答案不能为空</span></span>
                      <wang-edit v-model="subOption.Content"></wang-edit>
                    </a-col>
                  </a-row>
                </a-checkbox-group>

                <!-- {{ subOption.Option }}选项 -->
              </span>

            </p>
            <p v-for="subOption in subJudgeOptions" :key="subOption.id" v-show="questionId === '11,3'">
              <span class="options" >
                <!-- v-model="judge" -->
                <a-radio-group @change="onChangeSlect" >
                  <a-radio :value="subOption.Option">{{ subOption.Option }} {{ subOption.Content }}</a-radio>
                </a-radio-group>
                <!-- {{ subOption.Option }}选项 -->
              </span>
              <!-- <wang-edit v-model="subOption.Content"></wang-edit> -->
            </p>
          </div>
          <!-- <p>答案
                      <wang-edit v-model="answer"></wang-edit>
                    </p> -->
          <p v-if="questionId === '10,2' || questionId === '2,1'" style="float: right;font-size: 18px;margin-bottom: 0;">
            <span class="f-c">选项</span>
            <a-icon type="plus-circle" @click="AddOptions"/>&nbsp;&nbsp;&nbsp;&nbsp;
            <a-icon type="minus-circle" @click="ReduceOptions"/>
          </p>
        </div>
        <!-- 填空题 -->
        <div v-if="questionId === '5,4'" class="make-select fl">
          <p class="f-c">答案选项</p>
          <div v-for="( i, j) in 1" :key="j" class="clearfix">
            <!-- 空{{ j + 1 }}   {{ index }} -->

            <span class="blanks-infor" v-for="(item,index) in blanksUp" :key="item"> 空{{ index + 1 }} <p>{{ item }}</p></span>
            <span class="fg agreen-answer" @click="AddSynonym(j)" style="marginTop: 15px">添加同义答案</span>
            <!-- {{ synonym }} -->
            <!-- 同义答案 -->
            <div v-for="(s, t) in counter" :key="t" v-show="synonym && num === j">
              <SynonymAnswer :sd="num" @get="get"></SynonymAnswer>
              <!-- v-for="(Syn, k) in EchoSynonymousAnswer" :key="k" -->
            </div>
            <span class="sy-blocks" v-show="synonym && num === j">
              <a-icon type="plus-circle" @click="syAdd"/>&nbsp;&nbsp;&nbsp;&nbsp;
              <a-icon type="minus-circle" @click="syDel"/>
            </span>
          </div>
          <div class="echo-synonymous-answer">
            <p>已经添加的同义答案</p>
            <!-- <p class="echo-p" v-for="(Syn, k) in EchoSynonymousAnswer" :key="k">
              <span>{{ Syn }}</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
              <span @click="handleEchoSynonymousAnswer(Syn)" class="cur">删除</span>
            </p> -->
          </div>

        </div>

        <div class="make-analysis fl">
          <p class="f-c">解析</p>
          <span v-if="analysisCheck">
            <img src="@/assets/teacher/提示编组.png" alt="">&nbsp;
            <span style="vertical-align: middle;">解析不能为空</span>
          </span>
          <wang-edit v-model="subAnalysis"></wang-edit>
        </div>
      </div>
      <div class="edit-subject-operation">
        <span @click="cancelEdit">取消</span>
        <a-modal
          v-model="EditCance"
          title="温馨提示"
          width="630px"
          @ok="handleOkEdit"
          @cancel="handleCancelEdit"
          centered
        >
          <p>关闭后，正在编辑的题目将不做任何变更！是否保存</p>
        </a-modal>
        <!-- @click="handleSave" -->
        <span @click="handleSave">完成</span>
      </div>
    </a-drawer>
  </div>
</template>

<script>
import WangEdit from '@/components/WangEditor/WangEditor'
import SynonymAnswer from '@/components/AddSynonym/AddSynonym'
import footerLogo from '@/components/XuHuiFooter/FooterLogo'
export default {
  components: {
    WangEdit,
    SynonymAnswer,
    footerLogo
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    // 获取年级
    this.getGrade()
    // 获取题型
    this.getSubjectType()
    // 章节
    this.getGradeKnowledge()
    // 获取默认年级
    this.getDefaultGrade()
  },
  data () {
    return {
      // 题干校验
      titleCheck: false,
      // 选项校验
      selectCheck: false,
      // 解析校验
      analysisCheck: false,
      // 是否保存
      isSave: 0,
      // 题目属性
      gradeNameAttribute: '',
      KnowledgePoints: '',
      QuestionType: '',
      difficulty: '',
      userId: '',
      // m默认年级
      gradeInfo: '',
      gradeInfor: [],
      // 题型
      subjectType: [],
      // 章节
      options: [],
      form: this.$form.createForm(this),
      MakeTopic: false,
      // 编辑题干
      subTitle: '',
      // 题型ID
      questionId: '',
      // 编辑解析
      subAnalysis: '',
      // 取消编辑弹窗
      EditCance: false,
      // 默认选择空
      subOptions: [ { Option: 'A', Content: '' }, { Option: 'B', Content: '' }, { Option: 'C', Content: '' } ],
      subJudgeOptions: [{ Option: 'A', Content: '对' }, { Option: 'B', Content: '错' }],
      // 默认填空空
      completion: ['', '', ''],
      // 添加同意答案数量
      counter: [{}],
      // 同义答案的显示隐藏
      synonym: false,
      blanksUp: [],
      num: '',
      allSynonymAnswer: [],
      valueStar: 0,
      // 题型
      selectValue: '',
      // 年级
      grade: '',
      // 难度选择
      levelSelect: '',
      // 章节选择
      chapterSelect: '',
      // 题型
      questionItemType: ''
    }
  },
  methods: {
    // handleSubmit (e) {
    //   e.preventDefault()
    //   this.form.validateFields((err, values) => {
    //     if (!err) {
    //     }
    //   })
    // },
    // 获取默认年级
    getDefaultGrade () {
      this.$uwonhttp.post('/ManagerPersonalsController/ManagerPersonals/GetTeacherInfo', { userId: this.userId }).then(res => {
        console.log(' 获取默认年级', res)
        this.gradeInfo = res.data.Data.Grade
        this.grade = res.data.Data.Grade[-1, 0]
        console.log('默认年级======', this.grade)
      })
    },
    keyUp () {
      // console.log(this.subTitle)
      const re = /（(.*?)）/g
      let temp = []
      const a = []
      setTimeout(() => {
        while (temp = re.exec(this.subTitle)) {
          // debugger
          // console.log(temp[1])
          // debugger
          a.push(temp[1])
          // console.log(a)
        }
      }, 300)
      this.blanksUp = a
      // console.log(this.blanksUp)
      // console.log(this.subTitle)
      // const regex = /\（.+?\）/g
      // console.log(this.subTitle.match(regex))
      // /（(.*)）/g.exec(this.subTitle)
    },
    // 开始录入
    handleSubmit (e) {
      e.preventDefault()
      this.subTitle = ''
      this.subAnalysis = ''
      // debugger
      this.answer = ''
      this.titleCheck = false
      this.analysisCheck = false
      // 选项的校验
      this.selectCheck = false
      // this.subOptions.Content = ''
      this.subOptions = [ { Option: 'A', Content: '' }, { Option: 'B', Content: '' }, { Option: 'C', Content: '' } ]
      this.subJudgeOptions = [{ Option: 'A', Content: '对' }, { Option: 'B', Content: '错' }]
      this.MultipleChoice = []
      this.alonevalue = ''
      // 判断题的选择
      this.judge = ''
      this.blanksUp = []
      this.form.validateFields((err, values) => {
        if (!err) {
          console.log('Received values of form: ', values)
          this.MakeTopic = true
          this.questionId = values.subType
          this.gradeNameAttribute = values.grade.slice(1)
          console.log(this.gradeNameAttribute)
          this.valueStar = values.Level
          if (values.subType === '2,1') {
            this.QuestionType = '单项选择题'
          } else if (values.subType === '10,2') {
            this.QuestionType = '多项选择题'
          } else if (values.subType === '5,4') {
            this.QuestionType = '填空题'
          } else if (values.subType === '11,3') {
            this.QuestionType = '判断题'
          }
        }
      })
    },
    onCloseMakeTopic () {
      this.MakeTopic = false
    },
    // 取消题目编辑
    cancelEdit () {
      this.EditCance = true
      // this.MakeTopic = false
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
    // 取消弹框题目编辑
    handleCancelEdit () {
      this.EditCance = false
      setTimeout(() => { this.MakeTopic = false }, 300)
    },
    onChangeduo (checkedValues) {
      console.log('checked = ', checkedValues)
      this.answer = checkedValues
    },
    onChangeSlect (e) {
      console.log(e.target.value)
      this.answer = e.target.value
      console.log('答案---', this.answer)
    },
    // 单选题增加
    AddOptions () {
      switch (this.subOptions.length) {
        case 0:
          this.subOptions.push({
            Option: 'A'
          })
          break
        case 1:
          this.subOptions.push({
            Option: 'B'
          })
          break
        case 2:
          this.subOptions.push({
            Option: 'C'
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
            Option: 'E'
          })
          break
        case 5:
          this.subOptions.push({
            Option: 'F'
          })
          break
        case 6:
          this.subOptions.push({
            Option: 'G'
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
    // 添加同义答案按钮
    AddSynonym (index) {
      this.num = index
      this.synonym = true
      this.counter = [{}]
      console.log(this.num)
      if (this.synonymousAnswer !== '') {
        this.allSynonymAnswer.push({
          index: this.synonymousAnswers
        })
        console.log('所有同义答案的值--', this.allSynonymAnswer)
      }
      console.log('修改后的所有空值', this.mulAnswer)
    },
    get (data, sd) {
      console.log(data, sd)
      const obj = {}
      obj['Content'] = data
      this.allSynonymAnswer.push(obj)
      console.log('对应填空题的同义答案---', this.allSynonymAnswer)
    },
    // 增加同义答案的输入框
    syAdd () {
      this.counter.push({})
    },
    // 减少同义答案的输入框
    syDel () {
      this.counter.pop()
      this.allSynonymAnswer.pop()
      console.log('对应填空题的同义答案---', this.allSynonymAnswer)
      if (this.counter.length === 0) {
        this.counter = [{}]
      }
    },
    // 录入题目保存编辑题目
    handleSave () {
      console.log('题干---', this.subTitle)
      console.log('选项类型---', this.selectValue)
      console.log('解析---', this.subAnalysis)
      console.log('选项---', this.subOptions)
      console.log('填空---', this.completion)
      console.log('答案---', this.answer)
      console.log('题型ItemId---', this.questionItemType)
      // console.log('自身类型id---', this.myTypeQuestionId)
      // console.log('编辑单个题目id--', this.singleEditQuestionId)
      console.log('同意答案---', this.allSynonymAnswer)
      console.log('年级---', this.grade)
      console.log('章节---', this.chapterSelect)
      console.log('难度---', this.levelSelect)
      // 答案的格式编写
      this.saveAnswerWrite()
      // 保存提示
      this.checkSaveRule()
      console.log('是否进行保存---', this.isSave)
      if (this.isSave === 0) {
        return false
      } else {
        this.$http.post('/Paper/Exam_Item/SaveItemInfo', {
          analysis: this.subAnalysis,
          itemType: this.questionItemType,
          title: this.subTitle,
          options: this.subOptions,
          gradeTerm: this.grade,
          typeId: this.selectValue,
          answer: this.answer,
          // parentId: this.myTypeQuestionId,
          // paperId: this.paperId,
          // 知识点id
          chapter: this.chapterSelect,
          // 单题的id（用于修改）
          // id: this.singleEditQuestionId,
          // 同意答案
          OtherAnswer: this.allSynonymAnswer,
          level: this.levelSelect
        }).then(res => {
          console.log('保存录入题目---', res)
          if (res.Success) {
            this.$message.success('保存成功')
            // this.getMySubjectBank()
            this.MakeTopic = false
          }
        })
      }
    },
    // 校验保存规则
    checkSaveRule () {
      console.log(this.selectValue)
      debugger
      const aaa = this.subOptions.some(item => {
        return item.Content === ''
      })
      this.selectCheck = true
      if (this.subTitle === '') {
        this.titleCheck = true
        this.isSave = 0
      } else if (this.subTitle !== '') {
        this.titleCheck = false
      }
      if (this.selectValue === 5) {
        this.isSave = 1
      } else {
        // if (this.subTitle)
        if (aaa) {
          this.isSave = 0
        } else if (!aaa) {
          this.isSave = 1

          if (this.subAnalysis === '') {
            this.analysisCheck = true
            this.isSave = 0
          } else if (this.subAnalysis !== '') {
            this.analysisCheck = false
            this.isSave = 1
          }
        }
      }
      // if (!this.answer) {
      //   this.isSave = 0
      //   this.$message.error('注意答案的选择')
      // }
      // if (this.aloneLevel === 0) {
      //   this.isSave = 0
      //   this.selectCheckLevel = true
      // } else {
      //   this.isSave = 1
      //   this.selectCheckLevel = false
      // }
      // debugger
    },
    // 保存时答案格式编写
    saveAnswerWrite () {
      if (this.selectValue === 5) {
        // const blanks = this.completion.substring(heade + 1, end).split('').join('|')
        const blanks = this.completion.map(item => {
          return item.replace(/<[^<>]+>/g, '')
        })
        const subBlanks = blanks.join('|')
        console.log(subBlanks)
        this.answer = subBlanks
      }
      console.log('填空修改答案---', this.answer)
      // debugger
      console.log(typeof (this.answer))
      if (this.answer === 11) {
        if (this.answer === '') {
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
    // 获取年级
    getGrade () {
      this.$uwonhttp.get('/Class/TeacherClassManager/GetGradeList').then(res => {
        console.log('获取年级---', res)
        this.gradeInfor = res.data.Data
      })
    },
    // 获取题型
    getSubjectType () {
      this.$http.post('/Base_Manage/Exam_ItemType/GetDataList', { PageIndex: '1', PageRows: '50' }).then(res => {
        console.log('题型---', res)
        this.subjectType = res.Data
      })
    },
    // 章节
    getGradeKnowledge (id) {
      this.$http.post('/Paper/Exam_SubjectChapter/GetChapterTreeByUserId', { userId: this.userId }).then(res => {
        console.log('年级对应知识点---', res)
        this.options = ''
        this.options = res.Data
        console.log(this.options)
      })
    },
    // 章节
    onChangeChapter (value, label) {
      console.log('章节的选择---', label)
      this.chapterSelect = value[1]
      console.log('章节的选择---', this.chapterSelect)
      if (label.length > 2) {
        console.log('章节---', label[2].ChapterName)
        this.KnowledgePoints = label[2].ChapterName
      }
      // this.KnowledgePoints = label[1].ChapterName
    },
    // 难度选择
    starChange (value) {
      console.log(value)
      this.levelSelect = value
    },
    // 年级选择
    handleSelectChangeGrade (value) {
      console.log(value)
      console.log(value[-1, 0])
      this.grade = value[-1, 0]
      console.log(this.grade)
    },
    // 题型选择
    SubjectChange (value) {
      const item = value.split(',')
      console.log(item)
      this.selectValue = +item[0]
      this.questionItemType = +item[1]
      console.log('TypeId---', this.selectValue)
      console.log('ItemType---', this.questionItemType)
    }
  }
}
</script>

<style lang="less" scoped>
.level-star {
  /deep/.ant-form-item-control {
    height: 52px;
    // text-indent: 5px;
    padding-left: 6px;
    border: 1px solid #ccc;
    border-radius: 5px;
    line-height: 42px;
    background-color: #fff;
  }
}
/deep/.ant-select-selection-selected-value {
    display: block;
    opacity: 1;
    width: 349px;
    line-height: 52px;
    height: 52px;
    font-size: 16px;
}
/deep/.ant-select-selection__rendered {
  width: 349px;
    height: 52px;
}
/deep/.ant-select-selection--single {
  width: 349px;
  height: 52px;
}
.ant-form-item-control .has-success {
  width: 349px;
  height: 52px;
}
/deep/.ant-form-item-control {
  width: 349px;
  height: 52px;
}
/deep/.ant-cascader-input.ant-input {
  height: 52px;
}
 .blanks-infor {
    display: inline-block;
    text-align: center;
    margin-right: 15px;
    p {
      width: 60px;
      height: 50px;
      line-height: 50px;
      font-size: 16px;
      background-color: #ccc;
      border-radius: 5px;
      border: 1px solid #000;
      color: #fff;
    }
  }
  //题目属性
  .title-attribute {
    span {
      margin-left: 15px;
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
      background-color: #68BB97;
    }
  }
 // 制作题目
  /deep/.editor-wrapper {
    height: 90%;
  }
   /deep/.w-e-text-container {
      height: 89% !important;
      border: none;
    }
    /deep/.w-e-text {
      overflow-y: auto;
    }
    // 模态框
  .ant-modal-title {
    color: #fff;
  }
  /deep/.ant-modal-header {
    text-align: center;
    color: #fff;
    background-color: #68BB97;
  }
  /deep/.ant-modal-footer button + button {
    background-color: #68BB97;
    border: none;
  }
.make-topic {
    // display: flex;
    padding: 20px;

    .make-title {
      width: 49%;
      height: 615px;
      margin-right: 10px;
      padding: 20px;
      background-color: #fff;
      border-radius: 5px;
    }
    // 选择
    .make-select {
      width: 49%;
      min-height: 615px;
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
      width: 49%;
      height: 258px;
      padding: 20px;
      margin-top: 30px;
      background-color: #fff;
      /deep/.ant-editor-wang {
        height: 90%;
      }
    }
  }
  .ant-form-item-control {
    background-color: #fff;
  }
  .start-input {
    position: absolute;
    right: 228px;
    width: 114px;
    height: 60px;
    line-height: 60px;
    text-align: center;
    color: #fff;
    background-color: #68BB97;
    border-radius: 5px;
    cursor: pointer;
  }
  // 多选框
  .ant-checkbox-group {
    width: 100%;
  }
</style>
