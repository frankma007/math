<template>
  <div>
    <p>
      <span class="f-t">学习成绩分布</span>
      <span >
        <!-- 班级 -->
        <a-select v-if="classDefalutId && this.classInfor.length !== 0" style="width: 120px" @change="handleChangeClass" :default-value="classDefalutId">
          <a-select-option :value="item.GradeId" v-for="item in classInfor" :key="item.GradeId">
            {{ item.GradeName }}
          </a-select-option>
        </a-select>
      </span>
    </p>
    <p class="m-b">
      <span :class="{ 'chapter-color': firstChapter === ite.ChapterId}" @click="handleChapter(ite.ChapterId)" v-for="ite in chapterList" :key="ite.ChapterId" class="chapter-name" >{{ ite.ChapterName }}</span>
    </p>

    <div class="pop-info">
      <div v-if="showStudyRes" class="default-unit">
        <img src="@/assets/lack/暂无关注好友.png" alt="">
        <p>暂无相关数据</p>
      </div>
      <!-- 图表 -->
      <div v-show="!showStudyRes">
        <div class="study-result" ref="studyResult"></div>
      <!-- <div id="studyResult"></div> -->
      </div>
      <div class="notes">
        <p>页面注释：</p>
        <p>本页面旨在对各班学生成绩的具体分布有直观了解，图形之外的数据为异常值，上下边缘分别代表最大值和最小值，箱体上下分别代表上四分位点和下四分位点，箱体的中心实线代表中位数。</p>
      </div>
    </div>
  </div>
</template>

<script>
import echarts from 'echarts'
import dataTool from 'echarts/extension/dataTool'
// import { Chart } from '@antv/g2'
// import { View } from '@antv/data-set'
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    this.getClassInfo()
  },
  mounted () {
    this.init()
    // this.initAnt()
    // const e = document.createEvent('Event')
    // e.initEvent('resize', true, true)
    // window.dispatchEvent(e)
  },
  data () {
    return {
      userId: '',
      // 缺省
      showStudyRes: false,
      chapterName: 1,
      classInfor: [],
      // 默认年级
      classDefalutId: '',
      gradeId: '',
      // 章节
      chapterList: [],
      // 第一章节id
      firstChapter: '',
      allData: []
    }
  },
  methods: {
    init () {
      var data = echarts.dataTool.prepareBoxplotData([
        [56, 35, 65, 89, 75, 15, 35, 45, 78, 65, 42, 78, 90],
        [70, 86, 45, 56, 13, 54, 78, 64, 78, 42, 32, 65, 35]
      ])
      const Study = echarts.init(this.$refs.studyResult)
      Study.setOption({
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
          data: ['一（1）班', '二（1）班'],
          boundaryGap: true,
          nameGap: 30,
          splitArea: {
            show: false
          },
          axisLabel: {
            formatter: '{value}'
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
            tooltip: {
              formatter: function (param) {
                // console.log('盒须图', param)
                return [
                  // param.name + ': ',
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
    initAnt () {
      const DataSet = require('@antv/data-set')
      debugger
      // var ds = new DataSet()
      var _DataSet = DataSet
      var DataView = _DataSet.DataView

      var data = [{
        x: 'Oceania',
        low: 1,
        q1: 9,
        median: 16,
        q3: 22,
        high: 24
      }, {
        x: 'East Europe',
        low: 1,
        q1: 5,
        median: 8,
        q3: 12,
        high: 16
      }, {
        x: 'Australia',
        low: 1,
        q1: 8,
        median: 12,
        q3: 19,
        high: 26
      }, {
        x: 'South America',
        low: 2,
        q1: 8,
        median: 12,
        q3: 21,
        high: 28
      }, {
        x: 'North Africa',
        low: 1,
        q1: 8,
        median: 14,
        q3: 18,
        high: 24
      }, {
        x: 'North America',
        low: 3,
        q1: 10,
        median: 17,
        q3: 28,
        high: 30
      }, {
        x: 'West Europe',
        low: 1,
        q1: 7,
        median: 10,
        q3: 17,
        high: 22
      }, {
        x: 'West Africa',
        low: 1,
        q1: 6,
        median: 8,
        q3: 13,
        high: 16
      }]
      debugger
      var dv = new DataView().source(data)
      dv.transform({
        type: 'map',
        callback: function callback (obj) {
          obj.range = [obj.low, obj.q1, obj.median, obj.q3, obj.high]
          return obj
        }
      })

      const chart = new Chart({
        container: 'studyResult',
        forceFit: true,
        height: 450,
        padding: [50, 100, 50, 100]
      })
      chart.forceFit()
      chart.source(dv, {
        range: {
          max: 35
        }
      })
      chart.tooltip({
        showTitle: false,
        crosshairs: {
          type: 'rect'
        },
        itemTpl: '<li data-index={index} style="margin-bottom:4px;">' + '<span style="background-color:{color};" class="g2-tooltip-marker"></span>' + '{name}<br/>' + '<span style="padding-left: 16px">最大值：{high}</span><br/>' + '<span style="padding-left: 16px">上四分位数：{q3}</span><br/>' + '<span style="padding-left: 16px">中位数：{median}</span><br/>' + '<span style="padding-left: 16px">下四分位数：{q1}</span><br/>' + '<span style="padding-left: 16px">最小值：{low}</span><br/>' + '</li>'
      })
      chart.schema().position('x*range').shape('box').tooltip('x*low*q1*median*q3*high', function (x, low, q1, median, q3, high) {
        return {
          name: x,
          low: low,
          q1: q1,
          median: median,
          q3: q3,
          high: high
        }
      }).style({
        stroke: '#545454',
        fill: '#1890FF',
        fillOpacity: 0.3
      })
      // chart.source(data)
      chart.render()
    },
    // 获取学习成绩分布
    getStudyScatter () {
      this.$uwonhttp.post('/report/SchoolDeepAnalysis/FirstAchievementClassAnalysis', { GradeId: this.gradeId, FirstChapterId: this.firstChapter, TeacherId: this.userId }).then(res => {
        console.log('学习成绩分布---', res)
        if (res.data.Data.length === 0) {
          this.showStudyRes = true
          return
        } else {
          this.showStudyRes = false
        }
        const className = res.data.Data.map(value => {
          return value.ClassName
        })
        const arr1 = []
        res.data.Data.forEach(value => {
          const arr = []
          for (const i in value) {
            console.log('value哈哈哈', value)
            arr.push(value[i])
          }

          const Arr = arr.slice(2)
          console.log('循环完', Arr)
          Arr.pop()
          console.log('删除完的ARR', Arr)
          arr1.push(Arr)
          console.log('添加图表数据', arr1)
        })
        var data = echarts.dataTool.prepareBoxplotData(
          arr1
          // [
          //   [100, 96, 93, 89, 81, 90],
          //   [98, 93, 88, 83, 68, 84]
          // ]
        )
        const Study = echarts.init(this.$refs.studyResult)
        Study.setOption({
          xAxis: {
            type: 'category',
            data: className
          },
          series: [
            {
              name: 'boxplot',
              type: 'boxplot',
              data: data.boxData,
              itemStyle: { color: '#9AB5E2' }
            }
          ]
        })
      })
    },
    getClassInfo () {
      this.$uwonhttp.post('/Manager/ManagerAnalysis/GetGrade', {
        UserId: this.userId
      }).then(res => {
        console.log('老师带领的年级---', res)

        this.classInfor = res.data.Data.filter(value => {
          return value.GradeId !== 0
        })
        this.classDefalutId = res.data.Data[1].GradeId
        this.gradeId = res.data.Data[1].GradeId
        this.getChapter()
      })
    },
    getChapter () {
      this.$uwonhttp.post('/Chapter/Chapter/GetFirstChapterByGrade', { grade: this.gradeId, IsWeek: 1 }).then(res => {
        console.log('年级对应的章节---', res)
        this.chapterList = res.data.Data
        this.firstChapter = res.data.Data[0].ChapterId
        this.getStudyScatter()
      })
    },
    // 选择班级
    handleChangeClass (value) {
      this.gradeId = value
      this.getChapter()
      // this.getStudyScatter()
    },
    // 选择章节
    handleChapter (id) {
      this.firstChapter = id
      this.getStudyScatter()
    }
  }
}
</script>

<style lang="less" scoped>
  .m-b {
    margin-bottom: 20px;
  }
  .f-t {
    font-size: 22px;
    margin-right: 30px;
  }
   // 缺省
  .default-unit {
    margin-top: 0;
    padding-top: 60px;
  }
  .study-result {
    height: 450px;
  }
   .notes {
     font-size: 16px;
    padding: 14px 8px;
    background-color: #F5F5F5;
    border-radius: 5px;
    p:nth-child(1) {
      font-size: 18px;
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
   /deep/.ant-select-selection--single {
    border: none;
    background-color: rgba(240, 242, 245);
  }
  .chapter-color {
    color: #fff;
    background-color: #68BB97;
  }
</style>
