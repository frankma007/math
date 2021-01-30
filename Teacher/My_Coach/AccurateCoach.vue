<template>
  <div>
    <!--  题目题干 -->
    <div class="testQuestion">
      <!-- <span>{{ testNum }},&nbsp;</span> -->
      <span>1. </span>
      <span v-html="aloneSubject.ItemTitle"></span>
    </div>
    <!-- 题目选项 -->
    <span class="test-option" v-show="aloneSubject.ItemType === 2" v-for="item in aloneSubject.Opts" :key="item.Id">
      <span>{{ item.Opt }}.&nbsp;</span>
      <span v-html="item.OptContent"></span>
    </span>
    <!-- 试卷章节名称 -->
    <p class="chapter-name"><img src="@/assets/finemicro/link.png" alt=""> <span @click="toSubjectAnalysis">{{ aloneSubject.ChapterName }}</span></p>
    <!-- 正确率 -->
    <div class="accuracy">
      <a-table
        :columns="columns"
        :data-source="aloneSubject.Per"
        bordered
        :pagination="pagination">
      </a-table>
    </div>
    <!-- 详情统计 -->
    <div></div>
    <!-- 辅导讲解 -->
    <div class="coach-explain">
      <div style="position: relative" class="div">
        <p style="font-size: 16px;">辅导讲解</p>
        <textarea v-model="coaText"></textarea>
        <!-- 图片 -->
        <div>
          <span  v-for="(item,index) in ImgList" :key="index">
            <img class="upload-img change-big" :src="item.url" alt="" preview="1">
          </span>
          
            <!-- <img class="change-big" style="width: 200px;" src="@/assets/student/welcome-banner.png" alt="" preview="1"> -->
              <!-- <img style="width: 200px;" src="@/assets/student/welcome-banner.png" alt="" preview="2"> -->
        </div>
        <div>
          <p @click="playSound" v-if="soundTime !== '' && soundTime > 0" class="sound-show" :class="{'active': active === '1'}">{{ soundTime }}</p>
        </div>
        <div class="video-inter" v-show="uploadVideoList">
           <video-player
              class=" vjs-custom-skin"
              ref="videoPlayer"
              :playsinline="true"
              :options="playerOptions">
            </video-player>
        </div>
        <!-- 操作按钮 -->
        <div class="operation-btn">
          <span @click="handleCancel">取消</span>
          <span @click="handleOk">完成</span>
          <a-modal
              title="指向发布"
              :visible="visible"
              :width="650"
              okText="立即发布"
              centered
              @ok="handlePopupOk"
              @cancel="handlePopupCancel"
            >
              <div class="">
                <a-checkbox-group @change="classChange1">
                  <a-checkbox v-for="item in classInfor" :key="item.Id" :value="item.ClassId">{{ item.ClassName }}</a-checkbox>
                </a-checkbox-group>
                <div class="clearfix">
                  <div class="student-list fl" v-if="studentSeach">
                    <p><input class="input" type="text" placeholder="输入学生姓名"></p>
                    <!-- 错误 -->
                    <div>
                       <p>
                          <img src="@/assets/login/学校.png" alt="">
                          <span class="wrongStu" style="vertical-align: middle;">错误学生</span>
                          <!-- 全选 allcheckIcon-->
                          <img @click="wrongAllSel($event)" class="select-icon cur" :src="allcheckIcon" alt="">
                          <img class="cur" src="@/assets/schoolmamager/icon向下.png" alt="">
                        </p>
                        <p v-for="(item,index) in wrongList" :key="index"> 
                          <span class="student-info">
                            <span>{{ item.ClassName }}</span>
                            <img class="head-img"  :src="item.Photo" alt="">
                            <span style="vertical-align: middle;">{{ item.RealName }}</span>
                          </span>
                          <!-- 单选 checkIcon-->
                          <img @click="aloneCheck(item.RealName,item.Photo,item.StudentId,$event)" class="cur img" :src="checkIcon" alt="">
                        </p>
                    </div>
                    <!-- 正确 -->
                    <div>
                      <p>
                        <img src="@/assets/login/学校.png" alt="">
                        <span class="wrongStu" style="vertical-align: middle;">正确学生</span>
                        <img @click="correctSel($event)" class="select-icon cur" :src="allcheckCorrectIcon" alt="">
                        <img class="cur" src="@/assets/schoolmamager/icon向下.png" alt="">
                      </p>
                      <p v-for="(item,index) in correctList" :key="index">
                        <span class="student-info">
                          <span>{{ item.ClassName }}</span>
                          <img class="head-img"  :src="item.Photo" alt="">
                          <span style="vertical-align: middle;">{{ item.RealName }}</span>
                        </span>
                        <!-- 单选正确 -->
                        <img @click="aloneCheckCorrect(item.RealName,item.Photo,item.StudentId,$event)" class="cur cor-img" :src="checkCorrectIcon" alt="">
                      </p>
                    </div>
                  </div>
                  <div class="fl show-stu-all" v-if="studentSeach">
                    <p>已选中{{ allSelList.length }}名学生</p>
                    <div>
                      <!-- <span @click="delStuSel(item.RealName,item.Photo,item.StudentId,$event)" class="cur">×</span>{{ item.StudentId }} -->
                      <p class="showStudent" v-for="(item,index) in allSelList" :key="index"><img style="width:30px;" :src="item.Photo" alt="">{{ item.RealName }} </p>
                    </div>
                  </div>
                </div>
              </div>
          </a-modal>
        </div>
        <!-- 录音弹窗 -->
        <div v-if="soundPopup" class="sound-popup">
          <!-- <img @click="soundShow" v-show="stopItHide" src="@/assets/correcting/录音弹窗.png" alt="">
          <img @click="hideSound" v-show="stopIt" src="@/assets/correcting/点击停止.png" alt="">
          <p v-show="stopItHide">点击说话</p>
          <p v-show="stopIt">点击停止</p> -->

           <img @click="soundShow" v-show="stopItHide" src="@/assets/correcting/录音弹窗.png" alt />
            <!-- <img @click="hideSound" v-show="stopIt" src="@/assets/correcting/点击停止.png" alt /> -->
            <div v-show="stopIt" @click="hideSound" class="click-stop"></div>
            <p v-show="stopItHide">支持最多60s语音</p>
            <p v-show="stopItHide" @click="soundShow">点击说话</p>
            <p v-show="stopIt && showCountDown">{{ content }}</p>
            <p v-show="stopIt" @click="hideSound">点击停止</p>
        </div>
      </div>
      <div>
        <p><label for="inputId" class="img-uploader-label"><img src="@/assets/correcting/图片上传.png" alt=""></label></p>
        <input
          style="display: none"
          id="inputId"
          ref="input"
          type="file"
          @change="uploadImg($event)"
          accept="image/gif, image/jpeg, image/jpg, image/png, image/svg"
        >
        <!-- <a-upload id="inputId" name="file" :multiple="true" :action="uploadPictureUrl" :headers="headers" @change="filePicture">
            <a-button>
              <a-icon type="upload" />选择图片导入
            </a-button>
          </a-upload> -->
        <p><img @click="soundRecording" src="@/assets/correcting/录音图片.png" alt=""></p>
        <!-- 视频 -->
        <p><label for="videoId" class="img-uploader-label"><img src="@/assets/correcting/视频图片.png" alt=""></label></p>
        <input
          style="display: none"
          id="videoId"
          ref="input"
          type="file"
          @change="uploadVideo($event)"
        >
      </div>
    </div>

  </div>
</template>

<script>
import Recorder from 'js-audio-recorder'
const columns = [
  {
    title: '-----',
    dataIndex: 'name',
    key: 'name',
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
const data = [
  {
    key: '1',
    name: '平均正确率',
    age: '85%',
    address: '65%',
    area: '75%'
  }
]
const recorder = new Recorder()

export default {
  created () {
    this.gradeId = this.$route.query.gradeId
    this.userId = localStorage.getItem('UserId')
    // 当前班级ID
    this.classId = this.$route.query.classId
    // 当前试题id
    this.itemId = this.$route.query.itemId
    this.chapterId = this.$route.query.chapterId
    // 获取单个题信息
    this.getAloneInfor()
    this.getTeacherList()
    // 获取题目对应选项
    // this.getTestOption()
    // 获取章节名称
    // this.getChapterName()
  },
  watch: {
    $route () {
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/My_Coach/AccurateCoach?' + ChangId) {
        // this.getTestOption()
        this.gradeId = this.$route.query.gradeId
        // 当前班级ID
        this.classId = this.$route.query.classId
        // 当前试题id
        this.itemId = this.$route.query.itemId
        this.chapterId = this.$route.query.chapterId
        this.coaText = ''
         this.ImgList = []
        // 获取单个题信息
        this.getAloneInfor()
        this.getTeacherList()
      }
    }
  },
  data () {
    return {
      gradeId: '',
      // 辅导文字
      coaText: '',
      // 全选错误
      arr: [],
      allcheckIcon: require('@/assets/teacher/未勾选.png'),
      allcheckIconTrue: require('@/assets/teacher/勾选.png'),
      // 单选错误
      checkIcon: require('@/assets/teacher/未勾选.png'),
      // 正确全选
      arr1: [],
      allcheckCorrectIcon:require('@/assets/teacher/未勾选.png'),
      allcheckCorrectIconTrue: require('@/assets/teacher/勾选.png'),
      // 单选正确
      checkCorrectIcon: require('@/assets/teacher/未勾选.png'),
      studentSeach: false,
      allSelList: [],
      // 完成弹窗
      visible: false,
      classInfor: [],
      // 弹窗班级id
      PopupClassId: '',
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
      // 错误学生列表
      wrongList: [],
      // 正确学生
      correctList: [],
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
      userId: '',
      classId: '',
      itemId: '',
      chapterId: '',
      soundTime: '',
      // 上传图片
      ImgList: [],
      // 上传视频
      uploadVideoList: false,
      // 单题数据
      aloneSubject: {},
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
      pagination: false,
      length: 0,
      //videoUrl 上传视频地址
      videoUrl: '',
      urlUp: '',
      // 上传图片地址
      picUrl: ''
    }
  },
  methods: {
    // 获取老师对应班级
    getTeacherList () {
      this.$http.post('/HighFrequency/HighFrequency/GetClassByUserGrade', { userId: this.userId, grade: this.gradeId }).then(res => {
        this.classInfor = res.Data
        console.log('老师对应班级---', this.classInfor)
      })
    },
    // 获取单题信息
    getAloneInfor () {
      this.$http.post('/HighFrequency/HighFrequency/GetCocahItemInfo', { userId: this.userId,classId: this.classId, itemId: this.itemId,chapterId: this.chapterId }).then(res => {
        console.log('单题详情', res)
        this.aloneSubject = res.Data
        const reg = /(#&\d+@)/g
        const stem = this.aloneSubject.ItemTitle.replace(reg, ' ')
        this.aloneSubject.ItemTitle = stem
         this.aloneSubject.Per.forEach(value => {
          const key = 'name'
          value[key] = '平均正确率'
        })
      })
    },
    uploadImg (e) {
      debugger
      console.log('上传图片', e.target.files)
      let file=e.target.files[0]
      var reader = new FileReader()
      reader.readAsDataURL(file)
      let url = ''
      const that = this
      reader.onload = function (e) {
        url=this.result.substring(this.result.indexOf(',')+1)
        // console.log('啊哈哈哈','data:image/png;base64,'+url)
        that.uploadImgInter('data:image/png;base64,'+url)
      }
     
      // console.log(e.target)
    },
    // 上传图片
    uploadImgInter (url) {
      
      if (this.length === 3) {
        this.$message.success('只能上传3张图片')
      } else {
         this.$http.post('/Paper/Exam_MicroLesson/PictureFileBase64', { base64: url }).then(res => {
        console.log('上传图片---', res)
           this.ImgList.push({
            url: res.Data
          })
         this.length = this.ImgList.length
       
        console.log('上传图片集合---',  this.ImgList)
        const pic = this.ImgList.map(value => {
          return value.url
        })
        // console.log(pic)
        this.picUrl = pic.join(',')
        // console.log(this.picUrl)
      })
      }
     
    },
    uploadVideo (e) {
      let file=e.target.files[0]
      var reader = new FileReader()
      reader.readAsDataURL(file)
      this.urlUp = ''
      const that = this
      reader.onload = function (e) {
        this.urlUp=this.result.substring(this.result.indexOf(',')+1)
        console.log('s视频', 'data:image/png;base64,'+this.urlUp)
        that.uploadVideoInter('data:image/png;base64,'+this.urlUp)
      }
    },
    // 视频上传
    uploadVideoInter (url) {
      this.$http.post('/Paper/Exam_MicroLesson/VideoFileBase64', { base64: url }).then(res => {
        console.log('上传视频', res)
        
        this.playerOptions.sources[0].src = res.Data
        this.uploadVideoList = true
        this.videoUrl = res.Data
      })
    },
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
    hideSound () {
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
          that.$uwonhttp.post('/ExamItem/ExamItem/AddItemMp3Base64',
            { studentId: that.active, itemId: that.subjectItemId, mp3Name: '1', mp3Location: '1', teacherId: that.userId, paperId: that.paperId, base64: e.target.result }).then(res => {
            console.log('luyin-------', e.target.result)
          })
        }
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
    // 取消
    handleCancel () {
      this.ImgList = []
      this.coaText = ''
      this.uploadVideoList = false
      this.videoUrl = ''
      this.soundTime = ''
    },
    handlePopupCancel () {
      this.visible = false
    },
    // 确定
    handleOk () {
      this.visible = true
      
    },
    // 多选框，获取选中的班级
    classChange1 (checkedValues) {
      // this.PopupClassId = checkedValues[checkedValues.length - 1]
      this.PopupClassId = checkedValues.join(',')
      console.log('选中班级----', this.PopupClassId)
      if (this.PopupClassId) {
        this.$http.post('/HighFrequency/HighFrequency/GetRightErrorStudents', { classId: this.PopupClassId, itemId: this.itemId }).then(res => {
        console.log('学生作答情况', res)
        this.wrongList = res.Data.ErrorStudent
        this.correctList = res.Data.RightStudent
        
       
        
          console.log('错误学生', this.wrongList)
          console.log('正确学生', this.correctList)
          this.studentSeach = true
      })
      }
      
    },
    // 错误学生全选
    wrongAllSel (e) {
       if (e.target.src === require('@/assets/teacher/未勾选.png')) {
        // this.checkIcon = this.checkIconTrue
        this.allcheckIcon = this.allcheckIconTrue
        // this.arr = this.paperList.filter(value => {
        //   return value.AuditStatus === 0
        // })
        // this.allPaper = this.arr.map(value => {
        //   return value.PaperId
        // })
        console.log('全选错误学生渲染-----', this.wrongList)
        // this.arr = this.wrongList
        const aa = JSON.stringify(this.wrongList)
        this.arr = JSON.parse(aa)
        console.log('全选错误学生', this.arr)

        // this.chapterPaperId = this.allPaper.join(',')
        // console.log('全选的试卷id', this.chapterPaperId)
        // this.IsAssign = true
        Array.from(document.getElementsByClassName('img')).forEach(value => {
          value.src = require('@/assets/teacher/勾选.png')
        })
        this.allSelList = [...this.arr, ...this.arr1]
        console.log('全选错误学生渲染--后', this.wrongList)
      } else {
        // debugger
        this.checkIcon = require('@/assets/teacher/未勾选.png')
        // console.log('单个', document.getElementsByClassName('img'))
        // document.getElementsByClassName('img')[3].src = require('@/assets/teacher/new-sel.png')
        Array.from(document.getElementsByClassName('img')).forEach(value => {
          value.src = require('@/assets/teacher/未勾选.png')
        })
        this.allcheckIcon = this.checkIcon
        this.arr = []
        this.allSelList = [...this.arr, ...this.arr1]
      }
    },
    // 单选错误学生
    aloneCheck (id,photo,stuId,e) {
      console.log('点击取消', this.wrongList)
     console.log(e)
      if (this.arr.length === 0) {
        this.arr.push({
          RealName:id,
          Photo: photo,
          StudentId: stuId
        })
      }
      // debugger
      

      console.log('单选学生ID', this.arr)

      // console.log(this.arr1)
      if (e.target.src === require('@/assets/teacher/未勾选.png')) {
        // 选中
        e.target.src = require('@/assets/teacher/勾选.png')
        for (var i = 0; i < this.arr.length; i++) {
        console.log('glfdmgkfdmfsd----', this.arr[i])
           if(this.arr.length>0){
            if (this.arr.findIndex(s=>s.StudentId==stuId)==-1) {
              this.arr.push({
              RealName:id,
              Photo: photo,
              StudentId: stuId
            })
          }
        }
        }
      } else {
        
        e.target.src = require('@/assets/teacher/未勾选.png')
        // debugger
        console.log('CUOWU之前', this.wrongList)
        for (var i = 0; i < this.arr.length; i++) {
            if ((this.arr[i].StudentId).indexOf(stuId) > -1) {//判断key为999的对象是否存在，
                const index = i;
                this.arr.splice(index, 1);//存在即删除
            }
        }
        console.log('CUOWU',this.wrongList)
        // const idx = this.arr.indexOf(id)
        // if (idx !== -1) {
        //   this.arr.splice(idx, 1)
        // }
      }
      this.allSelList = [...this.arr, ...this.arr1]
      console.log('再次点击学生', this.arr)
      if (this.wrongList.length !== this.arr.length) {
        this.allcheckIcon = require('@/assets/teacher/未勾选.png')
      }
      if (this.wrongList.length === this.arr.length) {
        this.allcheckIcon = this.allcheckIconTrue
      }
      // this.arr1 = this.arr.join(',')
    },
    // 正确全选
    correctSel (e) {
        if (e.target.src === require('@/assets/teacher/未勾选.png')) {
        // this.checkIcon = this.checkIconTrue
        this.allcheckCorrectIcon = this.allcheckCorrectIconTrue
        
          const aa = JSON.stringify(this.correctList)
        this.arr1 = JSON.parse(aa)
        console.log('全选正确学生', this.arr1)
        this.allSelList = [...this.arr, ...this.arr1]
        // this.chapterPaperId = this.allPaper.join(',')
        // console.log('全选的试卷id', this.chapterPaperId)
        // this.IsAssign = true
        Array.from(document.getElementsByClassName('cor-img')).forEach(value => {
          value.src = require('@/assets/teacher/勾选.png')
        })
      } else {
        // debugger
        this.checkIcon = require('@/assets/teacher/未勾选.png')
        // console.log('单个', document.getElementsByClassName('img'))
        // document.getElementsByClassName('img')[3].src = require('@/assets/teacher/new-sel.png')
        Array.from(document.getElementsByClassName('cor-img')).forEach(value => {
          value.src = require('@/assets/teacher/未勾选.png')
        })
        this.allcheckCorrectIcon = this.checkIcon
        this.arr1 = []
        this.allSelList = [...this.arr, ...this.arr1]
      }
    },
    // 单选正确
    aloneCheckCorrect (id,photo,stuId,e) {
      if (this.arr1.length === 0) {
        this.arr1.push({
          RealName:id,
          Photo: photo,
          StudentId: stuId
        })
      }
      if (e.target.src === require('@/assets/teacher/未勾选.png')) {
        // 选中
        e.target.src = require('@/assets/teacher/勾选.png')
        // debugger
        if(this.arr1.length>0){
            if (this.arr1.findIndex(s=>s.StudentId==stuId)==-1) {
              this.arr1.push({
              RealName:id,
              Photo: photo,
              StudentId: stuId
            })
          }
        }
        console.log('单选学生ID', this.arr1)
      } else {
        e.target.src = require('@/assets/teacher/未勾选.png')
        // debugger
        console.log('CUOWU之前', this.correctList)
        for (var i = 0; i < this.arr1.length; i++) {
            if ((this.arr1[i].StudentId).indexOf(stuId) > -1) {//判断key为999的对象是否存在，
                const index = i;
                this.arr1.splice(index, 1);//存在即删除
            }
        }
       
      }
      this.allSelList = [...this.arr, ...this.arr1]
      console.log('再次点击学生', this.arr1)
      console.log('点击学生最后', this.allSelList)
      if (this.correctList.length !== this.arr1.length) {
        this.allcheckCorrectIcon = require('@/assets/teacher/未勾选.png')
      }
      if (this.correctList.length === this.arr1.length) {
        this.allcheckCorrectIcon = this.allcheckCorrectIconTrue
      }
    },
    delStuSel (id,photo,stuId,e) {
      this.aloneCheck(id,photo,stuId,e)
      this.aloneCheckCorrect(id,photo,stuId,e)
    },
    // 发布辅导
    handlePopupOk () {
      const stuId = this.allSelList.map(value => {
        return value.StudentId
      })
      const allStu = stuId.join(',')
      console.log(allStu)
      console.log('图片', this.picUrl)
      this.$http.post('/HighFrequency/HighFrequency/SaveItemCoach',  { StudentIds: allStu, UserId: this.userId,PaperId:this.paperId,ItemId:this.itemId,ChapterId:this.chapterId,CruxText:this.coaText,LanguageUrl: '',PictureUrl: this.picUrl,VideoUrl: this.videoUrl}).then(res => {
        console.log('上传辅导', res)
        if (res.Success) {
          this.$message.success('辅导成功')
          this.$router.push({ path: '/Teacher/My_Task/HighWrong/HighWrong' })
        }
        this.visible = false
      })
    },
    // 跳转逐题分析
    toSubjectAnalysis () {
      // this.$router.push({ path: '/Teacher/My_Task/InspectionWorkData',
      //   query: {
      //     id: '2'
      //   }
      // })
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
  .upload-img {
    width: 150px;
    margin-right: 32px;
  }
   // 图片
  .change-big {
    margin-right: 25px;
    cursor: url('../../../assets/student/bbig.png'),auto;
  }
  .video-inter {
    // width: 420px;
    margin-top: 16px;
    width: 40%;
  }
  .student-list {
    width: 298px;
    height: 428px;
    overflow: scroll;
    padding: 14px 0 14px 20px;
    border: 1px solid #ccc;
    .input {
      width: 256px;
      margin-bottom: 24px;
      text-indent: 10px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
    .wrongStu {
      margin-right: 24px;
      margin-bottom: 10px;
    }
    .select-icon {
      margin-right: 67px;
    }
    .student-info {
      display: inline-block;
      width: 150px;
      margin-left: 23px;
      margin-bottom: 10px;
      .head-img {
        width: 30px;
        margin-right: 6px;
        border-radius: 20px;
      }
    }
  }
  .show-stu-all {
    width: 300px;
    padding: 12px 12px 0 12px;
    border: 1px solid #ccc;
  }
  .showStudent {
    margin: 12px 0;
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
      // width: 137px;
      // height: 25px;
      line-height: 25px;
      font-size: 16px;
      text-align: center;
      vertical-align: middle;
      border-radius: 5px;
      // background-color: #ccc;
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
    .div {
      width: 78%;
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
  // 操作按钮
  .operation-btn {
    position: absolute;
    right: 55px;
    bottom: 29px;
    span {
      display: inline-block;
      width: 80px;
      height: 32px;
      margin-right: 24px;
      line-height: 32px;
      text-align: center;
      color: #fff;
      border-radius: 5px;
      cursor: pointer;
    }
    span:nth-child(1) {
      background-color: #B5B5B5;
    }
     span:nth-child(2) {
      background-color: #68BB97;
    }
  }
</style>
