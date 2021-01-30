<template>
  <div>
    <p>
      <span class="f-t">薄弱点分析</span>
      <span >
        <!-- 班级 -->
        <a-select v-if="classDefalutId" style="width: 120px" @change="handleChangeClass" :default-value="classDefalutId">
          <a-select-option :value="item.GradeId" v-for="item in classInfor" :key="item.ClassId">
            {{ item.GradeName }}
          </a-select-option>
        </a-select>
      </span>
    </p>
    <p>
      <span @click="handleChapterSwitch" class="chapter-name cur wid-text" :class="{ 'chapter-color': chapterName === 0 }">知识点</span>
      <span @click="handleChapterSwitch" class="chapter-name cur wid-text" :class="{ 'chapter-color': chapterName === 1 }">题型板块</span>
    </p>
    <p v-if="chapterName === 0">
      <!-- <span class="m-r small-chapter">一 10以内的数</span>
      <span class="m-r">二 10以内数的加减法</span> -->
      <span :class="{ 'chapter-color': firstChapter === ite.ChapterId}" @click="handleChapter(ite.ChapterId, ite.ChapterName)" v-for="ite in chapterList" :key="ite.ChapterId" class="chapter-name cur">{{ ite.ChapterName }}</span>
    </p>
    <!-- 题型版块 -->
    <p v-show="chapterName === 1">
      <span @click="handleQuesType(item.id,item.name)" class="m-r small-chapter chapter-name cur" :class="{ 'chapter-color': selectType === item.id}" v-for="(item,index) in quesTypeList" :key="index">{{ item.name }}</span>
    </p>

    <div class="pop-info">
      <div v-if="showWeak" class="default-unit">
        <img src="@/assets/lack/暂无关注好友.png" alt="">
        <p>暂无相关数据</p>
      </div>
      <!-- 图表 -->
      <div v-show="!showWeak">
        <div class="weak-pointer" ref="weakPointer"></div>
      </div>
      <div class="notes">
        <p>页面注释：</p>
        <p>1.本页面为班级某题型版块学业水平与总体学业水平的比较，旨在帮助教师发现学生的薄弱点和优势点，助力精准教学。</p>
        <p>2.图内各点代表各个班级，点击显示各版块与总体的比较。</p>
      </div>
    </div>
  </div>
</template>

<script>
import echarts from 'echarts'
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    this.shoolId = localStorage.getItem('SchoolId')
    this.getClassInfo()
    // 题型板块
    this.getQuestionType()
  },
  mounted () {
    this.init()
  },
  data () {
    return {
      shoolId: '',
      // 缺省
      showWeak: false,
      userId: '',
      gradeId: '',
      classId: '',
      // 班级默认id
      classDefalutId: '',
      chapterName: 0,
      classInfor: [],
      // 知识点
      chapterList: [],
      // 第一章节id
      firstChapter: '',
      // 题型板块
      quesType: [],
      // 选中题型板块
      selectType: 1,
      arr: [],
      // 图表显示的章节
      sleChapterName: '',
      // 题型板块列表
      quesTypeList: [],
      // 切换到知识点的默认章节
      switchFirstChapter: '',
      // 切换到知识点的默认章节名称
      switchSleChapterName: ''
    }
  },
  methods: {
    init () {
      // 散点图
      const weakPointer = echarts.init(this.$refs.weakPointer)
      weakPointer.setOption({
        title: {
          text: ''
        },
        grid: {
          left: '3%',
          right: '7%',
          bottom: '3%',
          containLabel: true
        },
        tooltip: {
          // trigger: 'axis',
          showDelay: 0,
          formatter: function (params) {
            // console.log(params)
            return '学生：' + params.seriesName + '<br>' +
                  '我的正确率：' + params.value[0] + '<br>' +
                  '总体分数：' + params.value[1]
          },
          axisPointer: {
            show: true,
            type: 'cross',
            lineStyle: {
              type: 'solid',
              width: 1
            }
          }
        },

        legend: {
          data: [],
          right: 'center'
        },
        xAxis: [
          {
            type: 'value',
            max: 100,
            scale: false,
            name: '(总体学业水平:%)',
            nameLocation: 'end',
            nameGap: 20,
            axisLabel: {
              formatter: '{value}'
            },
            splitLine: {
              show: true
            }
          }
        ],
        yAxis: [
          {
            type: 'value',
            max: 100,
            min: 0,
            scale: true,
            name: '（知识点：%）',
            nameLocation: 'end',
            axisLabel: {
              formatter: '{value}'
            },
            splitLine: {
              show: true,
              lineStyle: {
                type: 'dashed'
              }
            }
          }
        ],
        color: ['#5CC0FF', '#FACC14', '#448870', '#79BEBD', '#F57579'],
        series: [
          {
            name: '李小龙',
            type: 'scatter',
            data: [
              [61.2, 51.6, '一年级'], [67.5, 59.0, '一年级'], [59.5, 49.2, '一年级'], [57.0, 63.0, '一年级'], [55.8, 53.6, '一年级'],
              [0, 0], [0, 0], [0, 69.8], [0, 66.8], [0, 75.2]
            ]
          },
          {
            name: '王二',
            type: 'scatter',
            data: [[14.0, 65.6], [75.3, 71.8], [93.5, 80.7], [86.5, 72.6], [87.2, 78.8],
              [81.5, 74.8], [84.0, 86.4], [184.5, 78.4], [75.0, 62.0], [84.0, 81.6]
            ]
          },
          {
            name: '三年级',
            type: 'scatter'
          },
          {
            name: '四年级',
            type: 'scatter'
          },
          {
            name: '五年级',
            type: 'scatter'
          }
        ]
      })
    },
    // 获取老师所带班级
    getClassInfo () {
      this.$uwonhttp.post('/Manager/ManagerAnalysis/GetGrade', { UserId: this.userId }).then(res => {
        console.log('老师带领班级---', res)
        this.classInfor = res.data.Data.filter(value => {
          return value.GradeId !== 0
        })
        this.classDefalutId = res.data.Data[1].GradeId
        this.classId = res.data.Data[1].GradeId
        this.gradeId = res.data.Data[1].GradeId
        this.getChapter()
      })
    },
    // 获取知识点
    getChapter () {
      this.$uwonhttp.post('/Chapter/Chapter/GetFirstChapterByGrade', { grade: this.gradeId, IsWeek: 1 }).then(res => {
        console.log('年级对应的章节---', res)
        this.chapterList = res.data.Data
        this.firstChapter = res.data.Data[0].ChapterId
        this.switchFirstChapter = res.data.Data[0].ChapterId
        this.sleChapterName = res.data.Data[0].ChapterName
        this.switchSleChapterName = res.data.Data[0].ChapterName
        // this.getQuestionType()
        this.getWeakStu()
      })
    },
    // 获取题型板块
    getQuestionType () {
      this.$uwonhttp.get('/report/TeacherDeepAnalysis/GetPlateTypeList').then(res => {
        console.log('题型板块---', res)
        this.quesTypeList = res.data.Data
      })
    },
    // 获取薄弱学生统计
    getWeakStu () {
      // debugger
      this.$uwonhttp.post('/report/SchoolDeepAnalysis/FirstSchoolWeakPointsAnalysis', { ShoolId: this.shoolId, FType: this.chapterName, Fid: this.firstChapter, GradeId: this.gradeId }).then(res => {
        console.log('薄弱统计---', res)
        if (res.data.Data.length === 0) {
          this.showWeak = true
        } else {
          this.showWeak = false
          const arr1 = []
          const arr2 = []
          const option = {}
          option.series = []
          res.data.Data.forEach(value => {
            this.arr = []
            for (const key in value) {
              this.arr.push(value[key])
            }
            console.log('data数据', this.arr)
            console.log('输入的数据', this.arr.slice(4))
            console.log('输入的数据翻转数据', this.arr.slice(4).reverse())
            const reverArr = this.arr.slice(4).reverse()
            arr1.push(this.arr.slice(3, 4))
            // console.log('arr1===', arr1)
            arr2.push(this.arr.slice(4))
            option.series.push(
              {
                name: this.arr.slice(3, 4).join(' '),
                type: 'scatter',
                data: [reverArr]
              }
            )
          })
          console.log('外面ARR---', option)
          // console.log('外面ARR---', arr2)
          const that = this
          option.tooltip = {
          // trigger: 'axis',
            showDelay: 0,
            formatter: function (params) {
            // console.log(params)
              return params.seriesName + '<br>' +
                  that.sleChapterName + ':' + params.value[1] + '<br>' +
                  '总体分数：' + params.value[0]
            },
            axisPointer: {
              show: true,
              type: 'cross',
              lineStyle: {
                type: 'solid',
                width: 1
              }
            }
          }
          const weakPointer = echarts.init(this.$refs.weakPointer)
          weakPointer.setOption(option)
        }
      })
    },
    // 选择题型板块
    handleQuesType (id, name) {
      this.selectType = id
      this.firstChapter = id
      this.sleChapterName = name
      // switch (this.selectType) {
      //   case 1: this.sleChapterName = '数与运算'
      //     break
      //   case 2: this.sleChapterName = '图形与几何'
      //     break
      //   case 3: this.sleChapterName = '解决问题'
      //     break
      //   case 4: this.sleChapterName = '统计与概率'
      //     break
      //   case 5: this.sleChapterName = '概念'
      //     break
      //   case 7: this.sleChapterName = '方程与代数'
      //     break
      //   case 8: this.sleChapterName = '识记'
      //     break
      // }
      this.getWeakStu()
    },
    // 选择章节
    handleChapter (value, chapterName) {
      this.firstChapter = value
      this.sleChapterName = chapterName
      // this.getQuestionType()
      console.log(value)
      this.getWeakStu()
    },
    // 选择大章节
    handleChapterSwitch () {
      if (this.chapterName === 0) {
        this.chapterName = 1
        this.firstChapter = 1
        this.sleChapterName = '数与运算'
      } else {
        this.chapterName = 0
        this.firstChapter = this.switchFirstChapter
        this.sleChapterName = this.switchSleChapterName
      }
      this.getWeakStu()
    },
    handleChangeClass (value) {
      this.classDefalutId = value
      // console.log(value.split('|'))
      // const list = value.split('|')
      // this.classId = list[1]
      this.gradeId = value
      this.getChapter()
    }
  }
}
</script>

<style lang="less" scoped>
  .pop-info {
    padding: 0 20px;
    height: 700px;
    background-color: #fff;
    border-radius: 8px;
  }
    .f-t {
    font-size: 22px;
    margin-right: 30px;
    margin-bottom: 39px;
  }
  .wid-text {
    width: 98px;
    text-align: center;
  }
  .m-r {
    margin-right: 25px;
  }
   // 缺省
  .default-unit {
    margin-top: 0;
    padding-top: 60px;
  }
  .weak-pointer {
    width: 100%;
    height: 500px;
  }
  .notes {
    margin-top: 30px;
    font-size: 16px;
    padding: 14px 8px;
    background-color: #F5F5F5;
    border-radius: 5px;
    p:nth-child(1) {
      font-size: 18px;
    }
  }
   /deep/.ant-select-selection--single {
          border: none;
          background-color: rgba(240, 242, 245);
  }
  .small-chapter {
    color:#685C47;
    font-size: 14px;
  }
  .chapter-name {
    // width: 98px;
    padding: 5px 11px;
    display: inline-block;
    margin-right: 20px;
    margin-bottom: 15px;
    font-size: 14px;
    text-align: center;
    background-color: #ddd;
    border-radius: 20px;
  }
  .chapter-color {
    color: #fff;
    background-color: #68BB97;
  }
</style>
