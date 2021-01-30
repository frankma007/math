<template>
  <div>
    <p class="score-nav">
      <span>学习成绩分布</span>
      <span>{{ className }}</span>
    </p>
    <p class="m-b">
      <span :class="{ 'chapter-color': firstChapter === ite.ChapterId}" @click="handleChapter(ite.ChapterId)" v-for="ite in chapterList" :key="ite.ChapterId" class="chapter-name" >{{ ite.ChapterName }}</span>
    </p>
    <div class="pop-info">
      <div v-if="showScore" class="default-unit">
        <img src="@/assets/lack/暂无关注好友.png" alt="">
        <p>暂无相关数据</p>
      </div>
      <!-- 图表 -->
      <div v-if="!showScore">
        <div class="score-distribution" ref="scoreDistribution"></div>
      </div>
      <div class="notes">
        <p>页面注释：</p>
        <p>本页面旨在对学生成绩的具体分布有直观了解，图形之外的数据为异常值，上下边缘分别代表最大值和最小值，箱体上下分别代表上四分位点和下四分位点，箱体的中心实线代表中位数。</p>
      </div>
    </div>
  </div>
</template>

<script>
import echarts from 'echarts'
import dataTool from 'echarts/extension/dataTool'
export default {
  created () {
    this.className = this.$route.query.className
    this.gradeId = this.$route.query.gradeId
    this.classId = this.$route.query.classId
    this.getGradeChapter()
  },
  mounted () {
    this.init()
  },
  watch: {
    $route () {
      this.className = this.$route.query.className
      this.gradeId = this.$route.query.gradeId
      this.classId = this.$route.query.classId
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/My_Task/InspectionWorkData?' + ChangId) {
        this.getGradeChapter()
      }
    }
  },
  data () {
    return {
      // 缺省
      showScore: false,
      className: '',
      gradeId: '',
      classId: '',
      chapterList: [],
      firstChapter: ''
    }
  },
  methods: {
    init () {
      var data = echarts.dataTool.prepareBoxplotData([
        [56, 35, 65, 89, 75, 15, 35, 45, 78, 65, 42, 78, 90]
      ])
      const scoreAna = echarts.init(this.$refs.scoreDistribution)
      scoreAna.setOption({
        title: [
          {

          },
          {
            textStyle: {
              fontSize: 14
            },
            left: '10%',
            top: '90%'
          }
        ],
        tooltip: {
          trigger: 'item',
          axisPointer: {
            type: 'shadow'
          }
        },
        grid: {
          left: '10%',
          right: '10%',
          bottom: '15%'
        },
        xAxis: {
          type: 'category',
          data: [],
          boundaryGap: true,
          nameGap: 30,
          splitArea: {
            show: false
          },
          axisLabel: {
            formatter: 'expr {value}'
          },
          splitLine: {
            show: false
          }
        },
        yAxis: {
          type: 'value',
          name: '（单位：%）',
          min: 0,
          max: 100,
          splitArea: {
            show: true
          }
        },
        series: [
          {
            name: 'boxplot',
            type: 'boxplot',
            data: data.boxData,
            itemStyle: { color: '#9AB5E2' },
            tooltip: {
              formatter: function (param) {
                return [
                  '最大值: ' + param.data[5],
                  '上四分位数: ' + param.data[4],
                  '中位数: ' + param.data[3],
                  '下四分位数: ' + param.data[2],
                  '最小值: ' + param.data[1]
                ].join('<br/>')
              }
            }
          }
        ]
      })
    },
    // 获取年级对应的章节
    getGradeChapter () {
      this.$uwonhttp.post('/Chapter/Chapter/GetFirstChapterByGrade', { grade: this.gradeId, IsWeek: 1 }).then(res => {
        console.log('年级对应的章节---', res)
        this.chapterList = res.data.Data
        this.firstChapter = res.data.Data[0].ChapterId
        this.getResult()
      })
    },
    // 获取成绩分布
    getResult () {
      this.$uwonhttp.post('/report/SchoolDeepAnalysis/FirstAchievementAnalysis', { ClassId: this.classId, FirstChapterId: this.firstChapter }).then(res => {
        console.log('成绩分布---', res)
        if (res.data.Data.length === 0) {
          this.showScore = true
          return
        }
        const list = res.data.Data
        // const arr = [list.AverageNum, list.LowerQuartile, list.Maximum, list.MinNum, list.UpperQuartile, list.middleNum]
        const arr = [list.MinNum, list.LowerQuartile, list.middleNum, list.UpperQuartile, list.Maximum]
        const data = echarts.dataTool.prepareBoxplotData([
          arr
          // [12, 44, 56, 68, 100],
          // [53.8, 12.5, 41.2, 60, 71.1, 100]
        ], { boundIQR: 'none' })
        console.log(arr)
        console.log('丢进去的数据', data)
        const scoreAna = echarts.init(this.$refs.scoreDistribution)
        scoreAna.setOption({
          series: [
            {
              name: 'boxplot',
              type: 'boxplot',
              data: [arr],
              itemStyle: { color: '#9AB5E2' },
              tooltip: {
                formatter: function (param) {
                  // console.log(param)
                  return [
                    '最大值: ' + param.data[5],
                    '上四分位数: ' + param.data[4],
                    '中位数: ' + param.data[3],
                    '下四分位数: ' + param.data[2],
                    '最小值: ' + param.data[1]
                  ].join('<br/>')
                }
              }
            }
          ]
        })
      })
    },
    handleChapter (id) {
      this.firstChapter = id
      this.getResult()
    }
  }
}
</script>

<style lang="less" scoped>
  .m-b {
    margin-bottom: 15px;
  }
  .default-unit {
    margin: 0;
    padding-top: 60px;
  }
  .score-nav {
    span:nth-child(1) {
      font-size: 22px;
      margin-right: 30px;
    }
    span:nth-child(1) {
      font-size: 18px;
    }
  }
  .score-distribution {
    height: 450px;
  }
   .notes {
     font-size: 16px;
    padding: 14px 8px;
    background-color: #F5F5F5;
    border-radius: 5px;
    p:nth-child(1) {
      font-size: 16px;
    }
  }
  .chapter-name {
    padding: 5px 11px;
    display: inline-block;
    margin-right: 20px;
    margin-top: 15px;
    background-color: #ddd;
    border-radius: 20px;
    cursor: pointer;
  }
  .chapter-color {
    color: #fff;
    background-color: #68BB97;
  }
</style>
