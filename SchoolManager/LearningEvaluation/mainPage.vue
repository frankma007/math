<template>
  <div>
    <div>
      <div class="stu-analusis">
        <!-- 智能分析 -->
        <div class="int-analysis">
          <span style="width:100%;">
            <span class="mainMessage">{{ schoolName }}-智能分析 </span>
            <span style="float:right;">
              <a-select default-value="1" style="width: 100px" @change="getGradeId">
                <a-select-option value="1">
                  一年级
                </a-select-option>
                <a-select-option value="2">
                  二年级
                </a-select-option>
                <a-select-option value="3">
                  三年级
                </a-select-option>
                <a-select-option value="4">
                  四年级
                </a-select-option>
                <a-select-option value="5">
                  五年级
                </a-select-option>
              </a-select>
            </span>
          </span>
          <br>
          <br>
          <div>
            <span>
              <p class="mainMessage"><img src="@/assets/finemicro/红点.png" alt="">本校正确率
                <span class="stu-ana-col">
                  本届&nbsp;{{ mainPageMessage.NowAccuracy }}&nbsp;&nbsp;上届&nbsp;{{ mainPageMessage.PastAccuracy }}
                </span>
              </p>
            </span>
          </div>
          <br>
          <div >
            <span>
              <p class="mainMessage"><img src="@/assets/finemicro/红点.png" alt="">本校完成率
                <span class="stu-ana-col">
                  本届&nbsp;{{ mainPageMessage.NowFinish }}&nbsp;&nbsp;上届&nbsp;{{ mainPageMessage.PastFinish }}
                </span>
              </p>
            </span>
          </div>
          <br>
          <div >
            <span>
              <p class="mainMessage"><img src="@/assets/finemicro/红点.png" alt="">知识点统计（与上届相比）
                <span class="stu-ana-col">
                  进步&nbsp;{{ mainPageMessage.Forward }}个&nbsp;&nbsp;退步&nbsp;{{ mainPageMessage.BackOff }}个&nbsp;&nbsp;持平&nbsp;{{ mainPageMessage.Fair }}个
                </span>
              </p>
            </span>
          </div>
        </div>
        <!-- 抽屉 -->
        <div class="honor">
          <div class="cur" @click="handleHonorMedal">
            <div>
              <img src="@/assets/teacher/编组 9.png" alt="">   <br><span class="drawName"> 知识点分析</span>
            </div>
          </div>
          <div class="cur" @click="handleChapter">
            <div>
              <img src="@/assets/schoolmamager/编组 2.png" alt=""><br><span class="drawName">教师分析</span>
            </div>
          </div>

          <div class="cur" @click="termReport">
            <div>
              <img src="@/assets/schoolmamager/icon-微课.png" alt=""><br><span class="drawName">微课视频分析</span>
            </div>
          </div>
          <!-- toSchoolVip -->
          <div class="cur"  @click="toWeekly">
            <div>

              <img style="width:55px;" src="@/assets/weekly/icon_xqbg.png" alt=""><br><span class="drawName" >学期周报</span>
              <!-- <img src="@/assets/weekly/icon_xqbg.png" alt=""><br><span class="drawName"><img src="@/assets/vip/vip-ana.png" alt=""></span> -->

            </div>
          </div>
          <!-- 知识点分析 -->
          <a-drawer
            placement="right"
            :closable="false"
            :visible="chapterReport"
            @close="onCloseChapterReport"
            width="1170"
            :headerStyle="{ textAlign: 'center'}"
            :drawerStyle="{ backgroundColor: '#F7F8F8' }"
            :bodyStyle="{ padding: '50px'}"
          >
            <div class="chapter-report">
              <p class="title clearfix">
                <span>知识点分析</span>
                <span class="fg">
                </span>&nbsp;&nbsp;&nbsp;
                <span class="fg gradeSelect">
                  <a-select class="fg" style="width: 120px" @change="handleChangeKnoweldge" default-value="1_1">
                    <a-select-option v-for="(item,indx) in knowGrade" :key="indx" :value="item.Grade">
                      {{ item.GradeName }}
                    </a-select-option>
                  </a-select>
                </span>
                <span class="fg chapter-select">
                  <a-select class="fg" style="width: 150px" @change="handleChapterChoice" placeholder="章节">
                    <a-select-option v-for="(item,index) in gradeChapter" :key="index" :value="item.ChapterId" >
                      {{ item.ChapterName }}
                    </a-select-option>
                  </a-select>
                </span>
              </p>
              <p class="chart-operation">
                <span class="btn-col" :class="{ 'bgc-color': color === '1'}" @click="change">知识点</span>
                <span class="btn-col" :class="{ 'bgc-color': color === '2'}" @click="change1">学习水平</span>
                <span class="btn-col" :class="{ 'bgc-color': color === '3'}" @click="change2">题型模板</span></p>
              <!-- 知识点 -->
              <div v-show="color === '1'">
                <div ref="KnowledgePoints" class="knowledge-points"></div>
                <p>本届数据详情（正确率：%）</p>
                <a-table
                  :columns="columns1"
                  :data-source="knowledgeData"
                  :pagination="false"
                  bordered
                >
                </a-table>
              </div>
              <!-- 学习水平 -->
              <div v-show="color === '2'">
                <div ref="studyLevel" class="study-level"></div>
              </div>
              <!-- 题型模板 -->
              <div v-show="color === '3'">
                <div ref="questionType" class="question-type"></div>
              </div>
            </div>
          </a-drawer>

          <!-- 教师分析 -->
          <a-drawer
            :title="className"
            placement="right"
            :closable="false"
            :visible="VisibleTeacher"
            @close="CloseTeacher"
            width="1170"
            :headerStyle="{ textAlign: 'center'}"
            :drawerStyle="{ backgroundColor: '#F7F8F8' }"
            :bodyStyle="{ padding: '50px'}"
          >
            <div class="teacherTop">
              <div class="teacherGrade">
                <a-select style="width: 120px" @change="getTeacherGrade" placeholder="选择年级" defalut-value="1_1">
                  <a-select-option v-for="(item,indx) in teacherGrade" :key="indx" :value="item.Grade" >
                    {{ item.GradeName }}
                  </a-select-option>
                </a-select>
              </div>
              <div class="teacherGrade" >
                <a-cascader
                  :default-value="options.ChapterId"
                  @change="getTeacherChapter"
                  :options="options"
                  :field-names="{label:'ChapterName',value:'ChapterId',children:'Second'}"
                  placeholder="选择章节"
                />
              </div>
            </div>
            <a-divider></a-divider>
            <div style="width:100%">
              <div id="teacherEchart" ref="teacherEchart">

              </div>
              <br>
              <p id="teacherClass">{{ teacherTitle }}</p>
              <div id="teacherDetail" ref="teacherDetail">

              </div>
            </div>
          </a-drawer>

          <!-- 微课视频分析 -->
          <a-drawer
            :title="className"
            placement="right"
            :closable="false"
            :visible="VisibleTask"
            @close="onClose"
            width="1170"
            :headerStyle="{ textAlign: 'center'}"
            :drawerStyle="{ backgroundColor: '#F7F8F8' }"
            :bodyStyle="{ padding: '50px'}"
          >
            <p style="text-align:center;font-size: 20px;font-family: PingFangSC-Medium, PingFang SC;font-weight: 500;color: #373737;">微课视频分析</p>
            <a-divider></a-divider>
            <div class="micorCount">微课视频总数<br><span style="font-size:20px;font-weight:600;">{{ microCount }}</span></div>
            <div class="micVideo">

              <div class="micVideo" ref="micVideo">
              </div>
            </div>
            <p v-if="microLoad" style="text-align:center;"><a-spin></a-spin></p>
            <div id="bottom" v-if="!microLoad">
              <div class="left">
                <a-collapse :bordered="false" expand-icon-position="right">
                  <a-collapse-panel :header="item.ChapterName +'  （'+item.Count+'个视频）'" :style="{backgroundColor:'white'}" v-for="(item,index) in allChapter" :key="index" >
                    <div class="imgs" v-for="(m,ind) in item.Sections" :key="ind" @click="getChatpterMic(index,ind)">
                      <span>{{ m.SectionName }}</span>
                      <span class="Content">
                        <img style="margin-top:-3px;" src="@/assets/schoolmamager/sicon-微课.png" alt="">&nbsp;&nbsp;{{ m.Count }}
                      </span>
                    </div>
                  </a-collapse-panel>
                </a-collapse>
              </div>
              <div id="micRight">
                <div id="allMicro">
                  {{ checkChapterName }}
                  <a-divider></a-divider>
                  <div class="showMicros" v-for="(item,index) in knowMicro" :key="index">
                    <div class="video" :style="{ backgroundImage:'url(' + item.CoverUrl + ')',backgroundRepeat:'no-repeat',backgroundSize:'100% 100%' }">
                      <img class="microImg" src="@/assets/schoolmamager/icon-微课分析.png" alt="" @click="toMicoDetail(item.VideoId)">
                    </div>

                    <div class="videoDetail">
                      <span class="chapter">{{ item.VideoName }}</span>
                      <span class="time" v-if="item.VideoDuration"><img src="@/assets/schoolmamager/时长.png" alt="">{{ item.VideoDuration }}</span>
                      <span class="watchCount"><img src="@/assets/schoolmamager/播放量.png" alt="">{{ item.VideoPlay }}次</span>
                    </div>

                    <div class="videoTeacher">
                      <span class="rate">
                        <span v-for="(m,ind) in item.Score" :key="ind">★&nbsp;</span>
                      </span>
                      <span class="teacherName">
                        授课老师：{{ item.TrueName }}
                      </span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </a-drawer>
        </div>
      </div>
      <!-- 正确率统计 -->
      <div class="Accuracy">
        <div class="p">
          <div class="learnTitle">本校本学期练习正确率趋势</div>

          <p class="bottomMainMessage">{{ gradeName }}</p>
        </div>

        <div style="text-align:right;">
          <a-select default-value="" style="width: 120px" @change="changeTimes">
            <a-select-option value="">
              近一周
            </a-select-option>
            <a-select-option value="month">
              近一月
            </a-select-option>
            <a-select-option value="xq">
              本学期
            </a-select-option>
          </a-select>
        </div>
        <div class="situationAnalysis" ref="situationAnalysis"></div>
      </div>
      <p class="footer">Copyright©上海有我科技有限公司，All Rights Reserved</p>
    </div>
  </div>

</template>

<script>
import '@/utils/utils.less'
import echarts from 'echarts'
import TrendVue from '@/components/ChartCard/Trend.vue'
const columns = [
]
var columns1 = [

]

export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    this.IsVip = localStorage.getItem('IsVip')
    this.getMainMesage()
    this.getUserSchoolName()
  },
  mounted () {
    this.init()
  },
  data () {
    return {
      IsVip: 0,
      changeTime: 'week',
      // 校名
      schoolName: '',
      // 首页所有数据
      mainPageMessage: '',
      // 年级id
      gradeId: '1',
      gradeName: '一年级',
      // 知识点分析抽屉
      chapterReport: false,
      // 微课视频分析
      VisibleTask: false,
      // 教师分析
      VisibleTeacher: false,
      // 教师分析章节
      options: [],
      // 老师班级详细情况标题
      teacherTitle: '',
      // 知识点分析年级
      knowGrade: '',
      // 知识点分析年级Id
      gradeNum: '1_1',
      // 章节下拉框数据
      gradeChapter: '',
      // 选中的年级下的所有单元Id字符串
      checkChapterId: '',
      // 选中的年级下的所有单元Id字符串列表
      checkChapterIds: '',
      // 教师分析年级
      teacherGrade: '',
      // 教师年级
      teacGrade: '',
      // 教师选中的章节
      teacherChapter: '',
      // 点击的章节名字
      checkChapterName: '',
      // 微课视频量总数
      microCount: '',
      // 微课分析年级所有章节
      allChapter: '',
      // 该章节下知识点的所有微课视频
      knowMicro: '',
      microLoad: false,

      columns,
      // 数据详情
      columns1,
      data: [],
      // 知识点数据
      knowledgeData: [],
      // 章节选择图表
      color: '1',
      // 知识点
      // options: [],
      // 教师id
      userId: '',
      // 班级
      classInfo: [],
      stuChapterChoice: [],
      // 班级名称
      className: '',
      // 默认的班级id
      classId: '',
      // 章节id
      chapterId: '',
      // 班级做卷信息
      classMakeSubject: {},
      // 学生分析数据
      studentAnalysis: {},
      nameSchool: '',
      defalut: '1',
      load: false
    }
  },
  watch: {
    gradeId: function (newval, oldval) {
      if (newval == 1) {
        this.gradeName = '一年级'
      } else if (newval == 2) {
        this.gradeName = '二年级'
      } else if (newval == 3) {
        this.gradeName = '三年级'
      } else if (newval == 4) {
        this.gradeName = '四年级'
      } else if (newval == 5) {
        this.gradeName = '五年级'
      }
    }
  },

  methods: {
    // 获取使用者学校名字
    getUserSchoolName () {
      this.$uwonhttp.post('/ManagerPersonalsController/ManagerPersonals/GetTeacherInfo', {
        userId: this.userId
      }).then(res => {
        console.log(res)
        this.schoolName = res.data.Data.SchoolName
      })
    },

    // 获取年级id
    getGradeId (val) {
      console.log(val)
      this.gradeId = val
      this.getMainMesage()
    },
    // 获取首页章节
    getMainMesage () {
      this.$uwonhttp.post('/Manager/ManagerAnalysis/AnalysisLearning', {
        GradeId: this.gradeId,
        Time: this.changeTime,
        UserId: this.userId
      }).then(res => {
        console.log('折线图---', res)
        var Data = res.data.Data
        this.mainPageMessage = Data
        const newAccuracy = Data.Line.map(item => {
          return item.NowPer
        })
        const pastAccuracy = Data.Line.map(item => {
          return item.PastPer
        })
        const xTitle = Data.Line.map(item => {
          return item.PaperTitle
        })
        const myChart = echarts.init(this.$refs.situationAnalysis)
        myChart.setOption({
          xAxis: {
            data: xTitle
          },
          series: [
            { data: newAccuracy },
            { data: pastAccuracy }
          ]
        })
      })
    },
    changeTimes (val) {
      this.changeTime = val
      this.$uwonhttp.post('/Manager/ManagerAnalysis/AnalysisLearning', {
        GradeId: this.gradeId,
        Time: this.changeTime,
        UserId: this.userId
      }).then(res => {
        var Data = res.data.Data
        this.mainPageMessage = Data
        const newAccuracy = Data.Line.map(item => {
          return item.NowPer
        })
        const pastAccuracy = Data.Line.map(item => {
          return item.PastPer
        })
        const xTitle = Data.Line.map(item => {
          return item.PaperTitle
        })
        const myChart = echarts.init(this.$refs.situationAnalysis)
        myChart.setOption({
          xAxis: {
            data: xTitle
          },
          series: [
            { data: newAccuracy },
            { data: pastAccuracy }
          ]
        })
      })
    },
    // 绘制首页图表
    init () {
      const myChart = echarts.init(this.$refs.situationAnalysis)
      myChart.setOption({
        title: {
          text: '(百分比：%)' // 正确率
        },

        dataZoom: [
          {
            start: 0, // 默认为0
            end: 100 - 1500 / 50, // 默认为100
            type: 'slider',
            show: true,
            xAxisIndex: [0],
            handleSize: 0, // 滑动条的 左右2个滑动条的大小
            height: 14, // 组件高度
            // left: '10%', // 左边的距离
            // right: '10%', // 右边的距离
            bottom: '-5', // 右边的距离
            border: 'none',
            borderColor: '#fff',
            fillerColor: '#B5BEB9',
            borderRadius: 7,
            backgroundColor: '#F8F8F8', // 两边未选中的滑动条区域的颜色
            showDataShadow: false, // 是否显示数据阴影 默认auto
            showDetail: false, // 即拖拽时候是否显示详细数值信息 默认true
            realtime: true, // 是否实时更新
            filterMode: 'filter'
          }
        ],
        legend: {
          data: ['本届正确率', '上届正确率']
        },
        // 提示框内容
        tooltip: {
          trigger: 'axis'
        },
        // 自定义颜色
        color: ['#FF8B7F', '#71B6A1', '#52B4FF'],
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: ['加载中', '加载中', '加载中', '加载中', '加载中', '加载中', '加载中', '加载中', '加载中']
        },
        yAxis: {
          type: 'value',
          axisLabel: {
            formatter: '{value}%'
          },
          triggerEvent: true,
          max: 100
        },
        series: [
          {
            name: '本届正确率',
            type: 'line',
            // stack: '总量',
            data: [0],
            itemStyle: {
              normal: {
                lineStyle: {
                  width: 1, // 设置线宽
                  type: 'solid' // 'dotted'虚线 'solid'实线
                }
              }
            },
            // 背景阴影渐变
            areaStyle: {
              opacity: 0.9,
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: '#ff8b7f'
              }])
            }
          },
          {
            name: '上届正确率',
            type: 'line',
            // stack: '总量',
            data: [0],
            itemStyle: {
              normal: {
                lineStyle: {
                  width: 1, // 设置线宽
                  type: 'dashed' // 'dotted'虚线 'solid'实线
                }
              }
            },
            areaStyle: {
              opacity: 0.9,
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: '#74b8a3'
              }])
            }
          }
        ]
      })
      // 设置图表随窗口大小变化
      window.addEventListener('resize', function () {
        myChart.resize()
      })
    },
    // 知识点分析图表
    get () {
      const knowledge = echarts.init(this.$refs.KnowledgePoints)
      knowledge.setOption({
        title: {
          text: ''
        },
        tooltip: {
        },
        legend: {
          bottom: 0,
          data: ['本届掌握情况', '上届掌握情况']
        },
        color: ['#ffc0b9', '#a1bcad'],
        radar: [
          {
            indicator: [
              { text: '单元一', max: 100 },
              { text: '单元二', max: 100 },
              { text: '单元三', max: 100 },
              { text: '单元四', max: 100 },
              { text: '单元五', max: 100 }
            ],
            radius: 150,
            startAngle: 90,
            splitNumber: 5,
            shape: 'circle',
            name: {
              formatter: '【{value}】',
              textStyle: {
                color: '#000'
              }
            },
            splitArea: {
              areaStyle: {
                color: ['#e8ebd6',
                  '#f6f5f8', '#e8ebd6',
                  '#f6f5f8', '#e8ebd6']
              }
            },
            axisLine: {
              lineStyle: {
                color: 'rgba(255, 255, 255, 0.5)'
              }
            },
            splitLine: {
              lineStyle: {
                color: 'rgba(255, 255, 255, 0.5)'
              }
            }
          }
        ],
        series: [
          {
            name: '雷达图',
            type: 'radar',
            areaStyle: {},
            emphasis: {
              lineStyle: {
                width: 4
              }
            },
            data: [
              {
                value: [0],
                name: '本届正确率',
                symbol: 'rect',
                symbolSize: 5,
                lineStyle: {
                  color: '#ffc0b9',
                  width: 4,
                  type: 'dashed'
                }
              },
              {
                value: [0],
                name: '上届正确率',
                areaStyle: {
                  color: '#a1bcad'
                }
              }
            ]
          }
        ]
      })
    },
    // 教师分析图标初始化
    openTeacher () {
      const teacherEchart = echarts.init(this.$refs.teacherEchart)
      teacherEchart.setOption({
        color: ['#c9eafd', '#7ec5a6', '#FDC958', '#F87175'],
        title: {
          subtext: '(正确率：%)',
          left: '5%'
        },
        tooltip: {
          formatter: function (params) {
            // params.name + '<br/>'  +
            return params.seriesName + ' : ' + params.value
          }
        },
        legend: {
          data: ['正确率', '完成率', '校正确率均值线', '区正确率均值线']
        },
        xAxis: [
          {
            type: 'category'
          }
        ],
        yAxis: [{
          type: 'value'
        }],
        series: [
          {
            name: '正确率',
            type: 'bar',
            data: [0],
            barWidth: '45',
            barGap: '0%'
          },
          {
            name: '完成率',
            type: 'bar',
            data: [0],
            barWidth: '45',
            barGap: '0%'
          },
          {
            name: '校正确率均值线',
            type: 'line',
            data: [],
            lineStyle: {
              width: 3,
              type: 'dashed'
            }
          },
          {
            name: '区正确率均值线',
            type: 'line',
            data: [],
            lineStyle: {
              width: 3,
              type: 'dashed'
            }
          }

        ]
      })
      const teacherDetail = echarts.init(this.$refs.teacherDetail)
      teacherDetail.setOption({
        title: {
          subtext: '(正确率：%)',
          left: '5%'
        },
        calculable: true,
        xAxis: [
          {
            type: 'category'
          }
        ],
        yAxis: [{
          type: 'value'
        }],
        series: [
          {
            name: '正确率',
            type: 'bar',
            barWidth: '45',
            barGap: '0%'
          },
          {
            name: '完成率',
            type: 'bar',
            barWidth: '45',
            barGap: '0%'
          }
        ]
      })
    },
    // 初始化微课视频分析
    openVideoMethod () {
      const micVideo = echarts.init(this.$refs.micVideo)
      micVideo.setOption({
        color: ['#F5CBB5', '#F09A64', '#9DAFF1', '#AEC6B3', '#FAD481', '#F69598', '#B7E0FF', '#81C1D0', '#C7EBBC', '#96D0B2'],
        tooltip: {
          trigger: 'item',
          formatter: '{b}: {c}'
        },
        series: [
          {
            type: 'pie',
            radius: ['40%', '75%'],
            label: {
              formatter: '● {b|{b}：}{c} ',
              borderWidth: 1,
              borderRadius: 4,
              rich: {
                b: {
                  color: 'black',
                  fontSize: 16,
                  lineHeight: 33
                },
                c: {
                  color: 'black',
                  lineHeight: 22,
                  align: 'center'
                }
              }
            }
          }
        ]
      })
    },
    // 知识点分析
    handleHonorMedal () {
      this.chapterReport = true
      this.getKnowGrade()
      this.$nextTick(() => {
        this.get()
      })
    },
    // 视频微课分析
    termReport () {
      this.VisibleTask = true
      this.$nextTick(() => {
        this.openVideoMethod()
      })
      this.reMicEcharts()
    },
    // vip
    toSchoolVip () {
      console.log('vip')
      this.$router.push({ path: '/SchoolManager/DepthAnalysis/SchoolDepthAna' })
      // if (this.IsVip === '1') {
      //   this.$router.push({ path: '/SchoolManager/DepthAnalysis/SchoolDepthAna' })
      // } else {
      //   this.$message.success('您的学校暂未开放此功能')
      // }
    },
        toWeekly () {
      // console.log('vip')
      this.$router.push({ path: '/SchoolManager/LearningEvaluation/Weekly' })

    },
    reMicEcharts () {
      this.$uwonhttp.post('/Analysis/VideoAnalysis/AnalysisVideoGrade', { UserId: this.userId }).then(res => {
        var Data = res.data.Data
        console.log(Data)
        this.microCount = Data.VideoCount
        const gradeId = Data.List.map(item => {
          return item.Grade
        })
        const gradeName = Data.List.map(item => {
          return item.GradeName
        })
        const gradeMicCount = Data.List.map(item => {
          return item.Count
        })
        var allMessage = []
        for (var i = 0; i < Data.List.length; i++) {
          allMessage[i] = { value: gradeMicCount[i], name: gradeName[i] }
        }
        console.log(gradeId)
        const micVideo = echarts.init(this.$refs.micVideo)
        micVideo.setOption({
          series: [
            {
              data: allMessage
            }
          ]
        })
        // 点击显示该年级下的所有微课
        this.getGradeChapterAndMicro(gradeId[0])
        var that = this
        micVideo.on('click', function (params) {
          var index = gradeId[params.dataIndex]
          that.getGradeChapterAndMicro(index)
        })
      })
    },
    // 获取该年级的章节和视频数
    getGradeChapterAndMicro (grade) {
      this.microLoad = true
      console.log(grade)
      this.$uwonhttp.post('/Analysis/VideoAnalysis/AnalysisVideoChapter', {
        UserId: this.userId,
        GradeId: grade
      }).then(res => {
        var Data = res.data.Data
        this.microLoad = false
        console.log(Data)

        // 获取年级章节：
        this.allChapter = Data
      })
    },
    // 获取该章节的微课视频
    getChatpterMic (index, ind) {
      var ChatpterMic = this.allChapter[index].Sections[ind].Videos
      var chapterName = this.allChapter[index].Sections[ind].SectionName
      this.knowMicro = ChatpterMic
      this.checkChapterName = chapterName
      console.log(ChatpterMic)
      console.log(chapterName)
      // debugger
    },
    // 跳转微课详情
    toMicoDetail (vid) {
      this.VisibleTask = false
      this.$router.push({ path: '/SchoolManager/teachCenter/videoDetail/videoCond', query: { videoId: vid } })
    },
    // 打开教师分析
    handleChapter () {
      this.VisibleTeacher = true
      this.$nextTick(() => {
        this.openTeacher()
      })
      // 教师分析获取知识点年级
      this.$uwonhttp.post('/Manager/KnowledgeAnalysis/AnalysisKnowledgeGrade', { UserId: this.userId }).then(res => {
        var Data = res.data.Data
        this.teacherGrade = Data
        console.log('知识点年级', this.teacherGrade)
      })
      this.getTeacherGrade()
      this.inithapter()
    },
    // 关闭微课视频分析
    onClose () {
      this.VisibleTask = false
    },
    // 关闭知识点分析
    onCloseChapterReport () {
      this.chapterReport = false
    },
    // 关闭教师分析
    CloseTeacher () {
      this.VisibleTeacher = false
    },
    // 选择知识点分析中的年级下拉框
    handleChangeKnoweldge (value) {
      this.gradeNum = value
      // 获取知识点分析年级下拉框
      this.getKnowGrade()
      // 题型模板
      this.studyAnalys()
      // // 学习水平
      this.studyLevelList()
    },
    // 知识点分析获取知识点年级
    getKnowGrade () {
      this.$uwonhttp.post('/Manager/KnowledgeAnalysis/AnalysisKnowledgeGrade', { UserId: this.userId }).then(res => {
        var Data = res.data.Data
        this.knowGrade = Data
        // 获取知识点分析单元下拉框
        this.getChapterId()
      })
    },
    // 获取教师分析章节
    inithapter () {
      this.$uwonhttp.post('/Chapter/TeacherChapter/GetTwoLevelByTerm', { gradeId: '1_1' }).then(res => {
        var Data = res.data.Data
        this.options = Data
        console.log(Data)
      })
    },

    // 选择教师分析中的章节下拉框
    getTeacherChapter (val) {
      this.teacherChapter = val[1]
      this.reTeacherEcharts()
    },
    // 选择教师分析中的年级下拉框
    getTeacherGrade (value) {
      this.teacGrade = value
      this.reTeacherEcharts()
    },
    // 选择章节或年级后刷新echarts
    reTeacherEcharts () {
      console.log(this.teacGrade, this.teacherChapter)
      this.$uwonhttp.post('/Manager/ManagerAnalysis/AnalysisTeacherWeb', {
        userId: this.userId,
        gradeIds: this.teacGrade,
        chapterId: this.teacherChapter
      }).then(res => {
        var Data = res.data.Data
        console.log(Data)

        const AreaAccuracy = []
        const SchoolAccuracy = []
        //   SchoolAccuracy[0]=Data.SchoolAccuracy
        for (var i = 0; i < Data.TeacherRateInfos.length; i++) {
          AreaAccuracy[i] = Data.AreaAccuracy
          SchoolAccuracy[i] = Data.SchoolAccuracy
        }
        const teachername = Data.TeacherRateInfos.map(item => {
          return item.TeacherName.replace(' ', '')
        })
        console.log(teachername, AreaAccuracy, SchoolAccuracy)
        const teacherAccuracy = Data.TeacherRateInfos.map(item => {
          return item.TeacherAccuracy
        })
        const teacherFinish = Data.TeacherRateInfos.map(item => {
          return item.TeacherFinish
        })
        console.log(teacherAccuracy, teacherFinish)
        const teacherEchart = echarts.init(this.$refs.teacherEchart)
        teacherEchart.setOption({
          color: ['#c9eafd', '#7ec5a6'],
          xAxis: [
            {
              data: teachername
            }
          ],
          series: [
            {
              data: teacherAccuracy
            },
            {
              data: teacherFinish
            },
            {
              data: SchoolAccuracy
            },
            {
              data: AreaAccuracy
            }
          ]
        })
        var that = this
        teacherEchart.on('click', function (params) {
          var index = params.dataIndex
          // var titleTeacherName=params
          that.teacherTitle = params.name + '老师的班级'
          var allClass
          if (Data.TeacherRateInfos) {
            allClass = Data.TeacherRateInfos[index].ClassRateInfos
          }
          var classList = allClass.map(item => {
            return item.ClassName
          })
          var ClassAccuracy = allClass.map(item => {
            return item.ClassAccuracy
          })
          var ClassFinish = allClass.map(item => {
            return item.ClassFinish
          })
          const teacherDetail = echarts.init(that.$refs.teacherDetail)
          teacherDetail.setOption({
            color: ['#c9eafd', '#7ec5a6'],
            tooltip: {
            },
            xAxis: [
              {
                data: classList
              }
            ],
            series: [
              {
                data: ClassAccuracy,
                barWidth: '45'
              },
              {
                data: ClassFinish,
                barWidth: '45'
              }
            ]
          })
        })
      })
    },

    // 获取知识点分析单元下拉框
    getChapterId () {
      this.$uwonhttp.post('/Manager/KnowledgeAnalysis/AnalysisKnowledgePaper', { UserId: this.userId, Grade: this.gradeNum }).then(res => {
        var Data = res.data.Data
        this.gradeChapter = Data
        console.log(this.gradeNum)
        console.log(Data)
        const chapId = Data.map(item => {
          return item.ChapterId
        })
        this.checkChapterId = chapId.join(',')
        console.log(this.checkChapterId)

        this.checkChapterIds = ''
        // 发起更新echarts接口
        this.getUpdateEcharts()
      })
    },
    getUpdateEcharts () {
      console.log(this.gradeNum)
      console.log(this.checkChapterId)
      console.log(this.checkChapterIds)
      this.$uwonhttp.post('/Manager/KnowledgeAnalysis/KnowledgeAnalysis', {
        UserId: this.userId,
        GradeId: this.gradeNum,
        FirstChapterId: this.checkChapterId,
        SecondChapterId: this.checkChapterIds,
        Type: 1
      }).then(res => {
        var Data = res.data.Data.Knowledge
        const kNowName = Data.map(item => {
          return item.Name
        })
        var namelist = []
        for (var i = 0; i < kNowName.length; i++) {
          namelist[i] = { text: kNowName[i], max: 100 }
        }
        const nowAccuracy = Data.map(item => {
          return item.CurrencyAvg
        })
        const pastAccuracy = Data.map(item => {
          return item.CurrencyWebAvg
        })
        console.log(kNowName, nowAccuracy, pastAccuracy)
        const knowledge = echarts.init(this.$refs.KnowledgePoints)
        knowledge.setOption({
          radar: [
            {
              indicator: namelist
            }
          ],
          series: [
            {
              type: 'radar',
              tooltip: {
                trigger: 'item'
              },
              data: [
                {
                  value: nowAccuracy,
                  name: '本届掌握情况'
                },
                {
                  value: pastAccuracy,
                  name: '上届掌握情况',
                  lineStyle: {
                    type: 'dashed'
                  }
                }
              ]
            }
          ]
        })
      })
      // 获取表格数据
      this.getTableMessage()
    },
    // 选择单元
    handleChapterChoice (val) {
      console.log(val)
      var arr = this.gradeChapter.filter(item => {
        return item.ChapterId === val
      })
      console.log(arr)
      var chapterList = arr[0].SecondChapters.map(item => {
        return item.ChapterId
      })
      this.checkChapterIds = chapterList.join(',')
      // console.log(this.checkChapterIds);
      this.checkChapterId = ''
      this.getUpdateEcharts()
      // this.getKnowGrade()
      // 题型模板
      this.studyAnalys()
      // // 学习水平
      this.studyLevelList()
    },
    // 获取表格数据
    getTableMessage () {
      const tableHead = []
      var tableData = []
      var ChapterName = []
      this.$uwonhttp.post('/Manager/ManagerAnalysis/GetAnalysisInfoWeb', {
        ChapterId: this.checkChapterId ? this.checkChapterId : this.checkChapterIds,
        UserId: this.userId,
        Grade: this.gradeNum
      }).then(res => {
        var Data = res.data.Data
        console.log(Data)
        const data = Data.map(item => {
          return item.List
        })
        // 获取章节名字列表
        const ChapterName = Data.map(item => {
          return item.ChapterName
        })

        // console.log(ChapterName);
        // 获取表头班级
        const classList = data[0].map(item => {
          return item.ClassName
        })
        // 获取表格数据
        console.log(classList)
        for (var j = 0; j < data.length; j++) {
          tableData.push(data[j].map(item => {
            return item.Corrent
          }))
        }
        console.log(tableData)
        // 设置表头班级
        tableHead.push({ title: '单元/小节', dataIndex: 'chapters', key: 0, width: 80, align: 'center' })
        for (var i = 0; i < classList.length; i++) {
          tableHead.push({ title: classList[i], dataIndex: 'classname' + i, key: i + 1, width: 80, align: 'center' })
        }
        this.columns1 = tableHead
        // console.log(tableHead);

        const rowList = []
        var classnames = []
        for (var b = 0; b < ChapterName.length; b++) {
          rowList[b] = {}
          rowList[b].key = b
          rowList[b].chapters = ChapterName[b]
          for (var c = 0; c < tableData[0].length; c++) {
            rowList[b]['classname' + c] = tableData[b][c] + '%'
          }
        }
        this.knowledgeData = rowList
      })
    },
    change () {
      this.color = '1'
      const studyLevel = echarts.init(this.$refs.studyLevel)
      studyLevel.dispose()
      const questionType = echarts.init(this.$refs.questionType)
      questionType.dispose()
    },
    // 学习水平
    change1 () {
      this.color = '2'
      const studyLevel = echarts.init(this.$refs.studyLevel)
      studyLevel.dispose()
      const questionType = echarts.init(this.$refs.questionType)
      questionType.dispose()
      this.$nextTick(() => {
        const studyLevel = echarts.init(this.$refs.studyLevel)
        studyLevel.setOption({
          title: {
            text: ''
          },
          tooltip: {
            trigger: 'axis'
          },
          color: ['#ff9e94', '#9fbaaa'],
          radar: [
            {
              radius: 150
            }
          ],
          series: [
            {
              type: 'radar',
              tooltip: {
                trigger: 'item'
              },
              areaStyle: {},
              data: [
                {
                  value: [100],
                  name: '本届掌握状况'
                },
                {
                  value: [100],
                  name: '上届掌握状况'
                }
              ]
            }
          ]
        })
        window.addEventListener('resize', function () {
          studyLevel.resize()
        })
      })
      this.studyLevelList()
    },
    // 获取学习水平
    studyLevelList () {
      this.$uwonhttp.post('/Manager/KnowledgeAnalysis/KnowledgeAnalysis', {
        UserId: this.userId,
        GradeId: this.gradeNum,
        FirstChapterId: this.checkChapterId,
        SecondChapterId: this.checkChapterIds,
        Type: 2
      }).then(res => {
        var Data = res.data.Data.Level
        console.log(Data)
        const name = Data.map(item => {
          return item.Name
        })
        console.log(name)
        var nameMax = []
        for (var i = 0; i < name.length; i++) {
          nameMax[i] = { text: name[i], max: 100 }
        }
        const CurrencyAvg = Data.map(item => {
          return item.CurrencyAvg
        })
        const CurrencyWebAvg = Data.map(item => {
          return item.CurrencyWebAvg
        })
        console.log(nameMax)
        console.log(CurrencyAvg)
        console.log(CurrencyWebAvg)
        const studyLevel = echarts.init(this.$refs.studyLevel)
        studyLevel.setOption({
          radar: [
            {
              indicator: nameMax
            }
          ],
          legend: {
            bottom: 0,
            show: true
          },
          series: [{
            type: 'radar',
            tooltip: {
              trigger: 'item'
            },
            areaStyle: {},
            data: [{
              value: CurrencyAvg,
              name: '本届掌握状况'
            },
            {
              value: CurrencyWebAvg,
              name: '上届掌握状况'
            }]
          }]
        })
      })
    },
    // 题型板块
    change2 () {
      this.color = '3'
      const studyLevel = echarts.init(this.$refs.studyLevel)
      studyLevel.dispose()
      const questionType = echarts.init(this.$refs.questionType)
      questionType.dispose()
      this.$nextTick(() => {
        const questionType = echarts.init(this.$refs.questionType)
        questionType.setOption({
          title: {
            text: ''
          },
          tooltip: {
            trigger: 'axis'
          },
          color: ['#ff9e94', '#9fbaaa'],
          radar: [{
            radius: 150

          }],

          series: [
            {
              type: 'radar',
              tooltip: {
                trigger: 'item'
              },
              areaStyle: {},
              data: [
                {
                  value: [0],
                  name: '本届掌握情况'
                },
                {
                  value: [0],
                  name: '上届掌握情况'
                }
              ]
            }
          ]
        })
        window.addEventListener('resize', function () {
          questionType.resize()
        })
      })
      this.studyAnalys()
    },
    // 获取题型模板接口
    studyAnalys () {
      this.$uwonhttp.post('/Manager/KnowledgeAnalysis/KnowledgeAnalysis', {
        UserId: this.userId,
        GradeId: this.gradeNum,
        FirstChapterId: this.checkChapterId,
        SecondChapterId: this.checkChapterIds,
        Type: 3
      }).then(res => {
        var Data = res.data.Data.Plate
        console.log(Data)
        const name = Data.map(item => {
          return item.Name
        })
        console.log(name)
        var nameMax = []
        for (var i = 0; i < name.length; i++) {
          nameMax[i] = { text: name[i], max: 100 }
        }
        const CurrencyAvg = Data.map(item => {
          return item.CurrencyAvg
        })
        const CurrencyWebAvg = Data.map(item => {
          return item.CurrencyWebAvg
        })
        console.log(nameMax)
        console.log(CurrencyAvg)
        console.log(CurrencyWebAvg)
        const questionType = echarts.init(this.$refs.questionType)
        questionType.setOption({
          radar: [
            {
              indicator: nameMax
            }
          ],
          legend: {
            bottom: 0,
            show: true
          },
          series: [
            {
              data: [
                {
                  value: CurrencyAvg,
                  name: '本届掌握情况'
                },
                {
                  value: CurrencyWebAvg,
                  name: '上届掌握情况'
                }
              ]
            }
          ]
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.stu-ana-col {
    font-size: 18px;
    color: #68BB97;
}
// 荣誉勋章
.honor {
    width: 100%;
}
.cur {
    text-align: center;
    cursor: pointer;
    font-size: 18px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #4D5753;
    display: inline-block;
    height: 100%;
    width: 21%;
    margin-left:3.8%;
    background-color: white;
    vertical-align: middle;
    border-radius: 5px;
    img{
        margin-top: 15%;
    }
    .drawName{
        margin: 10px 0px;
        display: inline-block;
    }
}
.color {
    color: #68BB97;
}
.example {
    text-align: center;
}
// 校本练习
.school-partivce {
    text-align: center;
    margin-top: 75px;
}
// 表格
/deep/.ant-table-thead {
    /deep/.ant-table-row-cell-break-word {
    color: #fff;
    background-color: #68BB97;
    }

}
/deep/.ant-table-tbody tr:nth-child(odd) {
    background-color: #F4F4F4;
}
/deep/.ant-select-selection {
    border: none;
    border-radius: 200px;
}
.btn-col {
    margin-right: 60px;
    color: #C3C3C3;
    background-color: #F7F8F8;
    border: 1px solid #C6C7C7;
}
.stu-analusis {
    display: flex;
}
.span {
        margin-right: 25px;
}
// 智能分析
.int-analysis {
    padding: 10px 20px 10px 20px;
    border-radius: 5px;
    background-color: #fff;
    white-space: nowrap;
}

// 练习信息
    .practice-infor {
    margin-top: 31px;
    .f-z {
        font-size: 18px;
    }
    }
// 学情分析
.Accuracy {
    width: 100%;
    margin-top: 30px;
    padding-top: 20px;
    padding: 20px;
    background-color: #fff;
    border-radius: 5px;
    .p{
        position: relative;
        height: 50px;
    }
}
.learnTitle{
        position: absolute;
        left: 40%;
        margin: 0px auto;
        text-align: center;
        font-size: 22px;
        color: #4D5753;
    }
.situationAnalysis {
    width: 100%;
    height: 437px;
    margin-top: 30px;
}
// 章节报告
.chapter-report {
    .title {
    position: relative;
    height: 50px;
    text-align: center;
    border-bottom: 1px solid #ccc;
    .gradeSelect {
        position: absolute;
        right: 15%;
        /deep/.ant-select-selection--single {
        border: none;
        background-color: #f7f8f8;
        }
    }
    .chapter-select {
        /deep/.ant-select-selection--single {
        border: none;
        background-color: #f7f8f8;
        }
    }
    }
    // 图表操作
    .chart-operation {
    text-align: center;
    margin-top: 30px;
    span {
        display: inline-block;
        width: 111px;
        height: 48px;
        line-height: 48px;
        text-align: center;
        border-radius: 25px;
        cursor: pointer;
    }
    .bgc-color {
        color: #fff;
        background-color: #68BB97;
        border: none;
    }
    }
    // 知识点
    .knowledge-points {
        width: 100%;
        height: 628px;
        position: relative;
    }
    // 学习水平
    .study-level {
    width: 100%;
    height: 628px;
    }

    // 题型模板
    .question-type {
    width: 100%;
    height: 628px;
    }
}
.checkgrade{
    position: absolute;
    right: 5%;
}
.tips{
    display: inline-block;
    width: 100%;
    height: 30px;
    border-bottom:0px;
}
.tip{
    text-decoration: none;
    width: 150px;
    height: 30px;
    display: inline-block;
    border-bottom:0px;
}
#teacherEchart{
    width: 100%;
    height: 400px;
    background-color: white;
}
#teacherDetail{
    width: 100%;
    height: 400px;
    background-color: white;
}
.accuracyLine{
    color:#ffc0b9;
}
.finishLine{
    color:#a1bcad;
}
.micVideo{
    width: 100%;
    height: 400px;
    position: relative;
}
.micorCount{
    text-align:center;
}
#selectChapters{
    vertical-align: top;
    padding: 10px 10px;
    display: inline-block;
    width: 33%;
    height: 350px;
    overflow-y: scroll;
}

.backImg{
    width: 100%;
    height: 75%;
    position: relative;
}
.teacherGrade{
    width: 200px;
    margin: 0px 20px;
    display: inline-block;
    border-radius: 200px;
}
#bottom{
    width: 100%;
    border-radius: 10px;
    height: 1335px;
    background-color: white;
}
.left{
    overflow-y: scroll;
    width: 35%;
    height: 100%;
    display: inline-block;
    vertical-align: middle;
    // border: 1px solid;
    border-radius: 5px;
    position: relative;
}

.imgs{
    padding:0px 10px;
    height: 100%;
    width: 100%;
    // border: 1px solid;
    line-height: 45px;
    cursor: pointer;
    position: relative;
}
.imgs:hover{
    background-color: #F2F2F2;
    border-radius: 50px;
}
.Content{
    float: right;
}
#micRight{
    padding:10px;
    overflow-y: scroll;
    width: 65%;
    height: 100%;
    display: inline-block;
    vertical-align: middle;
}
#allMicro{
    width: 100%;
    background-color: #f2f2f2;
    border-radius: 5px;
    padding: 5px 10px;
}
.showMicros{
    vertical-align: middle;
    display: inline-block;
    width: 46%;
    margin: 10px 2%;
    height: 290px;
    background-color: white;
}
.video{
    width: 100%;
    height: 70%;
}
.videoDetail{
    width: 100%;
    height: 15%;
    padding: 0px 10px;
}
.videoTeacher{
    width: 100%;
    height: 15%;
    padding: 0px 10px;
    position: relative;
}

.chapter,.time,.watchCount,.rate,.teacherName{
    display: inline-block;
    vertical-align: middle;
}
.time,.watchCount,.rate,.teacherName{
    font-size: 10px;
    color:#B5B5B5;
}
.time{
    width: 25%;
    vertical-align: middle;
}
.chapter{
    width: 50%;
    color: #373737;
    overflow: hidden;
    white-space: nowrap;
    text-overflow:ellipsis;
    vertical-align: middle;
}
.watchCount{
     display: inline-block;
     width: 25%;
     white-space:nowrap;
     vertical-align: middle;
}
.rate{
    width: 40%;
    color: #ebeb05;
    display: inline-block;
}
.teacherName{
    position: absolute;
    display: inline-block;
    right: 10px;
}
.microImg{
    position: relative;
    position: relative;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    cursor: pointer;
    color: #F2F2F2;
}
.fg{
    width: 200px;
}
#teacherClass{
    text-align:center;
    background-color:white;
    font-size: 16px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #373737;
    height: 50px;
    line-height: 50px;
}
.mainMessage{
    font-size: 16px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #4D5753;
}
.bottomMainMessage{
    right: 0px;
    position: absolute;
    width: 100px;
    height: 50px;
    line-height: 50px;
    font-size: 16px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #4D5753;
}
.teacherTop{
    text-align: right;
    padding-right: 100px;
}
/deep/.ant-cascader-input,.ant-cascader-picker{
    border-radius: 200px;
    border: none;
    width: 300px;
}
/deep/.ant-cascader-picker-label{
    text-align: center;
}

</style>
