<template>
  <div>
    <div id="correcting-data">
      <!-- 试卷名称 -->
      <p class="title">{{ subjectList.Title }}</p>
      <!-- 试卷信息 -->
      <p class="paper-infor">
        <span>批改进程</span>
        <span>
          <a-progress
            :percent="(completeArrNum / itemlist.length) * 100"
            :showInfo="showInfo"
            :strokeWidth="strokeWidth"
          />
        </span>
        <!-- subNum -->
        <span>{{ completeArrNum }}/{{ itemlist.length }}</span>
        <span class="icon">V1</span>
        <span>{{ stuName }}</span>
        <span>{{ subjectList.ClassName }}</span>
        <span @click="submit" class="submit-correcting">提交</span>
      </p>
      <!-- 题目信息 -->
      <div class="sub-title">
        <!-- <p v-for="item in subjectInforList" :key="item.ItemId" v-html="item.Title"> </p> -->
        <p v-html="this.currentItem.Title"></p>
      </div>
      <!-- 学生 -->
      <div class="student-info" style="display: flex">
        <div class="module-list-wrap">
          <div style="position: relative">
            <swiper class="swiper" ref="swiper" :options="swiperOption">
              <swiper-slide style="text-align: center" v-for="(item, index) in itemlist" :key="index">
                <div
                  @click="revierewTest(item.StudentId, item.StudentName, item, index)"
                  class="li-div cur"
                  :class="{ col: active === item.StudentId }"
                >
                  <img :src="item.Photo" alt style="width: 40px; border-radius: 35px" />
                  <p>
                    <span style="margin-right: 5px">{{ item.UserPinyin }}</span>
                    <span>{{ item.StudentName }}</span>
                  </p>
                </div>
                <div v-if="item.HasCorrect === 1" class="complete-img">
                  <img src="@/assets/correcting/完成批阅.png" alt="" />
                </div>
              </swiper-slide>
            </swiper>

            <div class="left-arrow cur" slot="button-prev" @click="prevPhoto">
              <img class="arrow-img" src="@/assets/correcting/左边箭头.png" />
            </div>
            <div class="right-arrow cur" slot="button-next" @click="nextPhoto">
              <img class="fg arrow-img" src="@/assets/correcting/右边箭头.png" />
            </div>
          </div>
        </div>
      </div>
      <!-- 批改的画布 -->
      <div class="correcting-infor" style="display: flex">
        <img v-if="oneKeyMarks" class="correct-icon" src="@/assets/teacher/icon-输入正确 2.png" alt="" />
        <!-- <img class="prev-img cur" @click="prev()" src="@/assets/correcting/左边箭头.png" /> -->
        <!-- <img class="next-img cur" @click="next()" src="@/assets/correcting/右边箭头.png" /> -->
        <div class="correcting-canvas">
          <canvas id="c" width="800" height="500" class="canvas-img"></canvas>
          <!-- 录音语音泡 -->
          <!-- v-if="soundTime !== '' && soundTime > 0" -->
          <span
            v-if="soundTime !== '' && soundTime > 0"
            @mouseenter="showCollDel"
            @mouseleave="hideCollDel"
            style="display: inline-block"
          >
            <!-- 收藏语音 -->
            <img
              @click="collectionSound"
              class="cur"
              v-show="collDel"
              style="vertical-align: super; margin-right: 8px"
              src="@/assets/teacher/收藏.png"
              alt=""
            />
            <a-modal v-model="collectionPopup" title="收藏并编辑语音标签" @ok="handleCollection" width="630px" centered>
              <input class="coll-popup-input" v-model="soundLabelTitle" type="text" placeholder="不超过6个字" />
            </a-modal>
            <span @click="playSound" class="sound-show" :class="{ active: activePlay === '1' }">{{ soundTime }}</span>
            <!-- 删除语音 -->
            <span
              ><img
                v-show="collDel"
                class="cur"
                style="vertical-align: super; margin-left: 8px"
                src="@/assets/correcting/减少语音.png"
                alt=""
            /></span>
          </span>
          <p class="comment-display">评语: &nbsp;&nbsp; {{ newMyCommtent }}</p>
          <div class="app-btn clearfix">
            <span @click="changeEditMode('pen')">
              <img src="@/assets/correcting/批改作业／批改笔.png" alt />
              <p>批注</p>
            </span>
            <span @click="changeEditMode('arrow')">
              <img src="@/assets/correcting/批改作业／批改笔(1).png" alt />
              <p>箭头</p>
            </span>
            <span @click="changeEditMode('line')">
              <img src="@/assets/correcting/批改作业／批改笔(2).png" alt />
              <p>直线</p>
            </span>
            <!-- <el-button
            size="mini"
            @click="changeEditMode('dottedline')"
          >虚线</el-button>-->

            <span @click="changeEditMode('rectangle')">
              <img src="@/assets/correcting/批改作业／批改笔(3).png" alt />
              <p>方框</p>
            </span>
            <!-- <el-button
            size="mini"
            @click="changeEditMode('circle')"
          >正圆</el-button
          >-->
            <span @click="changeEditMode('ellipse')">
              <img src="@/assets/correcting/批改作业／批改笔(4).png" alt />
              <p>椭圆</p>
            </span>
            <!-- <el-button size="mini" @click="changeEditMode('text')">文字</el-button> -->
            <!-- <el-button
            size="mini"
            @click="changeEditMode('remove')"
          >擦除</el-button
          >-->
            <span @click="undo" :disabled="!config.undoStatus">
              <img src="@/assets/correcting/批改作业／批改笔(5).png" alt />
              <p>撤销</p>
            </span>
            <!-- <el-button size="mini" @click="saveImg">保存</el-button> -->
            <div class="fg subject-btn">
              <span class="judge" style="cursor: auto" v-if="assignment">
                <img v-if="fullMark === fristFraction" src="@/assets/correcting/对号.png" alt="" />
                <img v-if="fullMark === 0" src="@/assets/correcting/错误.png" alt="" />
                <img v-if="fullMark !== fristFraction && fullMark !== 0" src="@/assets/correcting/半对.png" alt="" />
              </span>
              <span v-if="assignment">
                得分
                <img @click="BonusPoints" src="@/assets/correcting/增加.png" alt /> &nbsp;&nbsp;
                <input @blur="setScore" v-model="fullMark" type="text" /> &nbsp;&nbsp;
                <img @click="MinusPoints" src="@/assets/correcting/减少.png" alt />
              </span>
              <span v-if="assignment" @click="FullMarks()" class="bys pad">一键满分</span>
              <span @click="prev()" class="bys ad">上一题</span>
              <!-- v-if="this.subjectNumNew !== this.itemCnt" -->
              <span @click="next()" class="bys ad">下一题</span>
              <!-- <span @click="stuPrev" class="bys ad">上一份</span> -->
              <!-- <span @click="stuNext" class="bys ad">下一份</span> -->
              <!-- <a-modal
              v-model="savePhoto"
              title="温馨提示:"
              width="630px"
              @ok="handleSavePhoto()"
            >
              <div>
                上一题还未保存，如果关闭批改的试题将不做保存
              </div>
            </a-modal> -->
              <!-- <span @click="submit" v-if="this.subjectNumNew === this.itemCnt" class="bys ad">提交</span> -->
              <!-- <p>
                <p v-if="!assignment">得分: 15</p>
                <p>满分: {{ fristFraction }}分</p><span>题目: {{ subjectNumNew }} / {{ TotalCount }}</span>
              </p> -->
              <p>
                <span v-if="!assignment">得分: {{ score }}</span>
                <span>满分: {{ fristFraction }}分</span><span>题目: {{ subjectNumNew }} / {{ TotalCount }}</span>
              </p>
            </div>
            <!-- 推荐答案 -->
            <!-- <p class="recommend-data">
              <span class="recommend-img"><img src="@/assets/correcting/推荐.png" alt=""></span>
              推荐为参考答案:
              <a-radio-group name="radioGroup" :default-value="1" @change="whetherPush">
                <a-radio :value="1">
                  否
                </a-radio>
                <a-radio :value="2">
                  是
                </a-radio>
              </a-radio-group>
              <a-checkbox-group
                v-model="value"
                v-if="this.WhetherPush === 2"
                name="checkboxgroup"
                :options="plainOptions"
                @change="onChange"
              />
            </p> -->
          </div>
          <!-- 录音弹窗 -->
          <div v-if="soundPopup" class="sound-popup">
            <img @click="soundShow" v-show="stopItHide" src="@/assets/correcting/录音弹窗.png" alt />
            <!-- <img @click="hideSound" v-show="stopIt" src="@/assets/correcting/点击停止.png" alt /> -->
            <div v-show="stopIt" @click="hideSound" class="click-stop"></div>
            <p v-show="stopItHide">支持最多60s语音</p>
            <p v-show="stopItHide" @click="soundShow">点击说话</p>
            <p v-show="stopIt && showCountDown">{{ content }}</p>
            <p v-show="stopIt" @click="hideSound">点击停止</p>
          </div>
        </div>
        <div class="canvas-btn">
          <div>
            <p @click="rotate90()">
              <img src="@/assets/correcting/批改作业／图片旋转.png" alt />
            </p>
            <p>图片旋转</p>
          </div>
          <div>
            <p @click="handleComment">
              <img src="@/assets/correcting/评语.png" alt />
            </p>
            <p>评语库</p>
          </div>
          <a-modal v-model="visible" title="我的评语库" @cancel="handleCommentOk" width="630px" :footer="null" centered>
            <ul class="comtent">
              <li v-for="item in comtentList" :key="item.Id">
                <p v-if="flag" class="clearfix">
                  {{ item.Comment }}
                  <span class="fg cur mr" @click="deleteComtent(item.Id)">删除</span>
                  <span class="fg cur mr" @click="edit(item.Id)">编辑</span>
                </p>
                <p v-if="editContent === item.Id" class="input-comtent">
                  <input v-model="item.Comment" type="text" />
                  <span @click="editComtent(item.Id, item.Comment)">确定</span>
                  <span @click="cancelEdit">取消</span>
                </p>
              </li>
              <li>
                <p v-if="flagAdd" class="input-comtent">
                  <input v-model="addComtentTipc" type="text" />
                  <span @click="handleAddComtent">确定</span>
                  <span @click="handleCancelAdd">取消</span>
                </p>
                <p class="clearfix">
                  <img @click="addComtent" class="fg" src="@/assets/correcting/编组.png" alt />
                </p>
              </li>
            </ul>
          </a-modal>
          <div>
            <p @click="handleWords">
              <img src="@/assets/correcting/文字.png" alt />
            </p>
            <p>文字</p>
          </div>
          <a-modal v-model="writeComment" title="写评语" @ok="handleOk" width="630px">
            <div>
              <div>
                评语参考: &nbsp;&nbsp;&nbsp;
                <span>
                  <a-select
                    style="width: 349px"
                    @change="handleChange"
                    placeholder="请选择"
                    v-model="myCommtent"
                    centered
                  >
                    <a-select-option :value="item.Comment" v-for="item in comtentList" :key="item.Id">{{
                      item.Comment
                    }}</a-select-option>
                  </a-select>
                </span>
              </div>
              <div>
                <textarea class="text-area" v-model="newMyCommtent"></textarea>
              </div>
            </div>
          </a-modal>
          <div>
            <!-- <p >
              <img @click="showSoundBank" src="@/assets/correcting/录音库.png" alt />
            </p>
            <p >录音库</p> -->
            <a-modal v-model="soundBank" title="我的语音库" @ok="handleSoundBank" width="630px">
              <p>
                1, 整数的理解 <span>语音。。。</span> <span @click="sendSound" class="recording-library fg">发送</span>
                <span @click="deleteSound" class="recording-library fg" style="color: #ccc">删除</span>
              </p>
            </a-modal>
          </div>
          <div>
            <p @click="soundRecording">
              <img src="@/assets/correcting/录音.png" alt />
            </p>
            <p>录音</p>
          </div>
        </div>
      </div>
    </div>
    <!-- <footerLogo :heightPage="heightPage"></footerLogo> -->
  </div>
</template>

<script>
import { fabric } from 'fabric'
import Recorder from 'js-audio-recorder'
import { Swiper, SwiperSlide } from 'vue-awesome-swiper'
import 'swiper/swiper-bundle.css'
import footerLogo from '@/components/newFooterLogo/newFooterLogo'

// import Swiper from 'swiper'
// import fabric from '@/utils/fabric.min.js'
const recorder = new Recorder()
export default {
  name: 'SwiperExampleChangeDirection',
  title: 'Change direction',
  components: {
    Swiper,
    SwiperSlide,
    fabric,
    footerLogo,
  },
  created() {
    this.userId = localStorage.getItem('UserId')
    this.classId = this.$route.query.classId
    this.paperId = this.$route.query.paperId
    console.log(this.paperId)
    // 获取主观题的信息
    this.getSubjectiveQuestions()
  },
  updated() {
    this.heightPage = document.getElementById('correcting-data').offsetHeight
    // console.log('元素高---', this.heightPage)
  },
  watch: {
    $route() {
      const ChangId = window.location.href.split('?')[1]
      this.paperId = this.$route.query.paperId
      this.classId = this.$route.query.classId
      if (this.$route.fullPath === '/Teacher/My_Task/InspectionWork/BatchModify?' + ChangId) {
        this.getSubjectiveQuestions()
      }
    },
  },
  mounted() {},

  data() {
    return {
      // 默认得分
      score: 0,
      // 是否展示赋分
      assignment: false,
      heightPage: 0,
      // 保存语音标签
      soundLabelTitle: '',
      plainOptions: [
        { label: '推荐给我带的班级', value: '1' },
        { label: '推荐到本校', value: '2' },
        { label: '推荐到全区', value: '3' },
      ],
      value: [],
      swiperOption: {
        slidesPerView: 3,
        spaceBetween: 30,
        direction: 'horizontal',
        navigation: {
          nextEl: '.swiper-button-next',
          prevEl: '.swiper-button-prev',
          hideOnClick: true,
        },
        // on: {
        //   resize: () => {
        //     this.$refs.swiper.changeDirection(
        //       window.innerWidth <= 960
        //         ? 'vertical'
        //         : 'horizontal'
        //     )
        //   }
        // }
      },
      existenceId: false,
      compltetVivew: '',
      // 下题保存
      savePhoto: false,
      // 当前题目
      subNum: 1,
      subjectNumNew: 1,
      // 学生列表
      itemlist: [],
      // 试卷信息
      subjectList: {},
      // 主观题单个题目
      subjectInforList: [],
      // 主观题总人数
      TotalCount: '',
      // 主观题图片
      imgSrc: '',
      // 题目数
      subjectNum: '',
      // 默认第一题满分
      fristFraction: '',
      xxx: 0,
      userId: '',
      //
      classId: '',
      paperId: '',
      // 当前学生姓名
      stuName: '',
      // 批阅状态
      active: '',
      newActive: '',
      activePlay: '',
      itemCnt: 0,
      // 满分
      fullMark: 0,
      // 试题的id
      subjectItemId: '',
      // 老师批阅需要的分数id
      ScoreId: '',
      width: 500,
      height: 400,
      showInfo: false,
      // 录音弹窗
      soundPopup: false,
      // 语音收藏/删除
      collDel: false,
      // 录音提示
      content: '',
      // 录音倒计时
      totalTime: 60,
      stopIt: false,
      stopItHide: false,
      // 录音时长
      soundTime: '',
      // 录音时长显示
      showCountDown: false,
      // 语音库
      soundBank: false,
      // 语音库编辑
      collectionPopup: false,
      // 评语库
      comtentList: [],
      // 评语
      visible: false,
      flag: true,
      editContent: '',
      flagAdd: false,
      // 写评语
      writeComment: false,
      comtent: '评语评语评语评语',
      // 添加评语
      addComtentTipc: '',
      // 自己的评语
      myCommtent: '',
      newMyCommtent: '',
      strokeWidth: 20,
      red: 0,
      chooseForm: {
        manSex: true,
        manPos: true,
        editMode: true,
      },
      mouseFrom: {},
      mouseTo: {},
      canvas: null,
      drawingObject: null, // 当前绘制对象
      drawType: '',
      textbox: null, // 文本编辑状态
      color: '#E34F51', // 画笔颜色
      drawWidth: 2, // 笔触宽度
      moveCount: 1, // 绘制移动计数器
      doDrawing: false, // 绘制状态
      config: {
        canvasState: [],
        currentStateIndex: -1,
        undoStatus: false, // 撤销状态
        undoFinishedStatus: 1,
      },
      currentStudent: {},
      currentItem: {},
      isFirst: true,
      angle: 90,
      // 推荐选择’
      WhetherPush: 1,
      // 一键满分图标
      oneKeyMarks: false,
      completeArrNum: 0,
      CorrectionProgress: '',
      evaluateArr: [] // 评价
    }
  },
  methods: {
    prevPhoto() {
      console.log('左')
      this.$refs.swiper.$swiper.slidePrev()
    },
    nextPhoto() {
      console.log('右')
      this.$refs.swiper.$swiper.slideNext()
    },
    // 上一题
    prev() {
      // 评语
      this.newMyCommtent = ''
      // 录音
      this.soundTime = ''
      let studentIdx = 0
      let itemIdx = 0
      // 找学生索引
      for (var i = 0; i < this.itemlist.length; i++) {
        if (this.itemlist[i].StudentId === this.currentStudent.StudentId) {
          studentIdx = i
        }
      }

      // 找题目索引
      var items = this.itemlist[studentIdx].SubjectiveItemInfos
      for (var i = 0; i < items.length; i++) {
        if (items[i].ItemId === this.currentItem.ItemId) {
          itemIdx = i
        }
      }

      if (itemIdx > 0) {
        itemIdx -= 1
      }

      if (this.itemlist[studentIdx].SubjectiveItemInfos[itemIdx]) {
        this.currentItem = this.itemlist[studentIdx].SubjectiveItemInfos[itemIdx]
      }
      console.log('上一题---', this.currentItem)
      if (this.subjectNumNew === 1) {
        this.subjectNumNew = 1
        return this.$message.success('已经没有上一题了')
      } else {
        this.subjectNumNew--
      }
      this.fristFraction = this.currentItem.ItemScore
      this.score = this.currentItem.TeacherScore
      this.imgSrc = this.currentItem.Url
      this.subjectItemId = this.currentItem.ItemId
      console.log(this.subjectItemId)

      this.config = {
        canvasState: [],
        currentStateIndex: -1,
        undoStatus: false, // 撤销状态
        undoFinishedStatus: 1,
      }

      this.isFirst = true
      this.initCanvas()
    },
    // 下一题
    next(index) {
      this.savePhoto = true
      // 评语
      this.newMyCommtent = ''
      // 录音
      this.soundTime = ''
      let studentIdx = 0
      let itemIdx = 0
      // 分数
      this.saveSubjectSore()
      this.initCanvas(true)

      // 保存批阅关系
      // this.saveReviewRelationship()
      // this.fullMark = 0
      debugger
      if (this.CorrectionProgress === '1') {
        for (var i = 0; i < this.itemlist.length; i++) {
          if (this.itemlist[i].StudentId === this.currentStudent.StudentId) {
            studentIdx = i
          }
        }

        var items = this.itemlist[studentIdx].SubjectiveItemInfos
        for (var i = 0; i < items.length; i++) {
          if (items[i].ItemId === this.currentItem.ItemId) {
            itemIdx = i
          }
        }

        this.itemCnt = this.itemlist[studentIdx].SubjectiveItemInfos.length
        // console.log('长度---', itemCnt)
        if (this.itemCnt > itemIdx) {
          itemIdx += 1
        }
        if (this.subjectNumNew === this.itemCnt) {
          this.subjectNumNew = this.itemCnt
          this.subjectItemId = this.currentItem.ItemId

          // 重新加载
          // this.getSubjectiveQuestions()
          return this.$message.success('已经是最后一题了')
        } else {
          this.subjectNumNew++
          if (this.itemlist[studentIdx].SubjectiveItemInfos[itemIdx]) {
            this.currentItem = this.itemlist[studentIdx].SubjectiveItemInfos[itemIdx]
          }
        }

        console.log('362362626226---', this.currentItem)
        this.ScoreId = this.currentItem.Id
        this.fristFraction = this.currentItem.ItemScore
        this.score = this.currentItem.TeacherScore
        this.imgSrc = this.currentItem.Url
        console.log('下一题的图片', this.imgSrc)
        this.initCanvas(true)
        this.config = {
          canvasState: [],
          currentStateIndex: -1,
          undoStatus: false, // 撤销状态
          undoFinishedStatus: 1,
        }
        this.isFirst = true

        this.subjectItemId = this.currentItem.ItemId
      }
      this.fullMark = 0
    },
    // 上一份
    stuPrev() {
      // if ()
      let stuIndex = 0
      this.itemlist.forEach((item, index) => {
        if (item.StudentId === this.currentStudent.StudentId) {
          stuIndex = index
        }
      })
      const stuList = this.itemlist[stuIndex - 1]
      const studentInx = stuIndex - 1
      if (studentInx === -1) {
        return this.$message.success('已经没有上一个学生了')
      }
      this.revierewTest(stuList.StudentId, stuList.StudentName, stuList, stuIndex, studentInx)
    },
    // 下一份
    stuNext() {
      debugger

      this.$uwonhttp
        .post('/ExamItem/ExamItem/UploadTeacherPicBase64', {
          itemId: this.currentItem.ItemId,
          studentId: this.currentStudent.StudentId,
          paperId: this.paperId,
          base64: this.currentItem.Url,
        })
        .then((res) => {
          console.log('保存图片---', res)
          if (res.data.Success) {
            this.saveReviewRelationship()
          }
        })
      let stuIndex = 0
      this.itemlist.forEach((item, index) => {
        if (item.StudentId === this.currentStudent.StudentId) {
          stuIndex = index
        }
      })
      const stuList = this.itemlist[stuIndex + 1]
      const studentInx = stuIndex + 1
      if (studentInx === this.itemlist.length) {
        return this.$message.success('已经到最后一个学生了')
      }
      this.revierewTest(stuList.StudentId, stuList.StudentName, stuList, stuIndex, studentInx)
      // this.saveReviewRelationship()
    },
    //
    // 提交
    submit() {
      // debugger
      const flag = this.itemlist.some((value) => {
        return value.HasCorrect === 0
      })
      if (flag) {
        return this.$message.warning('还有作业未批改，请统一批改完成后提交')
      } else {
        this.saveImg(true)
        // 分数
        this.saveSubjectSore()
        // 保存批阅关系
        // this.saveReviewRelationship()
        this.compltetVivew = this.active
        console.log('学生的id---', this.active)
        console.log(this.itemlist)
        // 点击提交就是批阅当前学生完成
        this.currentStudent.HasCorrect = 1
        console.log('当前学生信息---', this.currentStudent)
        if (this.itemlist) {
          const arr = this.itemlist.indexOf(this.active)
          if (arr !== -1) {
            this.existenceId = true
          }
        }
      }

      // console.log('有么有学生的id---', arr)
    },
    // 给满分
    FullMarks() {
      this.fullMark = this.fristFraction
      this.oneKeyMarks = true
      setTimeout(() => {
        this.oneKeyMarks = false
      }, 800)
    },
    // 加分
    BonusPoints() {
      if (this.fullMark !== this.fristFraction) {
        this.fullMark += 1
      }
      if (this.fullMark === this.fristFraction) {
        this.$message.success('不能超过满分')
      }
    },
    // 减分
    MinusPoints() {
      if (this.fullMark !== 0) {
        this.fullMark -= 1
      }
    },
    // 保存批阅关系
    saveReviewRelationship() {
      debugger
      this.$uwonhttp
        .post('/ExamItem/ExamItem/SavePyStudent', {
          classId: this.classId,
          paperId: this.paperId,
          itemId: this.subjectItemId,
          teacherId: this.userId,
          studentId: this.active,
        })
        .then((res) => {
          console.log('保存批阅关系', res)
        })
    },
    // 保存题目分数
    saveSubjectSore() {
      this.$uwonhttp
        .post('/ExamItem/ExamItem/TeacherSetScore', { id: this.ScoreId, score: this.fullMark })
        .then((res) => {
          console.log('保存题目分数---', res)
        })
    },
    // 输入分数
    setScore() {
      console.log(+this.fullMark)
      this.fullMark = +this.fullMark
    },
    // 获取主观题的信息
    getSubjectiveQuestions() {
      // debugger
      this.$uwonhttp
        .post('/ExamItem/ExamItem/GetSubjectiveStduents', { classId: this.classId, paperid: this.paperId })
        .then((res) => {
          console.log('试卷信息---', res)
          // 试卷信息
          this.subjectList = res.data.Data
          // 学生列表
          this.itemlist = res.data.Data.StudentInfos
          const reg = /(#&\d+@)/g
          //  const reg = /(#@)/g
          this.itemlist.forEach((value) => {
            // console.log('一侧',value)
            value.SubjectiveItemInfos.forEach((ite) => {
              // console.log('二层',ite)
              // console.log('二层',ite.Title)
              ite.Title = ite.Title.replace(reg, '')
              //  console.log('3层',ite.Title)
            })
          })
          const completeArr = this.itemlist.filter((value) => {
            return value.HasCorrect === 1
          })
          this.completeArrNum = completeArr.length
          console.log('完成的批阅', completeArr)

          console.log('学生列表', this.itemlist)
          if (this.itemlist[0].HasCorrect === 1) {
            this.assignment = false
          } else {
            this.assignment = true
          }
          // 默认第一道题的id
          this.subjectItemId = res.data.Data.StudentInfos[0].SubjectiveItemInfos[0].ItemId
          console.log('默认第一道题的id---', this.subjectItemId)
          // 主观题单个题目
          this.subjectInforList = res.data.Data.StudentInfos[0].SubjectiveItemInfos
          // 主观题总人数
          this.TotalCount = res.data.Data.StudentInfos[0].SubjectiveItemInfos.length
          // console.log('主观题题目---', this.subjectInforList)
          // 当前学生姓名
          this.stuName = res.data.Data.StudentInfos[0].StudentName
          // 默认选中第一个学生
          this.active = res.data.Data.StudentInfos[0].StudentId
          // console.log('默认第一个学生---', this.active)
          // 默认第一张图片
          this.imgSrc = res.data.Data.StudentInfos[0].SubjectiveItemInfos[0].Url
          // console.log('第一张图片', this.imgSrc)
          // 默认的满分
          this.fristFraction = res.data.Data.StudentInfos[0].SubjectiveItemInfos[0].ItemScore
          this.score = res.data.Data.StudentInfos[0].SubjectiveItemInfos[0].TeacherScore
          // 默认的第一题分数id
          this.ScoreId = res.data.Data.StudentInfos[0].SubjectiveItemInfos[0].Id
          // console.log(' 默认的第一题分数id---', this.ScoreId)

          console.log('批阅的所有题目', this.compltetVivew)

          this.currentStudent = res.data.Data.StudentInfos[0]
          this.currentItem = res.data.Data.StudentInfos[0].SubjectiveItemInfos[0]

          this.initCanvas()
        })

      // this.active = this.newActive
    },
    // 批阅试题
    revierewTest(id, name, item, index) {
      // debugger
      this.saveImg()
      // this.$uwonhttp.post('/ExamItem/ExamItem/UploadTeacherPicBase64', { itemId: this.currentItem.ItemId, studentId: this.currentStudent.StudentId, paperId: this.paperId, base64: this.currentItem.Url }).then(res => {
      //     console.log('保存图片---', res)
      //     if (res.data.Success) {
      //       this.saveSubjectSore()
      //       this.$uwonhttp.post('/ExamItem/ExamItem/SavePyStudent', { classId: this.classId, paperId: this.paperId, itemId: this.currentItem.ItemId, teacherId: this.userId, studentId: this.currentStudent.StudentId }).then(res => {
      //   console.log('保存批阅关系', res)
      // })
      //     }
      //   })
      // debugger
      this.fullMark = 0
      // 评语
      this.myCommtent = ''
      this.newMyCommtent = ''
      this.evaluateArr = []
      // 录音
      this.soundTime = ''
      // 选中状态
      this.active = id
      this.newActive = id
      console.log('学生id---', this.active)
      this.stuName = name
      this.subjectNumNew = 1
      console.log('点击学生时---', item)
      this.score = item.SubjectiveItemInfos[0].TeacherScore
      if (item.HasCorrect === 1) {
        this.assignment = false
      } else {
        this.assignment = true
      }
      this.subjectItemId = item.SubjectiveItemInfos[0].ItemId
      this.subNum = index + 1
      console.log('当前学生下标---', index)
      // this.getSubjectiveQuestions()
      this.currentStudent = item
      debugger
      this.currentItem = item.SubjectiveItemInfos[0]
      this.imgSrc = this.currentItem.Url
      // 题目总数
      this.TotalCount = item.SubjectiveItemInfos.length
      console.log('1111111111112=--------当前学生的题干', this.currentItem)
      console.log('----------------图片', this.imgSrc)

      // 重新初始化撤回结构，需要设置 isFirst = true
      this.config = {
        canvasState: [],
        currentStateIndex: -1,
        undoStatus: false, // 撤销状态
        undoFinishedStatus: 1,
      }
      this.isFirst = true

      this.initCanvas()
      this.state()
    },
    state() {
      this.$uwonhttp
        .post('/ExamItem/ExamItem/GetSubjectiveStduents', { classId: this.classId, paperid: this.paperId })
        .then((res) => {
          const arr = []

          for (let i = 0; i < res.data.Data.StudentInfos.length; i++) {
            this.itemlist[i].HasCorrect = res.data.Data.StudentInfos[i].HasCorrect
          }
          //  res.data.Data.StudentInfos.forEach(value => {
          //    arr.push(value.HasCorrect)
          //  })
          //  console.log(arr)
        })
    },
    edit(id) {
      this.flag = false
      this.editContent = id
    },
    // input () {
    //   this.flag = true
    //   this.editComtent()
    // },
    // 文字
    handleWords() {
      this.writeComment = true
      this.getComment()
    },
    // 文字确定
    handleOk() {
      this.writeComment = false
      this.flagAdd = false
      this.$uwonhttp
        .post('/ExamItem/ExamItem/SetTeacherMsg', {
          paperId: this.paperId,
          itemId: this.subjectItemId,
          studentId: this.active,
          teacherMsg: this.newMyCommtent
        })
        .then((resJson) => {
          console.log('文字添加成功')
        })
    },
    // 旋转
    rotate90(imgUrl, type) {
      const that = this
      // 1. 创建图片，canvas,获取画布
      let cwidth = this.canvas.width
      let cheight = this.canvas.height

      this.canvas.clear()
      let left = 0
      let top = 0
      const ang = that.angle / 90

      //       fabric.Image.fromURL(
      //         this.imgSrc,
      //         (oImg) => {
      // // 判断当前的宽高
      // var rate;
      // let left;
      // let top;
      /* if(oImg.width>oImg.height){
  // 比率为
  rate= this.canvas.width/oImg.width;
  oImg.width=this.canvas.width
  oImg.height=rate*oImg.height
   


}
if(oImg.width<oImg.height){
  // 比率为
  rate= this.canvas.height/oImg.height;
  oImg.width=rate*oImg.width
  oImg.height=this.canvas.height
   left=(this.canvas.width-oImg.width)/2
   top=(this.canvas.height-oImg.height)/2
 

}
*/
      // debugger
      // oImg.scaleToWidth(this.canvas.width)
      // oImg.scaleToHeight(this.canvas.height)
      // left=(this.canvas.width-oImg.width)/2
      // top=(this.canvas.height-oImg.height)/2

      //       if (ang === 1) {
      //         // left = this.canvas.width-(this.canvas.width-oImg.width)/2
      //         // top = (this.canvas.height-oImg.height)/2
      //         left=800
      //         top=0

      //       } else if (ang === 2) {
      // /*         left = this.canvas.width-(this.canvas.width-oImg.width)/2
      //         top = this.canvas.height-(this.canvas.height-oImg.height)/2 */
      //         left=800
      //         top=500
      //       } else if (ang === 3) {
      // /*         left = (this.canvas.width-oImg.width)/2
      //         top = this.canvas.height-(this.canvas.height-oImg.height)/2 */
      //         left=0
      //         top=500
      //       } else if (ang === 4) {
      //        /*  left = (this.canvas.width-oImg.width)/2
      //         top =(this.canvas.height-oImg.height)/2 */
      //         left=0
      //         top=0
      //       }

      //           oImg.set({
      //           crossOrigin: 'Anonymous',
      //           left,
      //           top,
      //           // scaleX: rate,
      //           // scaleY: rate,
      //           angle: that.angle,
      //             originX: 'center',
      //             originY: 'center',
      //           })

      //           this.canvas.add(oImg)
      //         }

      //       )

      if (ang === 1) {
        // left = this.canvas.width-(this.canvas.width-oImg.width)/2
        // top = (this.canvas.height-oImg.height)/2
        left = 550
        top = 0
      } else if (ang === 2) {
        debugger
        /*         left = this.canvas.width-(this.canvas.width-oImg.width)/2
        top = this.canvas.height-(this.canvas.height-oImg.height)/2 */
        left = 800
        top = 500
      } else if (ang === 3) {
        /*         left = (this.canvas.width-oImg.width)/2
        top = this.canvas.height-(this.canvas.height-oImg.height)/2 */
        left = 250
        top = 500
      } else if (ang === 4) {
        /*  left = (this.canvas.width-oImg.width)/2
        top =(this.canvas.height-oImg.height)/2 */
        left = 0
        top = 0
      }

      fabric.Image.fromURL(
        this.imgSrc,
        (oImg) => {
          var rate;
          if (ang === 1) {
            rate = this.canvas.height / oImg.width
            left =this.canvas.width- (this.canvas.width - oImg.height * rate) / 2
          }
          else if(ang === 2){
rate = this.canvas.height / oImg.height
            left =this.canvas.width- (this.canvas.width - oImg.width * rate) / 2
            top= this.canvas.height
          }
          else if(ang === 3) {
            rate = this.canvas.height / oImg.width
            left =(this.canvas.width - oImg.height * rate) / 2
          }else if(ang === 4){
rate = this.canvas.height / oImg.height
            left =(this.canvas.width - oImg.width * rate) / 2
            top= 0
          }   

          debugger
          oImg.scaleToWidth(this.canvas.width)
          oImg.scaleToHeight(this.canvas.height)
          oImg.set({
            left,
            top,
          })
          console.log(oImg)
          this.canvas.add(oImg)
        },
        {
          crossOrigin: 'Anonymous',
          angle: that.angle,
        }
      )

      /*    fabric.Image.fromURL(
        this.imgSrc,
        (oImg) => {
          oImg.scaleToWidth(this.canvas.width)
          oImg.scaleToHeight(this.canvas.height)

          this.canvas.add(oImg)
        },
   ) */

      // console.log('left:'+left+',top:'+top+',angle:'+that.angle)

      that.angle += 90
      if (that.angle > 360) {
        that.angle = 90
      }
    },
    // 展示收藏/删除
    showCollDel() {
      this.collDel = true
    },
    // 收藏语音标题
    collectionSound() {
      this.collectionPopup = true
    },
    // 收藏语音确定
    handleCollection() {
      if (this.soundLabelTitle.length > 6) {
        return this.$message.success('不能超过6个字')
      } else {
        this.collectionPopup = false
        console.log('收藏标题---', this.soundLabelTitle)
      }
    },
    // 语音库
    showSoundBank() {
      this.soundBank = true
    },
    // 语音库
    handleSoundBank() {
      this.soundBank = false
    },
    // 发送语音
    sendSound() {},
    // 删除语音库
    deleteSound() {},
    // 语音泡
    hideCollDel() {
      this.collDel = false
    },
    // 获取评语库
    handleComment() {
      this.visible = true
      this.getComment()
    },
    getComment() {
      this.$http.post('/Comment/TeacherComment/GetComments', { teacherId: this.userId }).then((res) => {
        console.log('评语库---', res)
        this.comtentList = res.Data
      })
    },
    // 关闭评语库
    handleCommentOk() {
      this.flagAdd = false
    },
    // 编辑确认评语库
    addComtent() {
      this.flagAdd = true
      //  this.$http.post('/Comment/TeacherComment/AddComment', { teacherId: this.userId,comment: this.comtent }).then(res => {
      //   console.log('添加---', res)
      // })
    },
    // 添加确认
    handleAddComtent() {
      this.flagAdd = false
      console.log(this.addComtentTipc)
      this.$http
        .post('/Comment/TeacherComment/AddComment', { teacherId: this.userId, comment: this.addComtentTipc })
        .then((res) => {
          console.log('添加---', res)
          if (res.Success) {
            this.addComtentTipc = ''
            this.handleComment()
          }
        })
    },
    // getC
    // 添加取消
    handleCancelAdd() {
      this.flagAdd = false
    },
    // 编辑评语库
    editComtent(id, content) {
      console.log(id)
      console.log(content)
      this.$http
        .post('/Comment/TeacherComment/EditComment', { id, teacherId: this.userId, comment: content })
        .then((res) => {
          console.log('编辑---', res)
          if (res.Success) {
            this.editContent = ''
            this.handleComment()
            this.flag = true
          }
        })
    },
    // 编辑评语取消
    cancelEdit() {
      this.editContent = ''
      this.flag = true
      this.handleComment()
    },
    // 删除评语
    deleteComtent(id) {
      this.$http.post('/Comment/TeacherComment/DeleteComment', { teacherId: this.userId, id }).then((res) => {
        console.log('删除---', res)
      })
      this.handleComment()
    },
    // 选择评语
    handleChange () {
      console.log(this.myCommtent)
      // this.newMyCommtent = ''
      // this.arr = []
      this.evaluateArr.push(this.myCommtent)
      this.newMyCommtent = this.evaluateArr.join(',')
      console.log(this.evaluateArr.join(','))
    },
    // 录音
    soundRecording() {
      this.soundPopup = true
      this.stopItHide = true
      this.stopIt = false
    },
    // 显示停止/ 开始录音
    soundShow() {
      this.stopItHide = false
      this.stopIt = true
      recorder.start().then(
        () => {
          this.content = `${this.totalTime}s后停止录音`
          const clock = window.setInterval(() => {
            this.totalTime--
            this.content = `${this.totalTime}s后停止录音`
            if (this.totalTime === 10) {
              this.showCountDown = true
            }
            if (this.totalTime < 0) {
              window.clearInterval(clock)
              this.content = '已停止录音'
              this.showCountDown = false
              this.totalTime = 60
              this.hideSound()
            }
          }, 1000)
        },
        (error) => {
          // 出错了
          // console.log(`${error.name} : ${error.message}`)
          alert('无法启动音频源,请打开麦克风权限')
          this.soundPopup = false
          this.stopIt = false
        }
      )
    },

    // 停止录音
    hideSound() {
      this.soundPopup = false
      var newblob = recorder.getWAVBlob()

      var reader = new FileReader()

      const that = this

      reader.readAsDataURL(newblob)
      this.soundTime = parseInt(recorder.duration)

      recorder.onplayend = this.playend
      console.log(this.soundTime)
      if (this.soundTime < 1) {
        this.$message.error('说话时间太短')
      } else {
        reader.onload = function (e) {
          that.$uwonhttp
            .post('/ExamItem/ExamItem/AddItemMp3Base64', {
              studentId: that.active,
              itemId: that.subjectItemId,
              mp3Name: '1',
              mp3Location: '1',
              teacherId: that.userId,
              paperId: that.paperId,
              base64: e.target.result,
            })
            .then((res) => {
              console.log('luyin-------', e.target.result)
            })
        }
      }
    },
    playend() {
      console.log('播放完成')
      this.activePlay = ''
    },
    // 播放录音
    playSound() {
      console.log('11111')
      this.activePlay = '1'
      recorder.play()
    },
    // 下载图片
    saveImg(isLast) {
      var type = 'png'
      console.log(this.canvas)
      // 返回一个包含JPG图片的<img>元素
      // var img_png_src = this.canvas.toDataURL('image/png') // 将画板保存为图片格式的函数
      // var imgData = img_png_src.replace(
      //   this._fixType(type),
      //   'image/octet-stream'
      // )

      // var filename = '创伤部位_' + new Date().getTime() + '.' + type

      var canvas = document.getElementById('c')

      var imgPngSrc = canvas.toDataURL('image/png', 1)
      // console.log('=============', imgPngSrc)// 将画板保存为图片格式的函数
      // const sd = img_png_src.indexOf(',')

      // console.log(this.active)
      if (this.active) {
        debugger
        this.CorrectionProgress = '1'
        this.$uwonhttp
          .post('/ExamItem/ExamItem/UploadTeacherPicBase64', {
            itemId: this.subjectItemId,
            studentId: this.active,
            paperId: this.paperId,
            base64: imgPngSrc,
          })
          .then((res) => {
            console.log('保存图片---', res)

            if (res.data.Success) {
              this.saveReviewRelationship()
            }
            if (res.data.Success && isLast) {
              // debugger
              this.saveReviewRelationship()
              this.$message.success('辛苦啦，批改完毕啦~')
              setTimeout(() => {
                this.$router.go(-1)
              }, 800)
            }
          })
      }
      // var imgData = img_png_src.replace(this._fixType(type))

      // var filename = '创伤部位_' + new Date().getTime() + '.' + type
      // this.saveFile(imgData, filename)
    },
    // 是否推荐
    whetherPush(e) {
      console.log('是否推荐', e.target.value)
      this.WhetherPush = e.target.value
    },
    // 选择推荐
    onChange(value) {
      console.log(' 选择推荐', value)
    },
    // 保存文件
    // saveFile (data, filename) {
    //   var save_link = document.createElementNS('http://www.w3.org/1999/xhtml', 'a')
    //   save_link.href = data // data is the value of created png
    //   console.log('11----------', save_link.href)
    //   save_link.download = filename
    //   var event = document.createEvent('MouseEvents')
    //   event.initMouseEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null)
    //   save_link.dispatchEvent(event)
    // },
    // _fixType (type) {
    //   type = type.toLowerCase().replace(/jpg/i, 'jpeg')
    //   var r = type.match(/png|jpeg|bmp|gif/)[0]
    //   return 'image/' + r
    // },
    // 初始化画板
    initCanvas(isSaveImg) {
      var self = this

      // 初始化画板
      // console.log(this.canvas)
      if (this.canvas == null) {
        this.canvas = new fabric.Canvas('c', {
          isDrawingMode: true,
          skipTargetFind: true,
          selectable: false,
          selection: false,
        })
      } else {
        if (isSaveImg) {
          this.saveImg()
        }

        this.canvas.clear()
      }
      // console.log('-------------', this.canvas)
      fabric.Image.fromURL(
        this.imgSrc,
        (oImg) => {
          oImg.scaleToWidth(this.canvas.width)
          oImg.scaleToHeight(this.canvas.height)
          let rate = this.canvas.height / oImg.height
          let left =(this.canvas.width - oImg.width * rate) / 2
          let top= 0
          oImg.set({
            left,
            top
          })
          this.canvas.add(oImg)
        },
        {
          crossOrigin: 'Anonymous',
        }
      )
      window.zoom = window.zoom ? window.zoom : 1

      this.canvas.freeDrawingBrush.color = this.color // 设置自由绘颜色
      this.canvas.freeDrawingBrush.width = this.drawWidth

      this.canvas.off('mouse:down')
      // 绑定画板事件
      this.canvas.on('mouse:down', (options) => {
        var xy = this.transformMouse(options.e.offsetX, options.e.offsetY)
        this.mouseFrom.x = xy.x
        this.mouseFrom.y = xy.y
        this.doDrawing = true
      })
      this.canvas.off('mouse:up')
      this.canvas.on('mouse:up', (options) => {
        var xy = this.transformMouse(options.e.offsetX, options.e.offsetY)
        this.mouseTo.x = xy.x
        this.mouseTo.y = xy.y
        this.drawingObject = null
        this.moveCount = 1
        this.doDrawing = false
        // 由于在箭头等非自由曲线描述时，属于连续触发
        this.updateCanvasState()
      })
      this.canvas.off('mouse:move')
      this.canvas.on('mouse:move', (options) => {
        if (this.moveCount % 2 && !this.doDrawing) {
          // 减少绘制频率
          return
        }
        this.moveCount++
        var xy = this.transformMouse(options.e.offsetX, options.e.offsetY)
        this.mouseTo.x = xy.x
        this.mouseTo.y = xy.y
        this.drawimg()
      })

      this.canvas.off('selection:created')
      this.canvas.on('selection:created', (e) => {
        if (e.target._objects) {
          // 多选删除
          var etCount = e.target._objects.length
          for (var etindex = 0; etindex < etCount; etindex++) {
            this.canvas.remove(e.target._objects[etindex])
          }
        } else {
          // 单选删除
          this.canvas.remove(e.target)
        }
        this.canvas.discardActiveObject() // 清除选中框
      })
      this.canvas.off('object:added')
      this.canvas.on('object:added', function () {
        // 由于初始化时，添加了默认背景图片，只需一次
        if (self.isFirst) {
          self.updateCanvasState()
          self.isFirst = false
        }
      })

      this.canvas.renderAll()
    },
    // 坐标转换
    transformMouse(mouseX, mouseY) {
      return { x: mouseX / window.zoom, y: mouseY / window.zoom }
    },
    drawimg() {
      if (this.drawingObject) {
        this.canvas.remove(this.drawingObject)
      }
      var canvasObject = null
      switch (this.drawType) {
        case 'arrow': // 箭头
          canvasObject = new fabric.Path(this.drawArrow(30, 30), {
            stroke: this.color,
            fill: 'rgba(255,255,255,0)',
            strokeWidth: this.drawWidth,
          })
          break
        case 'line': // 直线
          canvasObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
            stroke: this.color,
            strokeWidth: this.drawWidth,
          })
          break
        case 'dottedline': // 虚线
          canvasObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
            strokeDashArray: [3, 1],
            stroke: this.color,
            strokeWidth: this.drawWidth,
          })
          break
        case 'circle': // 正圆
          var left = this.mouseFrom.x
          var top = this.mouseFrom.y
          var radius =
            Math.sqrt(
              (this.mouseTo.x - left) * (this.mouseTo.x - left) + (this.mouseTo.y - top) * (this.mouseTo.y - top)
            ) / 2
          canvasObject = new fabric.Circle({
            left: left,
            top: top,
            stroke: this.color,
            fill: 'rgba(255, 255, 255, 0)',
            radius: radius,
            strokeWidth: this.drawWidth,
          })
          break
        case 'ellipse': // 椭圆
          var left = this.mouseFrom.x
          var top = this.mouseFrom.y
          canvasObject = new fabric.Ellipse({
            left: left,
            top: top,
            stroke: this.color,
            fill: 'rgba(255, 255, 255, 0)',
            originX: 'center',
            originY: 'center',
            rx: Math.abs(left - this.mouseTo.x),
            ry: Math.abs(top - this.mouseTo.y),
            strokeWidth: this.drawWidth,
          })
          break
        case 'square': // TODO:正方形（后期完善）
          break
        case 'rectangle': // 长方形
          var path =
            'M ' +
            this.mouseFrom.x +
            ' ' +
            this.mouseFrom.y +
            ' L ' +
            this.mouseTo.x +
            ' ' +
            this.mouseFrom.y +
            ' L ' +
            this.mouseTo.x +
            ' ' +
            this.mouseTo.y +
            ' L ' +
            this.mouseFrom.x +
            ' ' +
            this.mouseTo.y +
            ' L ' +
            this.mouseFrom.x +
            ' ' +
            this.mouseFrom.y +
            ' z'
          canvasObject = new fabric.Path(path, {
            left: left,
            top: top,
            stroke: this.color,
            strokeWidth: this.drawWidth,
            fill: 'rgba(255, 255, 255, 0)',
          })
          // 也可以使用fabric.Rect
          break
        case 'rightangle': // 直角三角形
          var path =
            'M ' +
            this.mouseFrom.x +
            ' ' +
            this.mouseFrom.y +
            ' L ' +
            this.mouseFrom.x +
            ' ' +
            this.mouseTo.y +
            ' L ' +
            this.mouseTo.x +
            ' ' +
            mouseTo.y +
            ' z'
          canvasObject = new fabric.Path(path, {
            left: left,
            top: top,
            stroke: this.color,
            strokeWidth: this.drawWidth,
            fill: 'rgba(255, 255, 255, 0)',
          })
          break
        case 'equilateral': // 等边三角形
          var height = this.mouseTo.y - this.mouseFrom.y
          canvasObject = new fabric.Triangle({
            top: this.mouseFrom.y,
            left: this.mouseFrom.x,
            width: Math.sqrt(Math.pow(height, 2) + Math.pow(height / 2.0, 2)),
            height: height,
            stroke: this.color,
            strokeWidth: this.drawWidth,
            fill: 'rgba(255,255,255,0)',
          })
          break
        case 'isosceles':
          break
        case 'text':
          this.textbox = new fabric.Textbox('', {
            left: this.mouseFrom.x,
            top: this.mouseFrom.y - 10,
            width: 120,
            fontSize: 18,
            borderColor: '#2c2c2c',
            fill: this.color,
            hasControls: false,
          })

          this.canvas.add(this.textbox)
          this.textbox.enterEditing()
          this.textbox.hiddenTextarea.focus()
          break
        case 'remove':
          break
        default:
          break
      }
      if (canvasObject) {
        this.canvas.add(canvasObject) // .setActiveObject(canvasObject)
        this.drawingObject = canvasObject
      }
    },
    undo() {
      const self = this
      if (this.config.undoFinishedStatus) {
        if (this.config.canvasState.length > 1) {
          if (this.config.currentStateIndex > 0) {
            this.canvas.loadFromJSON(this.config.canvasState[this.config.currentStateIndex - 1], function () {
              self.canvas.renderAll()
              self.config.currentStateIndex -= 1
              if (self.config.currentStateIndex <= 0) {
                self.config.undoStatus = false
              }
            })
          }
        }
      }
    },
    updateCanvasState() {
      var jsonData = this.canvas.toJSON()
      var canvasAsJson = JSON.stringify(jsonData)
      if (this.config.currentStateIndex < this.config.canvasState.length - 1) {
        var indexToBeInserted = this.config.currentStateIndex + 1
        this.config.canvasState[indexToBeInserted] = canvasAsJson
        var numberOfElementsToRetain = indexToBeInserted + 1
        this.config.canvasState = this.config.canvasState.splice(0, numberOfElementsToRetain)
      } else {
        this.config.canvasState.push(canvasAsJson)
      }

      this.config.currentStateIndex = this.config.canvasState.length - 1
      // console.log(this.config.currentStateIndex)
      if (this.config.currentStateIndex == this.config.canvasState.length - 1 && this.config.currentStateIndex > 0) {
        this.config.undoStatus = true
      }
    },
    drawArrow(theta, headlen) {
      const fromX = this.mouseFrom.x
      const fromY = this.mouseFrom.y
      const toX = this.mouseTo.x
      const toY = this.mouseTo.y
      theta = typeof theta !== 'undefined' ? theta : 30
      headlen = typeof theta !== 'undefined' ? headlen : 10
      // 计算各角度和对应的P2,P3坐标
      var angle = (Math.atan2(fromY - toY, fromX - toX) * 180) / Math.PI
      var angle1 = ((angle + theta) * Math.PI) / 180
      var angle2 = ((angle - theta) * Math.PI) / 180
      var topX = headlen * Math.cos(angle1)
      var topY = headlen * Math.sin(angle1)
      var botX = headlen * Math.cos(angle2)
      var botY = headlen * Math.sin(angle2)
      var arrowX = fromX - topX
      var arrowY = fromY - topY
      var path = ' M ' + fromX + ' ' + fromY
      path += ' L ' + toX + ' ' + toY
      arrowX = toX + topX
      arrowY = toY + topY
      path += ' M ' + arrowX + ' ' + arrowY
      path += ' L ' + toX + ' ' + toY
      arrowX = toX + botX
      arrowY = toY + botY
      path += ' L ' + arrowX + ' ' + arrowY
      return path
    },
    changeEditMode(mode) {
      this.drawType = mode
      this.canvas.isDrawingMode = false
      if (this.textbox) {
        // 退出文本编辑状态
        this.textbox.exitEditing()
        this.textbox = null
      }
      if (mode == 'pen') {
        this.canvas.isDrawingMode = true
      } else if (mode == 'remove') {
        this.canvas.selection = true
        this.canvas.skipTargetFind = false
        this.canvas.selectable = true
      } else {
        this.canvas.skipTargetFind = true // 画板元素不能被选中
        this.canvas.selection = false // 画板不显示选中
      }
    },
  },
}
</script>

<style lang="less" scoped>
.swiper-button-next {
  right: 20px;
  // transform: rotate(90deg);
}
// canvas适配
.canvas-img .lower-canvas {
  width: 767px !important;
}
// 语音库操作
.recording-library {
  display: inline-block;
  padding: 0 8px;
  cursor: pointer;
}
// 语音库编辑
.coll-popup-input {
  width: 300px;
  height: 32px;
  border: 1px solid #ccc;
  border-radius: 5px;
  text-indent: 10px;
}
.swiper-button-prev {
  left: 20px;
  // transform: rotate(90deg);
}
/deep/.swiper-slide .swiper-slide-active {
  width: 80px;
  margin-right: 30px;
  text-align: center;
}
.recommend-data {
  position: relative;
  right: -282px;
  text-align: right;
}
@keyframes run {
  0% {
    background: url('../../../../assets/correcting/录音动画-无黑框_00000.png');
  }
  25% {
    background: url('../../../../assets/correcting/录音动画-无黑框_00011.png');
  }
  50% {
    background: url('../../../../assets/correcting/录音动画-无黑框_00017.png');
  }
  75% {
    background: url('../../../../assets/correcting/录音动画-无黑框_00028.png');
  }
  100% {
    background: url('../../../../assets/correcting/录音动画-无黑框_00059.png');
  }
}
// 点击停止
.click-stop {
  width: 206px;
  height: 184px;
  margin-top: -103px;
  background: url('../../../../assets/correcting/录音动画-无黑框_00000.png') #ddd;
  background-color: #ddd;
  // animation:run 1s steps(1, start) infinite;
  animation: run 2s linear infinite;
}
// 正确图标
.correct-icon {
  position: absolute;
  left: 50%;
  top: 200px;
  width: 80px;
  z-index: 2;
}
// 切换题目
.prev-img {
  position: absolute;
  left: 200px;
  top: 230px;
  z-index: 2;
}
// 切换下一题
.next-img {
  position: absolute;
  right: 200px;
  top: 230px;
  z-index: 2;
}
// 推荐图
.recommend-img {
  margin: 0;
}

// 提交
.submit-correcting {
  display: inline-block;
  width: 80px;
  height: 35px;
  line-height: 35px;
  color: #fff;
  background-color: #68bb97;
  border-radius: 5px;
  cursor: pointer;
}
.swiper-slide {
  width: 95px !important;
}
.left-arrow {
  position: absolute;
  left: -74px;
  top: 8px;
}
.right-arrow {
  position: absolute;
  right: -70px;
  top: 13px;
}
.mr {
  margin-right: 15px;
}
.ant-progress-bg {
  background-color: #68bb97;
}
// 学生信息
.student-info {
  overflow: hidden;
  width: 80%;
  // height: 60px;
  margin: 0 auto;
  margin-bottom: 16px;
  background-color: #fff;
  border-radius: 5px;
}

.li-div {
  padding-top: 6px;
  padding-bottom: 1px;
  background-color: #ecf5f3;
  border-radius: 5px;
  cursor: pointer;
}
// 学生详情
.module-list-wrap {
  padding: 30px 80px;
  width: 100%;
  .arrow-img {
    margin: 0 20px;
    margin-top: 23px;
  }
  .module-list {
    li {
      width: 80px;
      margin-right: 16px;
      p {
        font-size: 12px;
        margin-bottom: 3px;
      }

      .complete-img {
        margin-top: 5px;
        text-align: center;
      }
    }
  }
}
@keyframes soundShow {
  0% {
    background: url('../../../../assets/correcting/播放1.png');
  }
  25% {
    background: url('../../../../assets/correcting/播放2.png');
  }
  50% {
    background: url('../../../../assets/correcting/播放3.png');
  }
  100% {
    background: url('../../../../assets/correcting/播放3.png');
  }
}
.active {
  animation: soundShow 2s linear infinite;
}
.sound-show {
  display: inline-block;
  width: 201px;
  height: 39px;
  line-height: 39px;
  text-align: center;
  background: url('../../../../assets/correcting/录音show.png');
  cursor: pointer;
}
.text-area {
  border: 1px solid #ccc;
  width: 600px;
  height: 147px;
  border-radius: 5px;
  margin-top: 15px;
  resize: none;
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
  cursor: pointer;
}
.text-area {
  border: 1px solid #ccc;
  width: 600px;
  height: 147px;
  border-radius: 5px;
  margin-top: 15px;
  resize: none;
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
// 评语显示
.comment-display {
  height: 50px;
  line-height: 50px;
  margin: 0px auto;
  margin-top: 18px;
  text-indent: 15px;
  background-color: #d8d8d8;
  border-radius: 5px;
}
.comtent {
  // input编辑
  .input-comtent {
    input {
      width: 300px;
      height: 32px;
      border: 1px solid #ccc;
      border-radius: 5px;
      text-indent: 10px;
    }
    span {
      margin-left: 10px;
      cursor: pointer;
    }
  }
}
// 批改主要内容
.correcting-infor {
  position: relative;
  // display: flex;
  // justify-content: center;
  .canvas-btn {
    position: absolute;
    right: 4%;
    margin-left: 24px;
    padding-top: 45px;
    div {
      margin-bottom: 30px;
    }
    p {
      margin: 0;
      text-align: center;
      cursor: pointer;
    }
  }
}
// 批改画布
.correcting-canvas {
  position: absolute;
  left: 50%;
  width: 80%;
  background-color: #fff;
  border-radius: 5px;
  transform: translateX(-50%);
  .canvas-img {
    width: 100% !important;
    height: 100% !important;
  }
}
// 画布按钮
.app-btn {
  position: absolute;
  width: 100%;
  // text-align: center;
  span {
    display: inline-block;
    text-align: center;
    margin-top: 10px;
    margin-right: 20px;
    cursor: pointer;
    p {
      margin-bottom: 0;
    }
  }
}
// 画布内容
/deep/.canvas-container {
  width: 800px;
  margin: 0 auto;
}
// 题的操作
.subject-btn {
  input {
    width: 50px;
    border: 1px solid #ccc;
    border-radius: 5px;
    /* text-indent: 10px; */
    text-align: center;
  }
  p {
    text-align: center;
  }
  .bys {
    background-color: #68bb97;
    border-radius: 5px;
    color: #fff;
  }
  .pad {
    padding: 1px 6px;
  }
  .ad {
    padding: 4px 14px;
  }
}
.col {
  border: 1px solid #0cfffd;
  border-radius: 5px;
}
.title {
  font-size: 30px;
  font-weight: 900;
  text-align: center;
}
// 进度条
.ant-progress-line {
  width: 200px;
}
.ant-progress-inner {
  background-color: #ddd;
}
// 图标
.icon {
  display: inline-block;
  padding: 0 3px;
  color: #68bb97;
  border: 1px solid #68bb97;
  border-radius: 6px;
}
// 试卷信息
.paper-infor {
  text-align: center;
  margin-top: 20px;
  span {
    margin-right: 30px;
  }
}
// 题目信息
.sub-title {
  width: 80%;
  margin: 0 auto;
  text-indent: 10px;
  margin-bottom: 7px;
  border-radius: 5px;
  padding: 10px 5px;
  background-color: #eeeeee;
}
</style>
