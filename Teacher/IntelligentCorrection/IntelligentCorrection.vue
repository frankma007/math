<template>
  <div>
    <div class="correction-wrapper">
      <h5>{{ paperName }}</h5>
      <div class="progress-info">
        <span>批改进程</span>
        <span>
          <a-progress
            :percent="(compeletedLength / student_length) * 100"
            :strokeWidth="16"
            :strokeColor="'#68bb97'"
            :showInfo="false"
          />
        </span>
        <!-- subNum -->
        <span>{{ compeletedLength }}/{{ student_length }}</span>
      </div>
      <div class="student-info">
        <!-- :compeleteList="compeleteList" -->
        <!-- :customExerciseId="parameterObj.customExerciseId" -->
        <Swiper :key="swiperKey" @eventGetCanvasInfo="eventGetCanvasInfo" :studentsList="studentsList"></Swiper>
      </div>
      <div class="correct-info">
        <!-- width="800" height="590" -->
        <div class="compelete" v-show="compelete === true">
          <img src="@/assets/teacher/IntelligentCorrection/compelete.png" alt />
          <p class="compelete-color">已全部批改完成，老师辛苦啦～</p>
        </div>
        <div v-show="compelete === false">
          <canvas class="canvas" id="c"></canvas>
        </div>

        <ul class="tools" v-show="compelete === false">
          <li class="tool-item" @click.stop="isSignature = true">
            <img src="@/assets/teacher/IntelligentCorrection/icon_signature.png" alt />
            <p>签名</p>
          </li>

          <li @click="openEvaluate()">
            <img src="@/assets/teacher/IntelligentCorrection/icon_evaluate.png" alt />
            <p>评价</p>
          </li>
          <li @click.stop="isSignature = true">
            <img src="@/assets/teacher/IntelligentCorrection/icon_write.png" alt />
            <p>手写</p>
          </li>
          <!--           <li @click="rotate90()">
            <img src="@/assets/teacher/IntelligentCorrection/icon_rotate.png" alt />
            <p>旋转</p>
          </li> -->

          <li @click="undo">
            <img src="@/assets/teacher/IntelligentCorrection/icon_repeal.png" alt />
            <p>撤销</p>
          </li>
        </ul>
      </div>

      <div class="submitinfo" v-show="compelete === false">
        <div class="pages">
          <span>当前批改：{{ activeCavasIndex + 1 }}/{{ canvasList.length }}</span>
          <div class="btn small" :class="{ gray: activeCavasIndex === 0 }" @click="prevpage()">上一页</div>
          <div class="btn small" :class="{ gray: activeCavasIndex === canvasList.length - 1 }" @click="nextpage()">
            下一页
          </div>
        </div>

        <div class="btn" :class="{ gray: compelete }" @click.prevent="SubmitCorrectStudentExercise()">提交批改</div>
      </div>
      <Signature @getSignature="getSignature" v-if="isSignature === true"></Signature>
      <Evaluate
        @evluateClose="evluateClose"
        :StudentName="studentsList[currentStudentIndex].StudentName"
        :exerciseData="exerciseData"
        v-if="isopenEvaluate === true"
        @emconfirmEvaluate="emconfirmEvaluate"
      ></Evaluate>
    </div>
  </div>
</template>

<script>
import Swiper from './componets/Swiper.vue'
import Signature from './componets/Signature.vue'
import Evaluate from './componets/Evaluate.vue'

import { fabric } from 'fabric'
export default {
  name: 'IntelligentCorrection',
  components: {
    Swiper,
    fabric,
    Signature,
    Evaluate,
  },
  data() {
    return {
      paperName: '',
      compeletedLength: 0,
      compeletedList: [],
      swiperKey: 0, //重绘swiper子组件的key
      student_length: 0,
      isCommitEva: false, //是否提交过评价
      currentStudentIndex: 0, //当前批改学生的信息索引
      exerciseData: null,

      //当前批改的页面
      activeCavasIndex: 0,
      StudentId: '', //请求canvas列表信息参数
      studentsList: null, //头像接口返回参数
      isopenEvaluate: false, //是否弹出评价窗口
      //是否签名
      isSignature: false,
      //待批改学生的入参
      parameterObj: {
        customExerciseId: '', //自定义练习Id
        classId: '', //班级id
      },
      imgSrc: [], //存储要提交的文件路径
      // '@/assets/teacher/IntelligentCorrection/compelete.png',
      compelete: false, //是否完成
      mouseFrom: {},
      mouseTo: {},
      drawingObject: null, // 当前绘制对象
      drawType: '',
      textbox: null, // 文本编辑状态
      color: '#E34F51', // 画笔颜色
      drawWidth: 2, // 笔触宽度
      moveCount: 1, // 绘制移动计数器
      doDrawing: false, // 绘制状态
      //关于撤销的一些配置
      // config: {
      //   canvasState: [],
      //   currentStateIndex: -1,
      //   undoStatus: false, // 撤销状态
      //   undoFinishedStatus: 1,
      // },
      canvas: null, //当前canvas 的情况
      canvasList: [], //从接口中获取的试卷数目
      //旋转
      // angle: 0,
      // signatureImg:''
      canvasJson: [], //存储绘图步骤jsoon
      logArr: [], //只能批改修改主动修改的记录
      scales: 1, //canvas图片缩放情况比率
      canvasArr: [], //canvas数组
      cvRoot: null,
      // isFirst:true,
      steps: 0, //记录撤销步骤
      //  compeleteList:[],
    }
  },
  created() {
    this.parameterObj = {
      customExerciseId: this.$route.query.customId, //自定义练习Id
      classId: this.$route.query.classId, //班级id
    }

    this.paperName = this.$route.query.paperName
    console.log(this.parameterObj)
    /* this.parameterObj.customExerciseId= this.$route.customId
this.parameterObj.classId= this.$route.classId */
  },
  mounted() {
    this.initC()
    this.getTodoCorrectStudents()
  },
  computed: {
    obj() {
      const { compelete, studentsList, student_length, steps } = this
      return {
        compelete,
        studentsList,
        student_length,
        steps,
      }
    },
  },
  watch: {
    obj: {
      handler(val) {
        this.$nextTick(() => {
          this.compelete = val.compelete
          this.studentsList = val.studentsList
          this.student_length = val.student_length
          this.steps = val.steps
        })
      },
      deep: true,
    },
  },
  methods: {
    //判断是否已经全部批改完成
    isAllCompelete() {
      var index = this.studentsList.findIndex((val) => {
        return val.compelete === false
      })
      if (index !== -1) {
        // this.$message.error('当前学生全部批改完成后方可提交')
        return false
      }
      return true
    },
    // 图片转换成二进制
    dataURItoBlob(dataURI) {
      var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0] // mime类型
      var byteString = atob(dataURI.split(',')[1]) // base64 解码
      var arrayBuffer = new ArrayBuffer(byteString.length) // 创建缓冲数组
      var intArray = new Uint8Array(arrayBuffer) // 创建视图

      for (var i = 0; i < byteString.length; i++) {
        intArray[i] = byteString.charCodeAt(i)
      }
      return new Blob([intArray], { type: mimeString })
    },
    //提交批改
    SubmitCorrectStudentExercise() {
      // /Customer/CustomExercise/SubmitCorrectStudentExercise
      // this.logArr
      this.canvas.renderAll()
      this.imgSrc[this.activeCavasIndex] = this.canvas.toDataURL('image/png')

      var index = this.imgSrc.findIndex((val) => {
        return val === ''
      })
      if (index !== -1) {
        this.$message.error('当前学生全部批改完成后方可提交')
        return
      }

      if (this.isCommitEva === false) {
        const evaluateprams = {
          AudioUrl: '',
          Comment: '',
          Epilogue: this.exerciseData.Epilogue,
          NeedCorrect: 1,
          StudentDoPaperInfoId: this.exerciseData.StudentDoPaperInfoId,
          TeacherId: localStorage.UserId,
        }
        this.SubmitComment(evaluateprams)
      }

      // console.log(this.logArr, '看看日志')
      debugger
      //重要
      this.canvasList.forEach((val, index) => {
        const formData = new FormData()
        const bold = this.dataURItoBlob(this.imgSrc[index]) //将图片转为二进制
        formData.set('TeacherId', localStorage.UserId)
        formData.set('StudentDoPaperInfoId', this.exerciseData.StudentDoPaperInfoId)

        formData.set('CorrectResultJson', JSON.stringify(this.logArr[index]))

        formData.set('file', bold, 'fileName')

        console.log(formData)
        debugger
        this.$uwonhttp.post('/Customer/CustomExercise/SubmitCorrectStudentExercise', formData).then((res) => {
          console.log('ok 成功了')
        })
      })

      console.log('ok 成功了')
      this.studentsList[this.currentStudentIndex].compelete = true
      this.compeletedList = this.studentsList.filter((val) => {
        return (val.compelete = true)
      })
      this.compeletedLength = this.compeletedList.length
      this.compelete = this.isAllCompelete()

      debugger
      // this.studentsList[this.currentStudentIndex].compelete = true

      // this.$refs.swiper.changeDate()
      this.swiperKey++

      //  this.compeleteList[this.currentStudentIndex]=this.currentStudentIndex

      /*       var obj = {
        TeacherId: localStorage.UserId,
        StudentDoPaperInfoId: this.exerciseData.StudentDoPaperInfoId,
        CorrectResultJson: JSON.stringify(this.logArr),
        // CorrectResultJson: [
        //   {
        //     Result: 'YES',
        //     MarkId: '1347045204967858186',
        //   },
        // ],
      }
      this.$uwonhttp.post('/Customer/CustomExercise/SubmitCorrectStudentExercise', obj).then((res) => {
        //  this.studentsList[0].StudentId
        //  this.studentsList[0].StudentId
        // currentStudentIndex
      }) */
    },
    //当前批改上一页
    prevpage() {
      if (this.activeCavasIndex === 0) {
        return
      }

      this.canvas.renderAll()
      this.canvasArr[this.activeCavasIndex] = this.canvas.toJSON([
        'ItemResult',
        'MarkId',
        'selectable',
        'selection',
        'hasControls',
        'borderColor', // 图层控件边框的颜色
      ])
      this.imgSrc[this.activeCavasIndex] = this.canvas.toDataURL('image/png')

      this.canvas.clear()
      this.canvas = null
      this.activeCavasIndex--
      this.saveImg()
    },
    //当前批改下一页
    nextpage() {
      if (this.activeCavasIndex === this.canvasList.length - 1) {
        return
      }
      //  this.canvasList[this.activeCavasIndex]=this.canvasArr[self.activeCavasIndex].toDataURL('image/png')
      this.canvas.renderAll()
      // this.canvasJson[this.activeCavasIndex] = this.canvas.toJSON()
      this.canvasArr[this.activeCavasIndex] = this.canvas.toJSON([
        'ItemResult',
        'MarkId',
        'selectable',
        'selection',
        'hasControls',
        'borderColor', // 图层控件边框的颜色
      ])
      debugger
      this.imgSrc[this.activeCavasIndex] = this.canvas.toDataURL('image/png')
      this.canvas.clear()
      this.canvas = null
      this.activeCavasIndex++
      this.saveImg()
    },
    //获取canvas 内容
    eventGetCanvasInfo(studentid, currentStudentIndex) {
      this.currentStudentIndex = currentStudentIndex
      this.getStudentTodoCorrectExercise(studentid)
    },
    emconfirmEvaluate(obj) {
      console.info(obj, '评价内容')
      this.SubmitComment(obj)
      this.isopenEvaluate = false //关闭弹窗
    },
    SubmitComment(obj) {
      this.$uwonhttp
        .post('/Customer/CustomExercise/SubmitComment', obj)
        .then((res) => {})
        .then(() => {
          this.isCommitEva = true
        })
    },

    openEvaluate() {
      this.isopenEvaluate = true //打开弹窗
    },
    getSignature(imgStr) {
      //关闭签名弹窗 将签名图增加在canvas中
      this.steps++
      this.isSignature = false
      // var imgobj=null
      var that = this
      fabric.Image.fromURL(imgStr, (oImg) => {
        //图片缩小0.3倍
        oImg.set({
          left: that.canvas.width * 0.7,
          top: that.canvas.height * 0.8,
          hasControls: true, // 是否开启图层的控件
          borderColor: 'orange', // 图层控件边框的颜色
          scaleX: 0.3,
          scaleY: 0.3,
        })
        console.log(oImg, '看看对象')

        // imgobj=oImg
        that.canvasJson[that.activeCavasIndex].push(oImg)
        that.canvas.add(oImg).setActiveObject(oImg)
        // that.canvas.selectedObj.bringForward()
        that.canvas.renderAll()
      })

      // setTimeout(() => {

      // }, 200);

      // this.isFirst=false
    },
    //撤销
    undo() {
      // console.log(this.steps,"kanaa");
      debugger
      if (this.steps === 0) {
        return
      }
      this.canvas._objects.pop()
      this.canvasJson[this.activeCavasIndex].pop()
      this.steps--
      this.canvas.renderAll()
    },

    // 下载图片
    saveImg() {
      // this.saveImg()

      this.initCanvas()
    },
    initC() {
      this.cvRoot = new fabric.Canvas('c')
    },

    // 初始化画板
    initCanvas() {
      window.zoom = window.zoom ? window.zoom : 1
      //  this.canvasList[index].json=this.canvas.toJSON()

      // console.log(img,"看看图");
      var self = this
      self.steps = 0

      self.canvas = null
      // 初始化画板
      // self.canvas = new fabric.Canvas('c')
      self.canvas = self.cvRoot
      self.canvas.clear()
      self.canvas.off('mouse:down')
      self.canvas.on('mouse:down', (e) => {
        // 选中图层事件触发时，动态更新赋值
        // this.selectedObj = e.target
        console.log(e.target, '当前选中的图层')
        // card.remove(this.selectedObj) // 传入需要移除的object
        if (!e.target || typeof e.target.ItemResult == 'undefined') {
          return
        }
        if (typeof e.target.ItemResult == 'undefined') {
          return
        }
        var { MarkId, left, top, ItemResult } = e.target

        let imgUrl = ''
        if (ItemResult === 'NO') {
          ItemResult = 'YES'
          imgUrl = require('@/assets/teacher/IntelligentCorrection/duihao.png')
        } else if (ItemResult === 'YES') {
          ItemResult = 'NO'
          imgUrl = require('@/assets/teacher/IntelligentCorrection/chahao.png')
        }
        //添加新图层并记录下修改过的markid
        fabric.Image.fromURL(imgUrl, (img) => {
          img.set({
            left,
            top,
            hasControls: true, // 是否开启图层的控件
            borderColor: 'orange', // 图层控件边框的颜色
            scaleX: 0.5,
            scaleY: 0.5,
            selectable: false,
            selection: false,
            ItemResult,
            MarkId,
          })
          // 添加对象
          self.canvas.add(img)
        })
        // this.canvas.renderAll()
        var json = {
          ItemResult,
          MarkId,
        }
        self.canvas.remove(e.target) // 传入需要移除的object
        self.canvas.discardActiveObject() // 清除选中框
        var ishaveMarkId = false
        if (self.logArr[self.activeCavasIndex].length > 0) {
          let index = self.logArr[self.activeCavasIndex].findIndex((item) => {
            return item.MarkId === json.MarkId
          })
          if (index !== -1) {
            self.logArr[self.activeCavasIndex][index].ItemResult = json.ItemResult
            //  self.logArr[self.activeCavasIndex].push(json)
          } else {
            self.logArr[self.activeCavasIndex].push(json)
          }
        } else {
          self.logArr[self.activeCavasIndex].push(json)
        }
      })

      debugger
      // 有路径步骤 就跳过
      if (!!self.canvasArr[self.activeCavasIndex]) {
        // 加载画布信息
        self.canvas.loadFromJSON(self.canvasArr[self.activeCavasIndex], () => {
          self.canvas.renderAll()
        })
        /*         self.canvas.Image.set({
 hasControls: true, // 是否开启图层的控件
            borderColor: 'orange', // 图层控件边框的颜色
          
          
            selectable: false,
            selection: false,
        }) */

        return
      }
      self.canvas.renderAll()
      // 在画布初始化后设置
      self.canvas.preserveObjectStacking = true // 禁止选中图层时自定置于顶部
      self.canvas.isDrawingMode = false //禁止自由画笔
      self.canvas.selection = false //禁止画布所有选中
      self.canvas.skipTargetFind = false
      self.canvas.selectable = false //控件不能被选择，不会被操作

      self.canvas.hoverCursor = 'pointer'
      self.canvas.setZoom(1) //设置画板缩放比例

      // if(self.canvasJson[self.activeCavasIndex].length){
      // self.canvas.loadFromJSON(self.canvasJson[self.activeCavasIndex], () => {
      //  self.canvas.renderAll()
      // })

      // }

      self.canvas.renderAll()
      fabric.Image.fromURL(
        self.canvasList[self.activeCavasIndex],
        (oImg) => {
          self.canvas.setWidth(oImg.width)
          self.canvas.setHeight(oImg.height)
          self.canvas.setBackgroundImage(oImg).renderAll()
        },
        {
          crossOrigin: 'Anonymous',
        }
      )

      // self.canvas.renderAll()
      // this.canvas.renderAll()
      // self.canvas.off('object:added')
      /*       self.canvas.on('object:added', () => {
        // // 由于初始化时，添加了默认背景图片，只需一次
        // if (self.isFirst) {
        //   self.updateCanvasState()
        //   self.isFirst = false
        // }
        // var arr2 = [].concat(arr1)

if(this.isFirst == true){
        this.stateArr = [];
    }

      }) */

      self.canvas.renderAll()
      self.getMark()
      /*       if(self.canvasJson[self.activeCavasIndex].length){
        // self.canvas.loadFromJSON(self.canvasJson[self.activeCavasIndex], () => {
        //  self.canvas.renderAll()
        // })
        self.canvasJson[self.activeCavasIndex].forEach(val => {
          self.canvas.add(val)
          self.canvas.renderAll()
        });
        
      } */

      // self.canvas =self.canvas
    },
    getMark() {
      //this.currentStudentIndex当前学生索引//当前试题索引this.activeCavasIndex
      var coordArr = this.exerciseData.Group[this.activeCavasIndex].Items //试卷里的对号叉号坐标
      coordArr.forEach((val, i, array) => {
        if (val.ItemResult === 'YES') {
          val.imgUrl = require('@/assets/teacher/IntelligentCorrection/duihao.png')
        } else if (val.ItemResult === 'NO') {
          val.imgUrl = require('@/assets/teacher/IntelligentCorrection/chahao.png')
        }
        //画图

        fabric.Image.fromURL(val.imgUrl, (img) => {
          //  img.scaleToWidth('24')
          //         img.scaleToHeight('24')
          img.set({
            left: val.X,
            top: val.Y,
            hasControls: true, // 是否开启图层的控件
            borderColor: 'orange', // 图层控件边框的颜色
            scaleX: 0.5,
            scaleY: 0.5,
            ItemResult: val.ItemResult,
            MarkId: val.MarkId,
            selectable: false,
            selection: false,
          })

          // 添加对象
          this.canvas.add(img)
          this.canvas.bringForward(img) // 上移
        })
      })
      console.log(coordArr, 'baabab')
      this.canvas.requestRenderAll()
    },
    getStudentTodoCorrectExercise(studentid) {
      var obj = {
        studentid,
        customExerciseId: this.$route.query.customId,
      }
      this.$uwonhttp.post('/Customer/CustomExercise/GetStudentTodoCorrectExercise', obj).then((res) => {
        // this.studentsList=res.data.Data
        // console.log(this.studentsList);
        //  this.studentsList[0].StudentId

        // this.$emit('eventGetCanvasInfo', data,this.isActive)

        this.exerciseData = res.data.Data

        //试卷数组
        var exercise = this.exerciseData.PaperImgUrl.split(',')

        this.canvasList = []
        this.canvasJson = []
        this.canvasArr = []
        this.logArr = []
        this.imgSrc = []
        exercise.forEach((val, i) => {
          this.canvasList.push(val)
          this.canvasJson[i] = []
          this.canvasArr[i] = null
          this.imgSrc[i] = ''
          this.logArr[i] = []
        })
        // this.canvasJson = new Array(this.canvasList.length)
        // this.canvasArr = new Array(this.canvasList.length)
        this.activeCavasIndex = 0
        this.canvas = null
        // setTimeout(() => {
        this.initCanvas()

        // }, 500)
      })
    },

    //获取待（批改）的学生集合
    getTodoCorrectStudents() {
      // /Customer/CustomExercise/GetTodoCorrectStudents
      // /Customer/CustomExercise/GetAllStudent
      // :key="componentKey"
      this.$uwonhttp.post('/Customer/CustomExercise/GetTodoCorrectStudents', this.parameterObj).then((res) => {
        this.studentsList = res.data.Data

        if (this.studentsList.length > 0) {
          this.studentsList.forEach((val, i) => {
            this.studentsList[i].compelete = false
            // this.compeleteList[i]=false
          })

          this.student_length = this.studentsList.length
        }
      })
    },
    evluateClose() {
      this.isopenEvaluate = false //关闭弹窗
    },
  },
}
</script>
<style lang="less" scoped>
@font-color: #4d5753;
.correction-wrapper {
  width: 100%;
  font-size: 14px;
  text-align: center;
  position: relative;
  height: 100vh;
  h5 {
    color: @font-color;
  }
  .progress-info {
    //   text-align: center;
    margin-top: 20px;
    span {
      margin: 0 15px;
      .ant-progress-line {
        width: 348px;
      }
      /deep/ .ant-progress-inner {
        //   提高优先级
        background-color: #ddd;
      }
    }
  }

  .student-info {
    overflow: hidden;
    width: 80%;
    margin: 24px auto;
    // display: flex;
    margin-bottom: 16px;
    background-color: #fff;
    border-radius: 5px;
    // 学生详情
  }
  .correct-info {
    width: 80%;
    background: #fff;
    // height: 590px;
    border-radius: 5px;
    margin: 0 auto;
    text-align: center;
    position: relative;
    padding: 14px 0;

    display: flex;
    justify-content: center;
    align-items: center;
    .canvas {
      width: 800px;
      height: 590px;
      text-align: center;
      vertical-align: middle;
    }
    .compelete {
      height: 400px;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      img {
        width: 180px;
      }
      p {
        margin-top: 24px;
      }
      .compelete-color {
        color: #b7b7b7;
      }
    }

    .tools {
      display: flex;
      align-items: center;
      flex-direction: column;
      justify-content: space-around;
      position: absolute;
      right: 30px;
      // top: 40px;
      height: 420px;
      li {
        color: #737976;
      }

      img {
        width: 38px;
        cursor: pointer;
      }
    }
  }
  .submitinfo {
    width: 80%;
    margin: 20px auto 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    .pages {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-content: center;
      line-height: 2;
    }
    .btn {
      background: #68bb97;
      color: #fff;
      text-align: center;
      padding: 4px 2em;
      border-radius: 1em;
      font-size: 16px;
      line-height: 2;
      cursor: pointer;

      &.gray {
        background: #b5b5b5;
      }
      &.small {
        padding: 0 7px;
        font-size: 14px;
        border-radius: 4px;
        margin-left: 1em;
      }
    }
  }
}
</style>
