<template>
  <div ref="lines"></div>
</template>

<script>
import echarts from 'echarts'
export default {
  data() {
    return {
      AreaAccuracy: [],
      ClassAccuracy: [],
      SchoolAccuracy: [],
      WeeklyTime: [],
      title: [ '全区正确率', '全校正确率'],
    }
  },
  props: {
    lineChartList: Array,
    PracticeId: String,
  },
  methods: {
    init() {
      //环形图
      const line = echarts.init(this.$refs.lines)

      var option = {
        grid: {
           left: 20,
          right: 60,
          bottom: 36,
          top:36,
          containLabel: true,
        },
        dataZoom: [
          {
            start: 0, // 默认为0
            end: 40, // 默认为100
              type: 'slider',
            show: true,
            xAxisIndex: [0],
            handleSize: 0, // 滑动条的 左右2个滑动条的大小
            height: 10, // 组件高度
            // left: '10%', // 左边的距离
            // right: '10%', // 右边的距离
            bottom: 8, // 右边的距离
            border: 'none',
            borderColor: '#fff',
            fillerColor: '#CCD0DB',
            borderRadius: 2,
            backgroundColor: '#ededed' , // 两边未选中的滑动条区域的颜色
            showDataShadow: false, // 是否显示数据阴影 默认auto
            showDetail: false, // 即拖拽时候是否显示详细数值信息 默认true
            realtime: true, // 是否实时更新
            filterMode: 'filter',
          },
        ],

        tooltip: {
          trigger: 'axis',
          label: {
             color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
          },
          formatter(params) {
            var str = params[0].axisValue + '<br>'
            params.forEach((vals) => {
              str = str + ` ${vals.marker} ${vals.seriesName}:${vals.value}%<br>`
            })
            return str
          },
        },
        legend: {
          data: this.title,
          top:6,
            textStyle: {
             color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
          },
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: this.WeeklyTime,
            offset: 10,
          axisLabel: {
            color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
          },
          axisLine: {
            lineStyle: {
              color: '#E2E2E2',
            },
          },
          axisTick: false,
        },
        yAxis: {
          type: 'value',
          axisLabel: {
            formatter: '{value} %',
             color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
          },
          axisLine: {
            lineStyle: {
              color: '#E2E2E2',
            },
          },
          axisTick: false,
        },
        series: [
          // {
          //   name: '班级正确率',
          //   type: 'line',
          //   smooth: true,
          //   data: this.ClassAccuracy,
          //   color: '#479AEB',
          // },
          {
            name: '全区正确率',
            type: 'line',
            // smooth: true,
            data: this.AreaAccuracy,
            color: '#F87175',
            lineStyle: {
              width: 3,
            },
          },
          {
            name: '全校正确率',
            type: 'line',
            // smooth: true,
            data: this.SchoolAccuracy,
            color: '#F8C84A',
            lineStyle: {
              width: 3,
            },
          },
        ],
      }
      if (this.PracticeId == '2') {
        option.series.pop()
      } 
      line.setOption(option)
      setTimeout(() => {
        window.onresize = () => {
          // debugger
          line.resize()
        }
      }, 200)
    },
    formatArr() {
      this.AreaAccuracy = []
      // this.ClassAccuracy = []
      this.SchoolAccuracy = []
      this.WeeklyTime = []

      // debugger
      this.lineChartList.forEach((val) => {
        this.AreaAccuracy.push(val.AreaAccuracy)
        // this.ClassAccuracy.push(val.ClassAccuracy)
        this.SchoolAccuracy.push(val.SchoolAccuracy)
        this.WeeklyTime.push(val.WeeklyTime)
      })
      // debugger
    },
    getTit() {
      if (this.PracticeId == '2') {
        this.title.pop()
      } 
    },
  },
  mounted() {
    this.getTit()
    this.formatArr()
    this.init()
  },
}
</script>


