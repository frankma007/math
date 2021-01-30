<template>
  <div>
    <p class="m-b">
      <span class="m-r" style="font-size: 16px;color:#000;">高频错题</span>
      <span @click="setUpAccuracy" style="color:#68BB97;font-size:16px;" class="m-r cur"><img src="@/assets/high/setUp.png" alt="">正确率设置</span>
      <!-- 设置正确率 -->
      <a-modal
        title="设置高频错题范围"
        :visible="visible"
        :width="650"
        :bodyStyle="{textAlign:'center'}"
        centered
        @ok="handleOk"
        @cancel="handleCancel"
      >
        <div class="teacher-popup">
          <P class="popup-p" v-for="item in classList" :key="item.ClassId"> {{ item.ClassName }} 低于 <input @blur="handleInput(item.ClassId,item.Accuracy)" class="popup-input" type="text" v-model="item.Accuracy">% (不含)</P>
          <!-- <P class="popup-p">三年级&四年级平均正确率 低于 <input class="popup-input" type="" v-model="TFAccuracy">(不含)</P> -->
          <!-- <P>五年级平均正确率 低于 <input class="popup-input" type="text" v-model="FAccuracy">(不含)</P> -->
        </div>
      </a-modal>
      <span @click="editPush" class="edit-btn m-r cur fg"><img src="@/assets/high/编组 (2).png" alt="">{{ editeBtn }}</span>
      <span @click="editHistory" class="edit-btn-history m-r cur fg"><img src="@/assets/lack/editHis.png" alt="">历史编辑</span>
    </p>
    <p class="select-info" v-if="this.loadChapter">
      <span class="m-r">
        <span class="f-s">时间</span>&nbsp;
        <a-select :defaultValue="defaultValue" style="width: 120px" @change="handleTime">
          <a-select-option value="week">本周</a-select-option>
          <a-select-option value="month">本月</a-select-option>
          <!-- <a-select-option value="xq">本学期</a-select-option> -->
        </a-select>
      </span>
      <span>
        <span class="f-s">班级</span>&nbsp;
        <span class="m-r">
          <a-select :defaultValue="defaultValueClass" style="width: 120px" v-if="defaultValueClass" @change="handleClass">
            <a-select-option :value="item.ClassId" v-for="item in classList" :key="item.ClassId">{{ item.ClassName }}</a-select-option>
            <!-- <a-select-option :value="0">全区练习</a-select-option> -->
          </a-select>
        </span>
        <span class="m-r" >
          <!-- <a-select style="width: 220px;" @change="handleChapter" placeholder="选择章节">
              <a-select-option :value="item.ChapterId" v-for="item in chapterList" :key="item.ChapterId">{{ item.ChapterName }}</a-select-option>
            </a-select> -->
          <!-- 联级默认 -->
          <!-- :defaultValue="defaultValueChapterDe" -->
          <a-cascader
            :field-names="{ label: 'ChapterName', value: 'ChapterId', children: 'Second' }"
            :options="chapterList"
            style="width: 220px;"
            change-on-select
            @change="handleChapter"
            @popupVisibleChange="handleChapterPopup"
            placeholder="选择章节"
          />
        </span>
      </span>
      <span class="f-s">题目</span>&nbsp;
      <span class="m-r">
        <a-select @change="handleArea" :defaultValue="defaultValueArea" style="width: 120px" >
          <a-select-option :value="0">区级题</a-select-option>
          <a-select-option :value="1">校级题</a-select-option>
          <a-select-option :value="2">班级题</a-select-option>
        </a-select>
      </span>
      </span>
    </p>
    <!-- 加载。。。 -->
    <div v-if="loadSubjectList" style="text-align: center;">
      <a-spin />
    </div>
    <div class="bottom" v-show="!showLack">
      <!-- <p class="chapterTitle">{{ titles }}</p> -->
      <p class="chapter-title">共{{ allSubjectNum }}道高频错题</p>
      <!-- 题目的正确率 -->
      <div class="showProcress">
        <div class="process" v-for="(item,index) in allDatas" :key="index">
          <div class="proce">
            <a-progress type="circle" :percent="item.ClassPer" :width="50" :format="(percent) =>{if(percent=='100'){return '100%'}else{return percent+'%'}}"/>
          </div>
          <div class="number">
            {{ index+1 }}
          </div>
        </div>
      </div>

    </div>
    <div class="showQuestion">
      <div v-show="showLack" style="text-align:center;position: relative;top: 25%;">
        <img src="@/assets/lack/错题收纳为空.png" alt="">
        <p style="color:#ccc">大家都很棒，当前暂无高频错题哦~</p>
      </div>
      <div class="type" v-for="(item,index) in allDatas" :key="index">
        <div class="question">
          <img v-show="selectTest" @click="selectPaper(item.ItemId,$event)" class="img fg cur " :src="checkIcon" alt="">
          {{ index+1+'.' }}
          <div class="anserTitle" v-html="item.ItemTitle"></div>
          <div class="mar">
            <!-- 选项内容 -->
            <span v-for="(ite,ind) in item.Opts" :key="ind" v-show="item.ItemTypeId !== 5">
              <span v-if="item.ItemType === 2 || item.ItemType === 10">{{ ite.Opt }}.</span>
              <span v-html="ite.OptContent" style="margin-right: 25px"></span>
            </span>

          </div>
        </div>

        <div class="answer">
          <p v-show="item.IsCoach === 0" @click="toWrongCoach(item.ItemId, item.ChapterId,item.Grade)" class="fg wrong-coach cur">错题辅导</p>
          <p v-show="item.IsCoach !== 0" class="fg  cur"><img src="@/assets/correcting/giveUp.png" alt=""><span class="already-coach">已做辅导</span></p>
          <p class="anserTitle"><span>【答案:】 </span><span v-html="item.Answer"></span></p>
          <br>
          <p class="anserTitle "><span>【解析:】 </span><span v-html="item.Analyze"></span></p>
          <!-- 表格 -->
          <div class="accuracy">
            <a-table
              :rowKey="(record,index)=>{return index}"
              :columns="columns"
              :data-source="item.Per"
              bordered
              :pagination="pagination">
            </a-table>
          </div>
          <!-- <p class="fg">
            <span @click="editSubject(item.ItemId)" class="edit cur">编辑</span>
            <span class="wrong-coach cur">发布</span>
          </p> -->
          <!-- 题目的编辑 -->

          <!-- <a-drawer
            placement="right"
            title="Basic Drawer"
            :closable="false"
            :visible="MakeTopic"
            @close="onCloseMakeTopic"
          > -->
          <!-- <span v-if="MakeTopic && this.makeModal === item.ItemId">
            <a-drawer
              title=""
              placement="right"
              :visible="MakeTopic"
              @close="onCloseMakeTopic"
              width="60%"
              height="100%"
              :drawerStyle="{ backgroundColor: '#FAF7F8'}"
              :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.1)'}"
              destroyOnClose
            >
              <p>题目编辑</p>
            </a-drawer>
          </span> -->
          <p class="anserTitle ana-bor-up" >
            正确<span v-html="item.ClassRight"></span>人
            <span class="progressSpan"><a-progress :success-percent="item.ClassRight/(item.ClassRight+item.ClassError)*100" :showInfo="false" :strokeWidth="15" /></span>
            错误<span v-html="item.ClassError"></span>人&nbsp;&nbsp;&nbsp;&nbsp;
            <span v-if="item.ClassRight !== 0 || item.ClassError !== 0" class="packInfo" @click="questionDetail(item.ItemId,index,$event)">展开详情↓</span>&nbsp;
          </p>
          <p class="clearfix">
            <span v-show="item.IsOnce === 'yes'" class="fl quote-arrangement">曾布置过</span>
          </p>
          <div v-if="loadCoach && showAloneId === item.ItemId" style="text-align: center;">
            <a-spin />
          </div>
          <!--  -->
          <div v-if="!loadCoach" class="answerDetail" v-show="showCoach && showAloneId === item.ItemId">
            <!-- 题目作答详情 -->
            <div class="answer-details" v-for="(ite, index) in coachList" :key="index" >
              <p class="m-b" v-show="ite.IsRight === 1">
                <span class="anw-p1">{{ ite.Answer }}</span>
                <span class="progressSpan"><a-progress :success-percent="(ite.Count/(item.ClassRight+item.ClassError))*100" :showInfo="false" :strokeWidth="15" /></span>
                <span>正确 &nbsp;{{ ite.Count }}人</span>
              </p>
              <p class="answer-correct m-b" v-show="ite.IsRight === 0">
                <span class="anw-p1">{{ ite.Answer }}</span>
                <span class="progressSpan"><a-progress :success-percent="(ite.Count/(item.ClassRight+item.ClassError))*100" :showInfo="false" :strokeWidth="15" /></span>
                <span>错误 &nbsp;{{ ite.Count }}人</span>
              </p>
            </div>

          </div>
          <!-- 辅导讲解 -->
          <div v-if="!loadCoach && coachData.IsHaveCoach !== 0" class="coach-explain" v-show="showCoach && showAloneId === item.ItemId">
            <p style="font-size: 18px;">辅导讲解</p>
            <p style="color:#B5B5B5;font-size:16px;">{{ coachData.CoachCruxText }}</p>
            <p class="img-p">
              <img
                v-for="(item,index) in coachData.CoachPictures"
                :key="index"
                class="change-big"
                style="width: 200px;"
                :src="item"
                alt=""
                :preview="index">
              <!-- <img style="width: 200px;" src="@/assets/student/welcome-banner.png" alt="" preview="2"> -->
            </p>
            <p>

            </p>
          </div>
          <!-- 语音 -->
          <div
            v-if="!loadCoach && coachData.IsHaveCoach !== 0 && showCoach && showAloneId === item.ItemId && coachData.CoachLanguage !== ''"
            class="play-adiuo"
            ref="playAdiuo"
            @click="playAdiuoed"
            :class="{ 'active': activePlay === '1' }">
            <audio ref="mp3" @ended="overAudio" :src="coachData.CoachLanguage">
              <!-- <source src="" type="audio/ogg"> -->
              <!-- <source src="horse.mp3" type="audio/mpeg"> -->
              <!-- <source src=""> -->
              您的浏览器不支持 audio 元素。
            </audio>
            <span style="color: #fff">{{ playTime }}</span>
          </div>
          <!-- 视频 -->
          <div v-if="!loadCoach && coachData.IsHaveCoach !== 0 && showCoach && showAloneId === item.ItemId && coachData.CoachVideo !== ''" class="coach-video">
            <video-player
              class="video-player vjs-custom-skin"
              ref="videoPlayer"
              :playsinline="true"
              :options="playerOptions">
            </video-player>
          </div>
        </div>
        <br>
        <br>
      </div>
    </div>
    <!-- 分页 -->
    <div class="paging">
      <a-pagination :default-current="pageNo" :total="this.pageTotal" :defaultPageSize="15" @change="onPaging" hideOnSinglePage />
    </div>
  </div>
</template>

<script>
import Recorder from 'js-audio-recorder'
const recorder = new Recorder()
const columns = [
  {
    title: '-----',
    dataIndex: 'average',
    key: 'average',
    align: 'center'
  },
  {
    title: '我班',
    dataIndex: 'ClassPer',
    key: 'ClassPer',
    align: 'center'
  },
  {
    title: '我校',
    dataIndex: 'SchoolPer',
    key: 'SchoolPer',
    align: 'center'
  },
  {
    title: '全区',
    dataIndex: 'AllPer',
    key: 'AllPer',
    align: 'center'
  }
]
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    // this.showAccuracy()
    // 老师所带班级
    this.getTeacherClass()
    // setTimeout(() => {
    //   this.playSound()
    // }, 8800)
  },
  watch: {
    $route () {
      if (this.$route.fullPath === '/Teacher/My_Task/HighWrong/HighWrong') {
        // this.getTeacherClass()
        this.GetHighFrequencyList()
      }
    }
  },
  data () {
    return {
      // 主页缺省
      showLack: false,
      playerOptions: {
        playbackRates: '', // 播放速度'[0.5, 1, 1.5, 2]'
        autoplay: false, // 如果true,浏览器准备好时开始回放。
        controls: true, // 控制条
        preload: 'auto', // 视频预加载
        muted: false, // 默认情况下将会消除任何音频。
        loop: false, // 导致视频一结束就重新开始。
        language: 'zh-CN',
        aspectRatio: '16:9', // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
        fluid: true, // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
        sources: [{
          // type: 'video/mp4',
          // type: 'video/ogg',
          type: 'video/webm',
          src: ''// 你所放置的视频的地址，最好是放在服务器上
        }],
        poster: '', // 你的封面地址（覆盖在视频上面的图片）
        width: document.documentElement.clientWidth,
        notSupportedMessage: '此视频暂无法播放，请稍后再试' // 允许覆盖Video.js无法播放媒体源时显示的默认信息。
      },
      // 加载辅导
      loadCoach: false,
      // 显示辅导的内容
      showCoach: false,
      // 答案分布详情
      coachList: [],
      coachData: {},
      showAloneId: '',
      // 加载
      loadSubjectList: true,
      // 总题数
      allSubjectNum: 0,
      // 分页
      pageNo: 1,
      // 试题总数
      pageTotal: 0,
      loadChapter: false,
      replaceFields: {
        children: 'Second',
        title: 'ChapterName',
        key: 'ChapterId',
        value: 'ChapterId'
      },
      checkIcon: require('@/assets/teacher/new-sel.png'),
      userId: '',
      // 教师所带班级
      classList: [],
      // 修改班级正确率
      classAccuracy: [],
      // 区/班/校
      paperType: 0,
      // 编辑弹窗
      makeModal: '',
      // 章节列表
      chapterList: [],
      // 编辑按钮
      editeBtn: '编辑推送',
      // 设置弹出 / 首次进入页面
      visible: false,
      columns,
      defaultValue: 'week',
      // 默认班级
      defaultValueClass: '',
      // 默认章节
      defaultValueChapter: '',
      // 默认地区
      defaultValueArea: 0,

      load: false,
      allDatas: '',
      isShowAnswer: false,
      allAnswerDetail: [],
      countNum: [],
      // 表格去除分页
      pagination: false,
      activePlay: '',
      // 语音时间
      playTime: 1,
      // 题目编辑
      MakeTopic: false,
      chapterAll: [],
      defaultValueChapterDe: [],
      // 单选题目
      arr: [],
      // 选中的题目
      arr1: '',
      // 显示单选题目
      selectTest: false,
      paperId: ''
    }
  },
  methods: {
    // 获取老师所带班级 / 返回的正确率
    getTeacherClass () {
      this.$http.post('/HighFrequency/HighFrequency/GetClassAccuracy', { userId: this.userId }).then(res => {
        console.log('教师所带班级正确率', res)
        // res.Data[1].Accuracy = 0
        const judge = res.Data.some(value => {
          return value.Accuracy === 0
        })
        if (judge) {
          this.visible = true
        } else {
          this.visible = false
        }
        this.classList = res.Data
        this.defaultValueClass = res.Data[0].ClassId
        this.getChapterList()

        this.GetHighFrequencyList()
      })
    },
    // 设置正确率
    setUpAccuracy () {
      this.visible = true
    },
    handleInput (itemId, id) {
      console.log('ITEMID / 真确率', itemId, id)
      // debugger
      const arr = this.classAccuracy.some(value => {
        return value.ClassId === itemId
      })
      // debugger

      if (!arr) {
        this.classAccuracy.push(
          {
            'ClassId': itemId,
            'Accuracy': +id
          }
        )
      } else {
        this.classAccuracy.forEach(value => {
          // debugger
          if (value.ClassId === itemId) {
            const idx = this.classAccuracy.indexOf(value)
            this.classAccuracy.splice(idx, 1)
          }
        })
        this.classAccuracy.push(
          {
            'ClassId': itemId,
            'Accuracy': +id
          }
        )
      }
      console.log('正确率的传参', this.classAccuracy)
    },
    // 设置确定
    handleOk () {
      // 设置正确率
      this.$http.post('/HighFrequency/HighFrequency/SetClassAccuracy', { jsonSo: JSON.stringify(this.classAccuracy) }).then(res => {
        console.log('正确率设置', res)
        this.visible = false
        this.loadSubjectList = true
        this.GetHighFrequencyList()
      })
      //
    },
    // 设置取消
    handleCancel () {
      this.visible = false
    },
    // 处理章节
    handleChapter (value) {
      this.chapterAll = value
      if (value.length === 1) {
        return console.log('hhaha')
      } else {
        this.defaultValueChapter = value[1]
        this.GetHighFrequencyList()
      }
      console.log('章节选择', value)
      // this.GetHighFrequencyList()
    },
    handleChapterPopup (value) {
      // console.log('哈哈哈哈', value)
      if (!value && this.chapterAll.length === 1) {
        this.$message.warning('请选择小章节')
      }
    },
    // 处理时间
    handleTime (value) {
      this.loadSubjectList = true
      this.defaultValue = value
      this.GetHighFrequencyList()
    },
    // 处理班级
    handleClass (value) {
      console.log('班级id', value)
      this.loadSubjectList = true
      this.defaultValueClass = value
      this.getChapterList()
      // this.GetHighFrequencyList()
    },
    // 处理地区
    handleArea (value) {
      console.log(value)
      this.loadSubjectList = true
      this.paperType = value
      this.GetHighFrequencyList()
    },
    // 获取高频错题列表
    GetHighFrequencyList () {
      this.$http.post('/HighFrequency/HighFrequency/GetHighFrequencyList', { userId: this.userId, classId: this.defaultValueClass, chapterId: this.defaultValueChapter, paperType: this.paperType, whereTime: this.defaultValue, pageNo: this.pageNo }).then(res => {
        console.log('高频错题', res)
        this.allDatas = res.Data
        if (res.Data.length !== 0) {
          this.pageTotal = res.Data[0].TotalCount
          this.allSubjectNum = res.Data[0].TotalCount
          res.Data.forEach(item => {
            const reg = /(#&\d+@)/g
            const inputele = ' '
            const stem = item.ItemTitle.replace(reg, inputele)
            item.ItemTitle = stem
          })
        }
        if (res.Data.length === 0) {
          this.showLack = true
          this.allSubjectNum = 0
        } else {
          this.showLack = false
        }
        this.loadSubjectList = false
      })
    },
    // 获取章节id
    getChapterList () {
      this.$uwonhttp.post('/Chapter/TeacherChapter/GetChapterByClass', { ClassId: this.defaultValueClass }).then(res => {
        console.log('章节信息', res)
        // debugger
        this.chapterList = res.data.Data
        this.defaultValueChapterDe = [res.data.Data[0].ChapterId, res.data.Data[0].Second[0].ChapterId]
        this.defaultValueChapter = res.data.Data[0].Second[0].ChapterId
        this.loadChapter = true
        console.log('默认章节', this.defaultValueChapterDe)
        this.GetHighFrequencyList()
      })
    },
    // 展开详情按钮
    questionDetail (itemId, index, e) {
      this.loadCoach = true
      // e.target.innerText === '展开详情↓' ? e.target.innerText = '收起详情↑' : e.target.innerText = '展开详情↓'
      if (e.target.innerText === '展开详情↓') {
        const info = document.getElementsByClassName('packInfo')
        for (const key of info) {
          console.log('key', key.innerText)
          key.innerText = '展开详情↓'
        }
        e.target.innerText = '收起详情↑'
        // 展开
        this.showCoach = true
        this.showAloneId = itemId
        console.log('哈哈哈')
        this.getOpenInfor(itemId)
      } else {
        const info = document.getElementsByClassName('packInfo')
        for (const key of info) {
          console.log('key', key.innerText)
          key.innerText = '展开详情↓'
        }
        e.target.innerText = '展开详情↓'
        this.showAloneId = ''
      }
    },
    // 获取展开详情
    getOpenInfor (id) {
      this.$http.post('/HighFrequency/HighFrequency/GetHighFrequencyInfo', { userId: this.userId, classId: this.defaultValueClass, ItemId: id, chapterId: this.defaultValueChapter }).then(res => {
        console.log('展开详情', res)
        // 答案详情
        this.coachList = res.Data.UserAnswers
        this.coachData = res.Data
        this.playerOptions.sources[0].src = res.Data.CoachVideo
        this.loadCoach = false
      })
    },
    // 加载展开详情
    loas (qid, index) {
      this.$http.post('/report/TeacherReport/GetPaperItemErrorInfoBySchool', {
        paperId: '1329629771541254145',
        userId: 74214,
        ItemId: qid
      }).then(res => {
        var Data = res.Data
        this.allAnswerDetail[index] = Data
        console.log(this.allAnswerDetail)
        const allCount = this.allAnswerDetail[index].map(item => {
          return item.Count
        })
        var len = Data.length
        var countNums = 0
        for (var value of allCount) {
          countNums += value
        }
        this.countNum[index] = countNums
        console.log(this.countNum)
        this.$forceUpdate()
      })
    },
    // 语音
    playAdiuoed () {
      console.log('播放')
      // debugger
      this.activePlay = '1'
      const audio = this.$refs.mp3
      audio.play()
      console.log(parseInt(audio.duration))
    },
    overAudio () {
      this.activePlay = ''
    },
    playSound () {
      console.log('11111')
      // debugger
      const audio = this.$refs.mp3
      this.playTime = parseInt(audio[0].duration)
      console.log(parseInt(audio[0].duration))
      // console.log(this.playTime)
      // this.activePlay = '1'
      // recorder.play()
      // recorder.onplayend = this.playend()
    },
    // 编辑推送
    editPush () {
      // this.$router.push({ path: '/Teacher/My_Task/HighWrong/EditPush' })
      if (this.editeBtn === '编辑推送') {
        this.editeBtn = '确定'
        this.selectTest = true
      } else {
        if (this.arr1 === '') {
          this.$message.success('请选择题目进行编辑推送')
        } else {
          this.editeBtn = '编辑推送'
          this.selectTest = false
          this.$http.post('/HighFrequency/HighFrequency/AddHighFrequencyPaper', { CreatorId: this.userId, classId: this.defaultValueClass, ChapterId: this.defaultValueChapter, itemIds: this.arr1 }).then(res => {
            console.log('选入错题列表', res)
            this.paperId = res.Data
            this.$router.push({ path: '/Teacher/My_Task/HighWrong/EditPush',
              query: {
                paperId: this.paperId,
                chapterId: this.defaultValueChapter
              } })
          })
        }
      }
    },
    // 单选试题
    selectPaper (id, e, show) {
      if (this.arr.indexOf(id) === -1) {
        this.arr.push(id)
      }

      console.log('单选试卷ID', this.arr)

      // console.log(this.arr1)
      if (e.target.src === require('@/assets/teacher/new-sel.png')) {
        // 选中
        e.target.src = require('@/assets/teacher/new-true-sle.png')
      } else {
        e.target.src = require('@/assets/teacher/new-sel.png')
        // debugger
        const idx = this.arr.indexOf(id)
        if (idx !== -1) {
          this.arr.splice(idx, 1)
        }
      }
      this.arr1 = this.arr.join(',')
      console.log('arr1=====', this.arr1)
      // this.chapterPaperId = this.arr1
    },
    // 历史编辑
    editHistory () {
      this.$router.push('/Teacher/My_Task/HighWrong/EditHistroy')
    },
    // 错题辅导
    toWrongCoach (id, chapterId, gradeId) {
      this.$router.push({ path: '/Teacher/My_Coach/AccurateCoach',
        query: {
          classId: this.defaultValueClass,
          itemId: id,
          chapterId: chapterId,
          gradeId
        } })
    },
    onPaging (pageNum) {
      console.log(pageNum)
      this.pageNo = pageNum
      this.GetHighFrequencyList(0)
    }
  }
}
</script>

<style lang="less" scoped>
  .m-r {
    margin-right: 15px;
  }
  .m-b {
    margin-bottom: 16px;
  }
  .f-s {
    font-size: 18px;
    color: #3B3B3B;
  }
  .mar {
    margin: 15px 0;
  }
  .quote-arrangement {
    width: 74px;
    height: 32px;
    line-height: 32px;
    margin-right: 20px;
    text-align: center;
    color: #FF682C;
    border: 1px solid #FF682C;
    border-radius: 5px;
  }
  // 编辑推送按钮
  .edit-btn {
    display: inline-block;
    width: 144px;
    height: 32px;
    line-height: 32px;
    text-align: center;
    color: #fff;
    background-color: #68BB97;
    border-radius: 20px;
  }
  .edit-btn-history {
    display: inline-block;
    width: 144px;
    height: 32px;
    line-height: 32px;
    text-align: center;
    color: #ccc;
    border: 1px solid #E4E4E4;
    border-radius: 20px;
  }

  // 图片
  .img-p {
    margin-bottom: 20px;
    .change-big {
      width: 140px;
      margin-right: 25px;
      cursor: url('../../../../assets/student/bbig.png'),auto;
    }
  }

  // 视频
  .coach-video {
    width: 300px;
  }
  // 表格
  .accuracy {
    width: 55%;
    margin-bottom: 20px;
  }
  // 设置
  .popup-input {
    width: 70px;
    margin: 0 15px;
    text-align: center;
    // text-indent: 20px;
    border: 1px solid #ddd;
  }
  .popup-p {
    margin: 10px 0;
    // color: #262626;
  }
  // 错题辅导按钮
  .wrong-coach {
    padding: 5px 10px;
    color: #fff;
    background-color: #68BB97;
    border-radius: 5px;
  }
  .already-coach {
    padding: 5px 10px;
    color: #FF682C;
    border: 1px solid #FF682C;
    border-radius: 5px;
  }
  // 编辑按钮
  .edit {
    margin-right: 25px;
    padding: 5px 10px;
    // color: #fff;
    border: 1px solid #BEBEBE;
    border-radius: 5px;
  }
  // 选择
  .select-info {
    margin-bottom: 16px;
    padding: 15px 0px 15px 15px;
    background-color: #fff;
    border-radius: 5px;
  }
  // 辅导讲解
  .coach-explain {
    p:nth-child(2) {
      margin-bottom: 15px;
    }
  }
  // 音频
  .play-adiuo {
      width: 201px;
      height: 39px;
      line-height: 39px;
      margin-top: 15px;
      margin-bottom: 15px;
      text-align: center;
      background: url('../../../../assets/correcting/录音show.png') no-repeat center bottom;
      cursor: pointer;
  }
  // 语音
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
      animation: soundShow 1.6s linear infinite;
    }
  .bottom{
    width: 100%;
    margin-bottom: 20px;
    border-radius: 5px;
    background-color: white;
    padding: 20px 20px;
    overflow: hidden;
}
  // 总题数
  .chapter-title {
    font-size: 20px;
  }
.showProcress{
    display: inline-block;
    padding: 20px 20px;
    // background-color: #F4F4F4;
    width: 100%;
}
.process{
    position: relative;
    margin-right: 12px;
    width: 60px;
    height: 95px;
    display: inline-block;
    // margin: 10px 0px;
}
.proce{
    width: 100%;
    height: 70%;
    text-align: center;
}
.number{
    width: 30%;
    height: 30%;
    text-align: center;
    margin: 0 auto;
}

.showQuestion{
    width: 100%;
    height:  900px;
    overflow-y: scroll;
    // background-color: white;
}
.type{
    width: 100%;
    margin-bottom: 16px;
    background-color: #fff;
    border-radius: 5px;
}
.question{
    padding: 10px 30px;
    width: 100%;
    font-size: 15px;
}
.answer{
    width: 98%;
    padding: 10px 30px;
    background-color: #ece9ea;
    border-radius: 5px;
}
.answer-details {
  border-bottom: 1px solid #DADADA;
}
.answerDetail{
    width: 100%;
    // padding: 10px 30px;
    font-size: 14px;
    margin-bottom: 20px;
    background-color: #ece9ea;
    // display:none;
    .anw-p1 {
      display: inline-block;
      width: 120px;
    }
}
.anserTitle{
    font-size: 16px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #4D5753;
}
.ana-bor-up {
    margin-bottom: 30px;
    // padding-bottom: 20px;
    // border-bottom: 1px solid #DADADA;
}
.options{
    display: inline-block;
}
.progressSpan{
    display: inline-block;
    width: 250px;
    height: 15px;
    margin-right: 10px;
    position: relative;
    /deep/.ant-progress-line{
        width:100%;
    }

}
.packInfo{
    cursor: pointer;
    color: #108ee9;
}
.isShowAnswer{
    display: none;
}
.detailOpt{

    display: inline-block;
}
.detailOpt:nth-of-type(1){
    white-space: nowrap;
}
.detailOptProgress{
    display: inline-block;
    width: 250px;
    height: 15px;
}
.Accuracy{
    color:#f6878a;
}
.AccuracyPrecent{
    font-size: 14px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
}
/deep/.ant-progress-text{
    font-size: 16px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #fd7b47;
}
  // 表格
  /deep/.ant-table-thead {
    /deep/.ant-table-align-center {
      color: #fff;
      background-color: #68BB97;
    }
  }
  /deep/.ant-table-tbody tr:nth-child(odd) {
      background-color: #F4F4F4;
  }
  /deep/.ant-table-tbody tr:nth-child(even) {
      background-color: #fff;
  }
  // 进度条
  /deep/ .ant-progress-success-bg{
    background-color:#1890FF;
}
  .answer-correct {
     /deep/ .ant-progress-success-bg{
        background-color:#FF682C;
    }
  }
  // /deep/.ant-select-selection {
  //   border: none;
  // }
</style>
