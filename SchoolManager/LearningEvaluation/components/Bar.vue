<template>
  <div ref="annular"></div>
</template>

<script>
import echarts from 'echarts'
export default {
  data() {
    return {
      Proportion: [],
      Time: [],
      IntervalInfos: [],
    }
  },
  props: {
    list: Array,
  },
  methods: {
    init() {
      //环形图
      const annular = echarts.init(this.$refs.annular)
      // let position= this.isLegend?'outter':'center'

      var option = {
        color: ['#0AB7EF'],
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            // 坐标轴指示器，坐标轴触发有效
            type: 'shadow', // 默认为直线，可选为：'line' | 'shadow'
          },
          formatter: function (params) {
            // console.log(params, 'ttt')
            return params[0].data + '%'
          },
        },
        grid: {
          top: '12',
          left: '3%',
          right: '10%',
          bottom: '11%',
          containLabel: true,
        },
        legend: {
          show: true,
          selectedMode: false, //不许点击触发事件
          orient: 'vertical',
          right: 10,
          bottom: 10,
          data: ['做题人数占比'],
        },
        xAxis: [
          {
            type: 'value',
            position: 'top',
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
        ],
        yAxis: [
          {
            type: 'category',
            data: this.Time,

            axisLabel: {
              // formatter:'{value} %',

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
        ],
        series: [
          {
            label: {
              show: true,
              position: 'right',
              color: '#333',
              fontSize: 14,
              fontFamily: ['Microsoft YaHei', 'sans-serif', 'Tahoma', 'Arial'],
              formatter: function (params) {
                return params.value + '%'
              },
            },
            barWidth: '8',
            name: '做题人数占比',
            type: 'bar',
            // barWidth: '60%',
            data: this.Proportion,
            itemStyle: {
              //柱形图圆角，初始化效果
              barBorderRadius: [2, 4, 4, 2],
            },
          },
        ],
      }
      annular.setOption(option)
      setTimeout(() => {
        window.onresize = () => {
          annular.resize()
        }
      }, 200)
      annular.on('click', (params) => {
        // console.log(this.IntervalInfos[params.dataIndex],'马壮马壮马壮');
        //弹窗需要展示的数据
        this.$emit('ev_getBarItem', this.IntervalInfos[params.dataIndex], params.name)
      })
    },
    formatArr() {
      this.list.forEach((val) => {
        //         BeginTime: "18:00:00"
        // EndTime: "19:00:00"
        // Proportion: 0
        // Time: "18:00-18:59"
        const { BeginTime, EndTime } = val
        this.Proportion.push(val.Proportion)
        this.Time.push(val.Time)
        this.IntervalInfos.push({ BeginTime, EndTime })
      })
    },
  },
  mounted() {
    this.formatArr()
    this.init()
  },
}
</script>


