<template>
  <div ref="annular"></div>
</template>

<script>
import echarts from 'echarts'
export default {
  props: {
    list: Array,
  },
  data() {
    return {
      /*       ChapterId: [],
      ChapterName: [],
      ColorHEX: [],
      Proportion: [], */
      newArr: [],
      color:[],
      ChapterName:[],
    }
  },

  methods: {
    init() {
      //环形图
      const annular = echarts.init(this.$refs.annular)
      //   let position= this.isLegend?'outter':'center'

      var option = {
       

        color: this.color,
        tooltip: {
          show: false,
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)',
        },
        // 图例组件。
        legend: {
          show: true,
          selectedMode: false, //不许点击触发事件
          orient: 'vertical',
          left: 10,
          top: 10,
          data: this.ChapterName,
          textStyle: {
            fontSize: 14,            
            fontFamily:[ "Microsoft YaHei",'sans-serif','Tahoma','Arial' ],
          },
        },
        // 系列列表
        series: [
          {
             width: '70%',
            left:'15%',
            height:'60%',
            top: '30%',
            bottom:'0',
            // name: '环形图',
            type: 'pie',
            radius: ['40%', '70%'],
             center: ['50%', '50%'] ,
            avoidLabelOverlap: false,
            label: {
              show: true,
              // position: 'center'

              position: 'outter',
              //   formatter: '{b}: {d}'
              fontSize: 16,
              formatter: function (params) {
                // console.log(params, '啊啊啊啊啊')
                return params.value + '%'
          
              },
            },
            // 高亮的扇区和标签样式。
            emphasis: {
              label: {
                show: true,
                // fontSize: '16',
                fontWeight: 'bold',
              },
            },
            // 图形样式
            itemStyle: {
              show: true,
              borderColor: '#f1f5f6',
              borderWidth: 2,
            },
            labelLine: {
              show: false,
            },
            data: this.newArr,
          },
        ],
      }

      annular.setOption(option)
      annular.on("click", (params)=>{
        // console.log(params.data,'马壮马壮马壮');
        this.$emit('ev_getAnnularItem', params.data);

      })
             setTimeout( ()=>{
        window.onresize = ()=> {
          annular.resize()
        }
      }, 200)
    },
    formatArr() {
      /*       ChapterId: [],
      ChapterName: [],
      ColorHEX: [],
      Proportion: [], */

      // debugger

      this.list.forEach((val) => {
        const { ChapterId, ChapterName, ColorHEX, Proportion } = val
        const obj = {
          value: Proportion,
          name: ChapterName,
          ChapterId,
          color: ColorHEX,
        }
        this.newArr.push(obj)
        this.color.push(val.ColorHEX)
        this.ChapterName.push(val.ChapterName)
      })
    },
  },
  mounted() {
    this.formatArr()
    this.init()
  },
}
</script>

