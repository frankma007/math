<template>
  <div ref="barLine"></div>
</template>

<script>
import echarts from 'echarts'
export default {
  data() {
    return {
      Accuracy: [],
      ClassName: [],
      StandardDeviation: [],
    }
  },
  props: {
    list: Array,
  },
  methods: {
    init() {
      //环形图
      const barLine = echarts.init(this.$refs.barLine)
      // let position= this.isLegend?'outter':'center'

      var option = {
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            crossStyle: {
              color: '#727272',
            },
          },
          formatter: (params) => {
            var str = params[0].axisValue + '<br>'
            params.forEach((vals) => {
              let o = ''
              o = vals.seriesIndex == 0 ? '%' : ''
              str = str + ` ${vals.marker} ${vals.seriesName}:${vals.value}${o}<br>`
            })
            return str
          },
        },
        grid: {
          top: '60px',
          // containLabel: true,
        },
        dataZoom: [
          {
            start: 0, // 默认为0
            end: 75, // 默认为100
             type: 'slider',
            show: true,
            xAxisIndex: [0],
            handleSize: 0, // 滑动条的 左右2个滑动条的大小
            height: 10, // 组件高度
            // left: '10%', // 左边的距离
            // right: '10%', // 右边的距离
            bottom: 12, // 右边的距离
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
        /*         toolbox: {
          feature: {
            // dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ['line', 'bar'] },
            restore: { show: true },
            // saveAsImage: { show: true },
            color: '#727272',
          },
        }, */
        legend: {
          data: ['正确率', '标准差'],
          top: '20px',
 textStyle: {
          color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
            
          },
          
        },
        xAxis: [
          {
            type: 'category',
            data: this.ClassName,
            axisPointer: {
              type: 'line',
              label: {
                show: true,
                color: '#fff',
                // fontSize: '10',
                backgroundColor: '#FFB718',
              },
            },
            axisTick: false,
            axisLabel: {
              color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
            
            },
            axisLine: {
              lineStyle: {
                color: '#D6DCDE',
              },
            },
          },
        ],
        yAxis: [
          {
            type: 'value',
            name: '',
            min: 0,
            // max: 100,
            interval: 50,
            axisLabel: {
              formatter: '{value} %',
            },
            axisTick: false,
            axisLine: {
              show: false,
            },
            axisLabel: {
               color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
            
            },
          },
          {
            type: 'value',
            name: '',
            // min: 0,
            // max: 15,
            interval: 5,
            axisLabel: {
              // formatter: '{value} ',
            },
            axisTick: false,
            axisLine: {
              show: false,
            },
            axisLabel: {
               color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
            
            },
          },
        ],
        series: [
          {
            name: '正确率',
            type: 'bar',
            data: this.Accuracy,
            barWidth: '18',
            color: '#479AEB',
            itemStyle: {
              barBorderRadius: [8, 8, 3, 3],
            },
          },

          {
            name: '标准差',
            type: 'line',
            yAxisIndex: 1, //使用第二个y坐标轴
            data: this.StandardDeviation,
            color: '#FFB718',
            lineStyle: {
              width: 3,
            },
          },
        ],
      }
      if (this.list.length < 4) {
        option.dataZoom = []
      }

      barLine.setOption(option)
    },
    formatArr() {
      this.list.forEach((val) => {
        this.Accuracy.push(val.Accuracy)
        if(typeof val.ClassName !='undefined'){
 this.ClassName.push(val.ClassName)
        }
        if(typeof val.TeacherName !='undefined'){
           this.ClassName.push(val.TeacherName)
        }
        // this.ClassName.push(val.ClassName)
        this.StandardDeviation.push(val.StandardDeviation)
      })
    },
  },
  mounted() {
    this.formatArr()
    this.init()
  },
}
</script>

