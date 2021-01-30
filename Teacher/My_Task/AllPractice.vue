<template>
  <div>
    <div class="example" v-if="load">
      <a-spin />
    </div>
    <!-- 搜索框 -->
    <div id="all-practice">
      <div class="search-all m-b">
        <span class="m-r ">章节筛选:</span>
        <a-cascader
          :options="options"
          :show-search="{ filter }"
          placeholder="全部练习"
          change-on-select
          @change="onChange"
          size="large"
          :field-names="{label:'ChapterName',value:'ChapterId',children:'Firsts'}"
        />
        <!-- <span class="search" @click="searchPaper">搜索</span> -->
      </div>
      <!-- 导航筛选 -->
      <div class="nav-bar">
        <!-- <p><span style="width:100px">练习属性：</span><span :class="{ 'color1' : paperTypeblue == '1' }" @click="changePaperType(1)">全区练习</span>
          <span :class="{ 'color1' : paperTypeblue == '2' }" @click="changePaperType(2)">班级练习</span></p> -->
        <p><span style="width:100px;margin-left: -1px;">发布状态: </span><span :class="{ 'color1' : racticeblue == '1' }" @click="changeRactice(0)">全部</span>
          <span style="margin-left: 25px;" :class="{ 'color1' : racticeblue == '2' }" @click="changeRactice(2)">待发布</span>
          <span :class="{ 'color1' : racticeblue == '3' }" @click="changeRactice(1)">已发布</span></p>
        <!-- <p v-show="practiceDispaly">练习状态：<span :class="{ 'color1' : practiceblue == '1' }" @click="changePractice(0)">全部</span>
          <span :class="{ 'color1' : practiceblue == '2' }" @click="changePractice(1)">做答中</span>
          <span :class="{ 'color1' : practiceblue == '3' }" @click="changePractice(2)">已截至</span>
          <span :class="{ 'color1' : practiceblue == '4' }" @click="changePractice(3)">待批阅</span>
          <span :class="{ 'color1' : practiceblue == '5' }" @click="changePractice(4)">已完成</span></p> -->
        <p v-if="sortShow"><span style="width:100px;margin-left: 24px;margin-right: 28px;">排序：</span><span :class="{ 'color1' : orderByblue == '1' }" @click="changeOrderBy(0)">推送时间</span>
          <!-- <span :class="{ 'color1' : orderByblue == '2' }" @click="changeOrderBy(1)">难度</span> -->
          <span :class="{ 'color1' : orderByblue == '3' }" @click="changeOrderBy(2)">正确率</span>
          <span :class="{ 'color1' : orderByblue == '4' }" @click="changeOrderBy(3)">做卷人数</span></p>
      </div>
      <!-- 选择班级 -->
      <div class="class-option">
        <span class="class-col" :class="{ 'color' : red == s.ClassId }" @click="change(s.ClassId)" v-for="s in gradeList" :key="s.ClassId">{{ s.ClassName }}</span>
      </div>
      <!-- 一班 -->
      <div style="display: flex;flex-wrap: wrap;">
        <div class="content-all clearfix" v-for="s in paperList" :key="s.PaId">
          <!-- 基本信息 -->
          <div class="fl content-main">
            <p>{{ s.PaperTitle }}<span v-if="s.TipText==='已发布'" class="goery">{{ s.TipText }}</span><span v-if="s.TipText==='待发布'" class="black">{{ s.TipText }}</span></p>
            <p>所属单元：{{ s.ChapterName }}</p>
            <p>{{ s.GradeName }} 出题人： {{ s.TrueName }} &nbsp; &nbsp; &nbsp;<span v-show="s.Vid!=null"><a-icon type="play-circle" /></span></p>
            <!-- 截至时间：{{ s.EndTime }} -->
            <p>推送时间：{{ s.CreatedTime }}</p>
          </div>
          <!-- <p class="area">{{ s.PaperType===0?'区':'班' }}</p> -->
          <p class="star" v-show="s.PaperType!=0"><a-rate v-model="s.DifficultyStar" :count="s.DifficultyStar" disabled /></p>
          <p class="states" v-show="s.PaperType!=0">{{ s.PracticeStatesText }}</p>
          <!-- 推送时间 -->
          <div class="fg changeTime">
            <p>{{ s.ClassName }}</p>
            <p><span @click="ClickPaper('info',s.PaId,s.ChapterId,'area')" v-if="s.PracticeStates===1 || s.PracticeStates===3 || s.TipText === '已发布'">检查作业</span>
              <!-- s.PracticeStates===2 || s.PracticeStates===4 && s.TipText === '待发布' -->
              <span @click="seePaper('info',s.PaId,s.ChapterId,s.TipText)" v-if="s.TipText === '待发布'">查看</span>
              <span @click="showRevokeModal(s.PaId)" v-if="s.ReleaseStates===2 && s.PaperType !==0">撤销发布</span>
              <span @click="ClickPaper('Correcting',s.PaId,s.ChapterId)" v-if="s.PracticeStates===3">去批改</span>
              <span style="backgroundColor:#A2C240" @click="showModal(s.CreatedTime,s.PaId)" v-if="s.ReleaseStates===2 && s.PaperType !==0 || s.TipText === '待发布'">修改推送时间</span>
              <a-modal
                title="设置发布信息"
                okText="发布"
                cancelText="取消"
                v-model="visible"
                @ok="hideModal(s.PaId)"
                :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.1)'}"
                centered>
                <p style="marginBottom: 12px;"><span>预计推送时间:</span> &nbsp;&nbsp;&nbsp;<span>{{ pushTime }}</span></p>
                <div>
                  修改推送时间：
                  <a-date-picker
                    :showTime="{ format: 'HH:mm'}"
                    format="YYYY-MM-DD HH:mm"
                    @change="TimeChange"
                    :disabledDate="disabledDate"
                  />
                  <br>
                  <!-- <br>
                  修改截止时间：
                  <a-date-picker
                    :showTime="{ format: 'HH:mm'}"
                    format="YYYY-MM-DD HH:mm"
                    @change="endTime"
                    :disabledDate="disabledDate"
                  /> -->
                </div>
              </a-modal>
            </p>
          </div>
          <!-- 做卷人数 -->
          <div class="make-volume" v-if="s.TipText==='待发布'? false : true">
            <!-- <div ref="Statistics" class="Statistics">
            </div> -->
            <a-progress type="circle" :percent="(s.DoCount/s.TotalCount)*100" height="100px" :width="45" :format="() => s.DoCount+'/'+s.TotalCount">
            </a-progress>
            <p>做卷人数</p>
          </div>
          <!-- 正确率 -->
          <div class="accuracy" v-if="s.TipText==='待发布'? false : true">
            <a-progress
              type="circle"
              :percent="s.Accuracy"
              height="100px"
              :width="45"
              :format="() => { if(s.Accuracy === 100) { return '100%'} else { return s.Accuracy+ '%'} }"
            />
            <p>正确率</p>
          </div>

          <div>
            <a-modal
              title="撤销发布"
              :visible="rvisible"
              :confirm-loading="confirmLoading"
              @ok="handleOk()"
              @cancel="handleCancel"
              centered
            >
              <p>{{ ModalText }}</p>
            </a-modal>
          </div>
        </div>
      </div>
      <!-- 分页 -->
      <div class="paging">
        <a-pagination :default-current="pageNo" :total="this.pageTotal" @change="onPaging" hideOnSinglePage />
      </div>
    </div>
    <p class="footer">Copyright©上海有我科技有限公司，All Rights Reserved</p>
    <!-- <footerLogo :heightPage="heightPage"></footerLogo> -->
  </div>
</template>

<script>
import '@/utils/utils.less'
import echarts from 'echarts'
import moment from 'moment'
import footerLogo from '@/components/newFooterLogo/newFooterLogo'
export default {
  components: {
    footerLogo
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    this.getGradeList()
    this.getGradeKnowledge()
    this.$nextTick(() => {
      // this.ring()
    })
  },
  mounted () {
    // this.ring()
  },
  updated () {
    this.heightPage = document.getElementById('all-practice').offsetHeight
  },
  data () {
    return {
      heightPage: 0,
      options: [
        {
          value: 'zhejiang',
          label: 'Zhejiang',
          children: [
            {
              value: 'hangzhou',
              label: 'Hangzhou',
              children: [
                {
                  value: 'xihu',
                  label: 'West Lake'
                },
                {
                  value: 'xiasha',
                  label: 'Xia Sha',
                  disabled: true
                }
              ]
            }
          ]
        },
        {
          value: 'jiangsu',
          label: 'Jiangsu',
          children: [
            {
              value: 'nanjing',
              label: 'Nanjing',
              children: [
                {
                  value: 'zhonghuamen',
                  label: 'Zhong Hua men'
                }
              ]
            }
          ]
        }
      ],
      red: '1',
      paperTypeblue: '1',
      racticeblue: '1',
      practiceblue: '1',
      orderByblue: '1',
      // 控制模态框的显示
      visible: false,
      width: 100,
      userId: '',
      gradeList: '',
      paperList: [],
      load: true,
      chapterId: '',
      paperType: 1,
      releaseStates: 0,
      practiceStates: 0,
      orderById: 0,
      practiceDispaly: false,
      pageTotal: 0,
      pageNo: 1,
      rvisible: false,
      ModalText: '撤销发布后，该练习将被删除！',
      confirmLoading: false,
      paperId: '',
      success: '',
      // 待发布显隐
      sortShow: true,
      // 推送时间
      pushTime: '',
      // 修改推送时间的试卷id
      modifyPaperId: '',
      // 截止时间
      selectEndTime: '',
      // 修改的时间
      modifyTime: ''
    }
  },
  methods: {
    // 不可选择时间
    disabledDate (current) {
      // 不可选过去时间
      return current < moment().add(-1, 'd')
      // 不可选未来时间
      // return current && current > Date.now()
    },
    getGradeList () {
      // debugger
      this.$uwonhttp.get('/Class/TeacherClassManager/GetTeachClass', { params: { UserId: this.userId } }).then(resJson => {
        this.gradeList = resJson.data.Data
        if (this.gradeList.length < 1) {
          this.load = false
          confirm('您还未带领班级')
        } else {
          this.red = this.gradeList[0].ClassId
          this.getPaperView()
        }
      })
    },
    // 获取试卷信息
    getPaperView () {
      debugger
      this.$uwonhttp
        .post('/Paper/TeacherPaper/GetTeacherPaperListPractice', {
          ChapterId: this.chapterId,
          PaperType: 1,
          ReleaseStates: this.releaseStates,
          PracticeStates: this.practiceStates,
          OrderById: this.orderById,
          ClassId: this.red,
          UserId: this.userId,
          PageNo: this.pageNo
        })
        .then(resJson => {
          this.load = false
          this.paperList = resJson.data.Data.Items
          console.log('试卷信息---', this.paperList)
          this.pageTotal = resJson.data.Data.TotalItems
          this.chapterId = ''
        })
    },
    // 获取年级知识点
    getGradeKnowledge (id) {
      this.$http.post('/Paper/Exam_SubjectChapter/GetChapterTreeByUserId', { userId: this.userId }).then(res => {
        this.options = ''
        this.options = res.Data[0].Firsts
      })
    },
    // 图表处理
    ring () {
      // 实例化eacharts
      const myChart = echarts.init(this.$refs.Statistics)

      // 绘制环形图
      myChart.setOption({
        tooltip: {
          trigger: 'item',
          formatter: ''
        },
        legend: {
          orient: 'vertical',
          left: 10,
          data: []
        },
        series: [
          {
            name: '',
            type: 'pie',
            radius: ['50%', '70%'],
            avoidLabelOverlap: false,
            label: {
              show: false,
              position: 'center'
            },
            emphasis: {
              label: {
                show: false,
                fontSize: '12',
                fontWeight: 'bold'
              }
            },
            labelLine: {
              show: false
            },
            data: [
              { value: '28', name: '做卷人数' },
              { value: '7', name: '未做' }
            ]
          }
        ]
      })
    },
    // 联级选择框，选中之后回调
    onChange (value, selectedOptions) {
      debugger
      if (value.length === 1) {
        this.chapterId = value[0]
      } else if (value.length === 2) {
        this.chapterId = value[0] + ',' + value[1]
      }
      this.getPaperView()
    },
    filter (inputValue, path) {
      return path.some(option => option.label.toLowerCase().indexOf(inputValue.toLowerCase()) > -1)
    },
    // 获取当前分页
    onPaging (pageNum) {
      console.log('页码：' + pageNum)
      this.pageNo = pageNum
      this.load = true
      this.getPaperView()
    },
    // 显示修改推送时间
    showModal (time, id) {
      this.visible = true
      this.pushTime = time
      this.modifyPaperId = id
      console.log('试卷ID---', id)
    },
    // 隐藏推送时间框
    hideModal (id) {
      this.visible = false
      // debugger
      this.$uwonhttp.post('/Paper/TeacherPaper/UpdatePaperPublishTime', { teacherId: this.userId, paperId: this.modifyPaperId, beginTime: this.modifyTime, classId: this.red }).then(res => {
        console.log('修改推送时间---', res)
        if (res.data.Success) {
          this.$message.success('成功修改推送时间')
          this.getPaperView()
        }
      })
    },
    // 获取修改推送时间
    TimeChange (data, dataString) {
      console.log(data, dataString)
      this.modifyTime = dataString
    },
    // 获取截止推送时间
    endTime (data, dataString) {
      console.log('截止时间---', data, dataString)
      this.selectEndTime = dataString
    },
    // 跳转班级数据
    toClassData () {
      this.$router.push({ path: '/Teacher/My_Task/InspectionWorkData',
        query: {
          paperId: '110'
        } })
    },
    change (classId) {
      console.log('班级id --', classId)
      this.pageNo = 1
      this.load = true
      this.red = classId
      this.getPaperView()
    },
    changePaperType (id) {
      // 试卷类型 区/班
      this.paperType = id
      if (id === 1) {
        this.paperTypeblue = 1
        this.practiceDispaly = false
      } else {
        this.paperTypeblue = 2
        this.practiceDispaly = true
      }
      this.load = true
      this.getPaperView()
    },
    changeRactice (id) {
      // 发布状态
      this.releaseStates = id
      this.pageNo = 1
      this.getPaperView()
      if (id === 0) {
        this.racticeblue = 1
        this.sortShow = true
      } else if (id === 2) {
        this.racticeblue = 2
        this.sortShow = false
      } else if (id === 1) {
        this.racticeblue = 3
        this.sortShow = true
      }
    },
    changeOrderBy (id) {
      // 排序
      // debugger
      this.orderById = id
      this.pageNo = 1
      this.getPaperView()
      if (id === 0) {
        this.orderByblue = 1
      } else if (id === 1) {
        this.orderByblue = 2
      } else if (id === 2) {
        this.orderByblue = 3
      } else if (id === 3) {
        this.orderByblue = 4
      }
    },
    changePractice (id) {
      // 练习状态
      this.practiceStates = id
      if (id === 0) {
        this.practiceblue = 1
      } else if (id === 1) {
        this.practiceblue = 2
      } else if (id === 2) {
        this.practiceblue = 3
      } else if (id === 3) {
        this.practiceblue = 4
      } else if (id === 4) {
        this.practiceblue = 5
      }
    },
    // 查看试卷
    seePaper (er, id) {
      this.$router.push({ path: '/Teacher/My_Task/ViewExercises', query: { paperId: id } })
    },
    ClickPaper (title, id, chapterNum) {
      // 检查作业 查看按钮
      // debugger
      console.log('章节 -- 名称', chapterNum)
      if (title === 'info') {
        this.$router.push({ path: '/Teacher/My_Task/InspectionWorkData',
          query: {
            paperId: id,
            classId: this.red,
            chapterNum: chapterNum,
            area: 'area'
          } })
      } else if (title === 'Correcting') {
        // 去批改
        this.$router.push({ path: 'Teacher/My_Task/InspectionWork/BatchModify',
          query: {
            paperId: id,
            classId: this.red
          } })
      }
    },
    searchPaper () {
      this.load = true
      this.getPaperView()
    },
    showRevokeModal (id) {
      this.paperId = id
      this.rvisible = true
    },
    handleOk () {
      this.ModalText = '正在撤销...'
      this.confirmLoading = true
      setTimeout(() => {
        this.rvisible = false
        this.confirmLoading = false
      }, 2000)
      this.$http
        .post('/Paper/Exam_Paper/RevokePublishPaper', {
          paperId: this.paperId,
          classId: this.classId
        })
        .then(resJson => {
          this.load = true
          this.getPaperView()
        })
    },
    handleCancel (e) {
      this.rvisible = false
    }
  }
}

</script>

<style lang="less" scoped>
.color1 {
  color: #68BB97;
}
.m-r {
  margin-right: 30px;
}
.m-b {
  margin-bottom: 35px;
}
.footer {
  margin-top: 80px;
}
/deep/.ant-cascader-picker {
  border-radius: 50px;
}
/deep/.ant-cascader-picker-show-search .ant-cascader-input.ant-input {
      width: 289px;
      border-radius: 50px;
      text-indent: 15px;
}
// load图标
  .example {
    text-align: center;
  //   background: rgba(0, 0, 0, 0.05);
  //   border-radius: 4px;
  //   margin-bottom: 20px;
  //   padding: 30px 50%;
  //   margin: 20px 0;
  }
  // 搜索
  .search-all {
     .search {
      display: inline-block;
      width: 60px;
      height: 40px;
      line-height: 40px;
      margin-left: 10px;
      text-align: center;
      border-radius: 24px;
      background-color: #ccc;
      cursor: pointer;
  }
  }
  // 导航栏
  .nav-bar {
    // border-bottom: 2px solid #ccc;
    span {
      // margin-left: 20px;
      margin-right: 33px;
      cursor: pointer;
    }
  }
  .class-col {
    color: #B5B5B5;
    // background-color: #F4F4F4;
    border: 1px solid #B5B5B5;
  }
  // 班级选择
  .class-option {
    span {
      display: inline-block;
      width: 100px;
      height: 40px;
      line-height: 40px;
      margin-top: 20px;
      margin-right: 35px;
      text-align: center;
      border-radius: 20px;
      cursor: pointer;
    }
  .color {
    border: none;
    color: #fff;
    background-color: #68BB97;
    }
  }
  // 内容
  .content-all {
    display: block;
    // height: 160px;
    margin-top: 20px;
    padding: 10px 15px 5px 15px;
    // padding: 25px 15px;
    // border: 1px solid grey;
    border-radius: 5px;
    position: relative;
    width: 48%;
    margin-right: 20px;
    background-color: #fff;
  }
  // 主要内容
  .content-main {
    // margin-top: 6px;
    p {
      margin-top: 5px;
    }
    p:nth-child(1) {
      font-weight: 700;
      font-size: 18px;
      span {
        // display: inline-block;
        // width: 64px;
        padding: 4px 7px;
        line-height: 24px;
        margin-left: 14px;
        font-size: 12px;
        font-weight: 500px;
        font-size: 10px;
        text-align: center;
        border-radius: 50px;
      }
    }
    p:nth-child(3) {
      margin-bottom: 20px;
    }
  }
  // 修改推送时间
  .changeTime {
    line-height: 3;
    margin-top: 35px;
    margin-right: 0px;
    // text-align: center;
    p:nth-child(1) {
       text-align: right;
       margin-bottom: 48px;
       margin-top: -20px;
       margin-right: 40px;
    }
    p:nth-child(2) {
      height: 36px;
      line-height: 36px;
      font-size: 14px;
      text-align: center;
      border-radius: 5px;
      cursor: pointer;
      span {
        height: 40px;
        line-height: 40px;
        padding: 4px 7px;
        margin-left: 30px;
        font-size: 14px;
        text-align: center;
        // display: inline-block;
        // width: 100px;
        color: #fff;
        background-color: #68bb97;
        border-radius: 6px;
      }
    }
  }
  .goery{
    color: white;
    background-color: gray;
  }
  .black{
    color: white;
    background-color: #4D5753;
  }
  // 区域
  .area {
    position: absolute;
    left: 300px;
    top: 107px;
    font-size: 20px;
    color: blue;
  }
  // 星星
  .star {
      position: absolute;
      left: 270px;
      top: 80px;
      font-size: 20px;
      color: blue;
  }
  // 状态
  .states {
      position: absolute;
      right: 1%;
      top: 5px;
      font-size: 10px;
      color: orange;
  }
  // 做卷人数
  .make-volume {
    position: absolute;
    // left: 430px;
    left: 50%;
    top: 60px;
    text-align: center;
  }
  // 做卷人数图表
  .Statistics {
    width: 50px;
  }
  // 正确率
  .accuracy {
    position: absolute;
    // left: 500px;
    left: 61%;
    top: 60px;
    text-align: center;
  }
  // 分页
  .paging {
    margin-top: 40px;
    text-align: center;
     /deep/.ant-pagination-item:focus, .ant-pagination-item:hover {
      border-color: #68BB97;
      border-color: #68BB97;
    }
    /deep/.ant-pagination-item-active a {
      color: #000;
      border-color: #68BB97;
      background-color: #68BB97;
    }
    /deep/.ant-pagination-item-active {
      border-color: #68BB97;
    }
  }

</style>
