<template>
  <div id="question-bank">
    <div class="clearfix" style="margin-bottom: 14px;">
      <span class="fl" style="font-size: 18px;color: #000;margin-top:17px">题目属性</span>
      <!-- <p class="fg input-title"><span @click="InputSubject">录入题目</span></p> -->
      <span class="fg input-title" @click="InputSubject">录入题目</span>
    </div>
    <!-- 题库信息 -->
    <div class="question-bank">
      <p>
        知识点:&nbsp;&nbsp;&nbsp;
        <a-cascader
          :field-names="{ label: 'ChapterName', value: 'ChapterId', children: 'Firsts' }"
          style="width: 349px"
          :options="options"
          change-on-select
          allowClear
          @change="onChangeKnowledge"
          placeholder="全部题目"/>
      </p>
      <p class="question-type">题型:
        <span :class="{ 'col-mytest': itemType == 0 }" @click="allItemType(0)">全部</span>
        <span :class="{ 'col-mytest': itemType == item.Id }" @click="SubjectSelect(item.Id)" v-for="item in subjectList" :key="item.Id">{{ item.ItemTypeName }}</span>
      </p>
      <ul class="difficulty">
        <li>难度:</li>
        <li :class="{ 'yellow': yel === 0 }" @click="Difficulty(0)">全部</li>
        <li :class="{ 'yellow': yel === 1 }" @click="Difficulty(1)">☆</li>
        <li :class="{ 'yellow': yel === 2 }" @click="Difficulty(2)">☆☆</li>
        <li :class="{ 'yellow': yel === 3 }" @click="Difficulty(3)">☆☆☆</li>
        <li :class="{ 'yellow': yel === 4 }" @click="Difficulty(4)">☆☆☆☆</li>
        <li :class="{ 'yellow': yel === 5 }" @click="Difficulty(5)">☆☆☆☆☆</li>
      </ul>
    </div>
    <div class="clearfix">
      <p class="fl"><span>默认</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span>使用次数&nbsp;&nbsp;<span style="fontSize:">↓</span></span></p>
    </div>
    <!-- 题库题目 -->
    <div
      v-for="(ite, inde) in SubjectBank"
      :key="inde"
      @click="IndividualSubject(ite.ItemId)"
      :class="{ 'qus-type': IndividualSettings === ite.ItemId }"
      class="clearfix subject-tip"
      style="">
      <p class="fl">{{ inde + 1 }}.</p>
      <div v-html="ite.Title" style="padding: 0 20px;cursor:pointer;">
      </div>
      <!-- 带有选项类的题 -->
      <div class="paper-title" v-show="ite.ItemTypeId !== 5">
        <P v-for="(t,n) in ite.AuditPaperItemAnswers" :key="n" v-show="t.Type === 1">
          <span>{{ t.Option }}</span> : <span style="display: inline-block;" v-html="t.Content"></span>
        </P>
        <!-- <P>B : 2</P>
        <P>C : 3</P> -->
      </div>
      <div v-show="IndividualSettings === ite.ItemId" :class="{ 'bor': IndividualSettings === ite.ItemId }">
        <p class="clearfix" style="margin-top: 15px;">
          <span class="fg z-span" @click="deleteAlone(ite.Id)">删除</span>
          <span class="fg z-span" @click="singleEditSubject(ite.ItemId,ite.TypeId,ite.ItemType,ite.GradeId)">编辑</span>
          <a-drawer
            title=""
            placement="top"
            :visible="MakeTopic"
            @close="onCloseMakeTopic"
            height="100%"
            :drawerStyle="{ backgroundColor: '#FAF7F8' }"
            :destroyOnClose="true"
          >
            <div class="make-topic clearfix">
              <div class="make-title fl">
                <p class="f-c"><span>题干</span></p>
                <wang-edit @keyup.native="keyUp" style="height: 90%" ref="edit" v-model="subTitle"></wang-edit>
                <!-- <wang-edit v-if="aa === '2'" style="height: 90%" ref="edit" v-model="subjectListTitle"></wang-edit> -->
              </div>
              <!-- 选择题 -->
              <div v-if="questionId === 2 || questionId === 10 || questionId === 11" class="make-select fl">
                <p class="f-c">答案选项</p>
                <span v-if="questionId === 2">单选</span>
                <span v-if="questionId === 10">多选</span>
                <span v-if="questionId === 11">判断题</span>
                <!-- <a-radio-group @change="onChangeSlect" v-model="selectValue">
                      <a-radio :value="2">单选</a-radio>
                      <a-radio :value="10">多选</a-radio>
                      <a-radio :value="11">判断题</a-radio>
                    </a-radio-group> -->
                <div style="margin-top:50px">
                  <p v-for="subOption in subOptions" :key="subOption.id" v-show="questionId === 2">
                    <span class="options" >
                      <a-radio-group @change="onChangeSlect" v-model="alonevalue">
                        <a-radio :value="subOption.Option">{{ subOption.Option }} 选项</a-radio>
                      </a-radio-group>
                      <!-- {{ subOption.Option }}选项 -->
                    </span>
                    <!-- <span v-if="subOption.Content !== ''" style="color:#FF682C">选项不能为空</span> -->
                    <wang-edit v-model="subOption.Content"></wang-edit>
                  </p>
                  <p v-show="questionId === 10">
                    <span class="options" >
                      <a-checkbox-group @change="onChangeduo" :value="MultipleChoice">
                        <a-row>
                          <a-col v-for="subOption in subOptions" :key="subOption.id" :span="24">
                            <a-checkbox :value="subOption.Option">
                              {{ subOption.Option }}选项
                            </a-checkbox>
                            <wang-edit v-model="subOption.Content"></wang-edit>
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
                    <!-- <wang-edit v-model="subOption.Content"></wang-edit> -->
                  </p>
                </div>
                <!-- <p>答案
                      <wang-edit v-model="answer"></wang-edit>
                    </p> -->
                <p v-if="questionId === 10 || questionId === 2" style="float: right;font-size: 18px;margin-bottom: 0;">
                  <span class="f-c">选项</span>
                  <a-icon type="plus-circle" @click="AddOptions"/>&nbsp;&nbsp;&nbsp;&nbsp;
                  <a-icon type="minus-circle" @click="ReduceOptions"/>
                </p>
              </div>
              <!-- 填空题 -->
              <div v-if="questionId === 5" class="make-select fl">
                <p class="f-c">答案选项</p>
                <div v-for="( i, j) in 1" :key="j" class="clearfix">
                  <!-- 空{{ j + 1 }}
                      <wang-edit v-model="completion[j]"></wang-edit> -->
                  <!-- {{ index }} -->
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
                  <p class="echo-p" v-for="(Syn, k) in EchoSynonymousAnswer" :key="k">
                    <span>{{ Syn }}</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                    <span @click="handleEchoSynonymousAnswer(Syn)" class="cur">删除</span>
                  </p>
                </div>
                <!-- <span class="fg fill-blocks">
                      选项
                      <a-icon type="plus-circle" @click="AddOptions"/>&nbsp;&nbsp;&nbsp;&nbsp;
                      <a-icon type="minus-circle" @click="ReduceOptions"/>
                    </span> -->
              </div>
              <!-- <div v-if="questionId === 23" class="make-select fl">
                    <p class="f-c">答案</p>
                    <wang-edit v-model="answer" style="height: 200px"></wang-edit>
                    <span class="fg agreen-answer" @click="AddSynonym(j)" style="marginTop: 15px">添加同义答案</span>
                    <div v-for="(cou, i) in counter" :key="i" v-show="synonym && num === i">
                      <SynonymAnswer :sd="num"></SynonymAnswer>
                    </div>
                    <span class="sy-blocks" v-show="synonym && num === i">
                      <a-icon type="plus-circle" @click="syAdd"/>&nbsp;&nbsp;&nbsp;&nbsp;
                      <a-icon type="minus-circle" @click="syDel"/>
                    </span>
                  </div> -->
              <div class="make-analysis fl">
                <p class="f-c">解析</p>
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
              <span @click="handleSave">保存</span>
            </div>
          </a-drawer>
        </p>
        <p v-for="(answerdata, i) in ite.AuditPaperItemAnswers" :key="i.Id" v-show="answerdata.Type === 8" style="font-size: 14px">答案: <span style="display: inline-block;margin-left: 20px" v-html="answerdata.Content"></span></p>
        <p v-for="(e,d) in ite.AuditPaperItemAnswers" :key="d" v-show="e.Type === 10" style="font-size: 14px">解析: <span style="display: inline-block;margin-left: 20px" v-html="e.Content"></span></p>
        <p><a-rate disabled :count="ite.Level" v-model="ite.Level"></a-rate>&nbsp;|&nbsp;<span>{{ ite.TypeName }}</span>&nbsp;|&nbsp;<span>已组卷次数{{ ite.OrganizationPaperCount }}</span></p>
      </div>
    </div>
    <!-- <a-empty v-if="noData" /> -->
    <div v-if="noData" class="no-question-bank">
      <p><img src="@/assets/lack/暂无搜索记录.png" alt=""></p>
      <p>暂无题库数据</p>
    </div>
    <a-pagination class="paging" :default-current="1" :total="total" @change="onChange" hideOnSinglePage />
    <!-- <p class="footer">Copyright©上海有我科技有限公司，All Rights Reserved</p> -->
    <footerLogo :heightPage="heightPage"></footerLogo>
  </div>
</template>

<script>
import WangEdit from '@/components/WangEditor/WangEditor'
import SynonymAnswer from '@/components/AddSynonym/AddSynonym'
import footerLogo from '@/components/newFooterLogo/newFooterLogo'

export default {
  components: {
    WangEdit,
    SynonymAnswer,
    footerLogo
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    // 获取年级知识点
    this.getGradeKnowledge()
    // 获取我的题库数据
    this.getMySubjectBank()
    // 添加题目
    this.handleAddSubject()
  },
  updated () {
    this.heightPage = document.getElementById('question-bank').offsetHeight
    console.log('this.heightPage', this.heightPage)
  },
  watch: {
    $route () {
      if (this.$route.fullPath === '/Teacher/My_QuestionBank/QuestionBank') {
        // debugger
        console.log('wath==========')
        // 获取我的题库数据
        this.getMySubjectBank()
        // 添加题目
        this.handleAddSubject()
        this.getGradeKnowledge()
      }
    }
    // SubjectBank: {
    //   handler () {
    //     this.$nextTick(() => {
    //       this.$MathJaxToDom()
    //     })
    //   },
    //   deep: true // true 深度监听可监听到对象、数组的变化
    // }
  },
  data () {
    return {
      heightPage: 0,
      value: 1,
      userId: '',
      // 单选
      alonevalue: '',
      // 知识点
      options: [],
      // 暂无数据
      noData: false,
      // 试题总数
      total: 10,
      // 页码
      pageNo: 1,
      // 难度
      level: 0,
      // 难度颜色
      yel: 0,
      // 题型数据
      subjectList: [],
      // 题型
      itemType: 0,
      // 题库数据
      SubjectBank: [],
      // 知识点
      chapter: '',
      // 编辑题干
      subTitle: '',
      IndividualSettings: false,
      MakeTopic: false,
      questionId: '',
      EditCance: false,
      // 编辑解析
      subAnalysis: '',
      subOptions: [ { Option: 'A' }, { Option: 'B' }, { Option: 'C' } ],
      // 编辑选项性质
      selectValue: '',
      // 添加同意答案数量
      counter: [{}],
      // 同义答案的回显
      EchoSynonymousAnswer: [],
      num: '',
      // 同义答案的显示隐藏
      synonym: false,
      // 默认填空空
      completion: ['', '', ''],
      // 多选题
      MultipleChoice: [],
      // 默认判断
      subJudgeOptions: [{ Option: 'A', Content: '对' }, { Option: 'B', Content: '错' }],
      // 单选的选中值
      judge: '',
      blanksUp: [],
      // 年级
      grade: '',
      gradeId: ''
    }
  },
  methods: {
    // 处理填空
    handleFillBanks (data) {
      data.forEach((item, index) => {
        const reg = /(#&\d+@)/g
        const inputele = ' '
        const stem = item.Title.replace(reg, inputele)
        item.Title = stem
      })
    },
    keyUp () {
      // console.log(this.subTitle)
      const re = /（(.*?)）/g
      let temp = []
      const a = []
      setTimeout(() => {
        while (temp = re.exec(this.subTitle)) {
          // console.log(temp[1])
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
    onChangeSlect (e) {
      console.log(e.target.value)
    },
    // 多选
    onChangeduo (checkedValues) {
      console.log('checked = ', checkedValues)
      this.answer = checkedValues
    },
    // 获取年级知识点
    getGradeKnowledge (id) {
      this.$http.post('/Paper/Exam_SubjectChapter/GetChapterTreeByUserId', { userId: this.userId }).then(res => {
        console.log('年级对应知识点---', res)
        this.options = ''
        this.options = res.Data
        console.log(this.options)
      })
    },
    // 获取我的题库数据   itemType 题型
    getMySubjectBank (gradeId) {
      this.$http.post('/Paper/Exam_Item/GetMyItemList', { userId: this.userId, pageNo: this.pageNo, chapterId: this.chapter, itemType: this.itemType, level: this.level, gradeId: gradeId }).then(res => {
        console.log('我的题库数据---', res)
        this.SubjectBank = res.Data.Items
        this.total = res.Data.TotalItems
        this.handleFillBanks(res.Data.Items)
        this.SubjectBank.forEach(value => {
          const reg = /(#&\d+@)/g
          // const reg = /(#@)/g
          value.Title = value.Title.replace(reg, ' ')
        })
        // console.log('我的题库总数--', this.total)
        if (res.Data.Items.length === 0) {
          this.noData = true
        } else {
          this.noData = false
        }
      })
    },
    // 添加题目
    handleAddSubject () {
      this.$http.post('/Base_Manage/Exam_ItemType/GetDataList', { PageIndex: '1', PageRows: '50' }).then(res => {
        console.log('题型---', res)
        this.subjectList = res.Data
      })
    },
    // 全部题型
    allItemType (id) {
      this.itemType = id
      console.log(id)
      this.getMySubjectBank()
    },
    // 分页
    onChange (pageNumber) {
      console.log('Page: ', pageNumber)
      this.pageNo = pageNumber
      this.getMySubjectBank()
    },
    // 题型的选择
    SubjectSelect (id) {
      this.itemType = +id
      this.pageNo = 1
      console.log('题型的选择', this.itemType)
      this.getMySubjectBank()
    },
    // 知识点选择(我的题库)
    onChangeKnowledge (value) {
      console.log('知识点', value)
      this.chapter = value.join(',')
      console.log(value[0])
      this.getMySubjectBank()
    },
    // 难度
    Difficulty (id) {
      this.yel = id
      this.level = id
      console.log('难度颜色---', id)
      this.getMySubjectBank()
    },
    // 关闭编辑题目
    onCloseMakeTopic () {
      this.MakeTopic = false
    },
    // 单题设置操作按钮控制
    IndividualSubject (itemId) {
      this.IndividualSettings = itemId
      // console.log('单题设置操作按钮控制', this.IndividualSettings)
      // this.aa = '2'
    },
    // 删除单个题目
    deleteAlone (id) {
      this.$http.post('/Paper/Exam_PaperItemMapping/Remove', { id }).then(res => {
        console.log('删除单个题目---', res)
        if (res.Success) {
          this.getMySubjectBank()
          this.$message.success('删除成功')
        } else {
          this.$message.error('删除失败')
        }
      })
    },
    // 取消弹框题目编辑
    handleCancelEdit () {
      this.EditCance = false
      setTimeout(() => { this.MakeTopic = false }, 300)
    },
    // 取消题目编辑
    cancelEdit () {
      this.EditCance = true
      // this.MakeTopic = false
    },
    // 单个题目编辑
    singleEditSubject (id, TypeId, itemType, grade) {
      this.MakeTopic = true
      debugger
      this.questionId = +TypeId
      console.log('当前题目id--', id)
      console.log('当前题目类型---', TypeId)
      // 自身ID
      this.singleEditQuestionId = id
      // 年级
      this.grade = grade
      // parentId
      // this.myTypeQuestionId = parentId
      // itemType
      this.questionItemType = itemType
      // itemtype保存
      // 默认类型
      this.selectValue = +TypeId
      // 回显单个题目题干
      this.handleEchoSubject(id)
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
    // 删除同义答案
    handleEchoSynonymousAnswer (syn) {
      this.$http.post('/Paper/Exam_Item/RemoveAnswer', { itemId: this.singleEditQuestionId, value: syn }).then(res => {
        console.log('删除同义答案---', res)
        if (res.Success) {
          this.handleEchoSubject(this.singleEditQuestionId)
        }
      })
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
    // 回显单个题目
    handleEchoSubject (id) {
      this.$http.post('/Paper/Exam_Item/GetTheData', { id }).then(res => {
        console.log('题干---', res)
        this.subTitle = res.Data.Title
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetAnalysisDataByItemId', { itemId: id }).then(res => {
        console.log('解析---', res)
        this.subAnalysis = res.Data.Content
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetAnswerDataByItemId', { itemId: id }).then(res => {
        console.log('答案---', res)
        debugger
        this.answer = res.Data.Content
        this.alonevalue = res.Data.Content
        this.judge = res.Data.Content
        // debugger
        if (res.Data.Content.length > 1) {
          const arr = res.Data.Content.split('|')
          this.MultipleChoice = arr
        }

        console.log('多选答案---', this.MultipleChoice)
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetOptionsByItemId', { itemId: id }).then(res => {
        console.log('选项---', res)
        debugger
        this.subOptions = res.Data
      })
      this.$http.post('/Paper/Exam_Item/GetMoreItemAnswer', { itemId: id }).then(res => {
        console.log('同义答案---', res)
        // this.subOptions = res.Data
        this.EchoSynonymousAnswer = res.Data
      })
    },
    // 保存编辑题目
    handleSave () {
      console.log('题干---', this.subTitle) //
      console.log('选项类型---', this.selectValue)
      console.log('解析---', this.subAnalysis) //
      console.log('选项---', this.subOptions) //
      console.log('填空---', this.completion) //
      console.log('答案---', this.answer) //
      console.log('题型ItemId---', this.questionItemType)
      // console.log('自身类型id---', this.myTypeQuestionId)
      // console.log('编辑单个题目id--', this.singleEditQuestionId)
      console.log('同意答案---', this.allSynonymAnswer)
      console.log('年级---', this.grade)
      // console.log('章节---', this.chapterSelect)
      // console.log('难度---', this.levelSelect)
      if (this.selectValue === 5) {
        // const blanks = this.completion.substring(heade + 1, end).split('').join('|')
        const blanks = this.completion.map(item => {
          return item.replace(/<[^<>]+>/g, '')
        })
        const subBlanks = blanks.join('|')
        console.log(subBlanks)
        this.answer = subBlanks
      }
      // if (this.answer.length > 1 && this.selectValue !== 5 && this.selectValue === 10) {
      //   this.answer = this.answer.join('|')
      // }
      if (this.selectValue === 11) {
        this.subOptions = this.subJudgeOptions
      }

      this.$http.post('/Paper/Exam_Item/UpdateMyItem', {
        Analysis: this.subAnalysis,
        ItemType: this.questionItemType,
        Title: this.subTitle,
        Options: this.subOptions,
        GradeTerm: this.grade,
        TypeId: this.selectValue,
        Answer: this.answer,
        // parentId: this.myTypeQuestionId,
        // paperId: this.paperId,
        // 知识点id
        // chapter: this.chapterSelect,
        // 单题的id（用于修改）
        Id: this.singleEditQuestionId,
        // 同意答案
        OtherAnswer: this.allSynonymAnswer
        // level: this.levelSelect
      }).then(res => {
        console.log('保存录入题目---', res)
        if (res.Success) {
          this.$message.success('保存成功')
          // this.getMySubjectBank()
          this.MakeTopic = false
          this.getMySubjectBank()
        }
      })
    },
    // 跳转录入题目
    InputSubject () {
      this.$router.push('/Teacher/My_QuestionBank/SubjectAttribute')
    },
    // 难度选择
    starChange (value) {
      console.log(value)
    }
  }
}
</script>

<style lang="less" scoped>
  .yellow {
    color: #eac718;
  }
  .col-mytest {
    color: #6BBD9A;
    border: none;
  }
  .bor {
    border-top: 1px solid #ccc;
  }
  .no-question-bank {
    text-align: center;
    color: rgba(165, 166, 167);
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
  .question-bank {
    margin-bottom: 15px;
    padding: 20px 15px;
    background-color: #fff;
    border-radius: 5px;
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
  // 录入题目
  .input-title {
      display: inline-block;
      width: 100px;
      height: 40px;
      line-height: 40px;
      text-align: center;
      border-radius: 35px;
      color: #fff;
      background-color: #68BB97;
      cursor: pointer;
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
  // 题目
  .subject-tip {
    padding: 24px 24px 0 24px;
    border-radius:5px;
    margin-bottom: 20px;
    background-color: #fff;
    cursor:pointer;
  }

   // 题型边框
  .qus-type {
    border: 1px dotted #D6D6D6;
  }
  .z-span {
        font-size: 14px;
        float: right;
        margin-right: 32px;
      }
  // 题型
  .question-type {
    span {
      font-size: 16px;
      margin-left: 20px;
      cursor: pointer;
    }
  }
  // 难度
  .difficulty {
    display: flex;
    li {
      font-size: 16px;
      margin-right: 20px;
      cursor: pointer;
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
      .paging {
        margin-top: 40px;
        text-align: center;
         /deep/.ant-pagination-item:focus, .ant-pagination-item:hover {
            border-color: #68BB97;
            border-color: #68BB97;
        }
        /deep/.ant-pagination-item-active a {
          color: #000;
          background-color: #68BB97;
        }
      }
</style>
