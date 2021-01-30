<template>
  <div ref="annular"></div>
</template>

<script>
import echarts from 'echarts'
export default {
  data() {
    return {}
  },
  props: {
    value: Object,
  },
  methods: {
    init() {
      //环形图
      const annular = echarts.init(this.$refs.annular)
      // let position= this.isLegend?'outter':'center'

      var option = {
        color: ['#F87175', '#DBEBF0'],
        // hoverAnimation: true,//禁止初始化动画
        // 图例组件。
        legend: {
          show: false,
        },
        // 系列列表
        series: [
          {
             hoverAnimation:false,
           
            legendHoverLink :false,
             selectedMode:false,
            // silent: false,
            selectedMode:false,
            type: 'pie',
            radius: ['40%', '70%'],
            avoidLabelOverlap: false,
            // hoverAnimation: true,
            hoverAnimation: false, //鼠标经过禁止动画
            label: {
              show: true,
              position: 'center',             
              fontSize: '14',
              formatter: function (params) {                
                if (params.dataIndex != 0) {
                  return
                }
                return params.value + '%'
                
              },
            
            },
            // 高亮的扇区和标签样式。
            emphasis: {
              label: {
                show: false,
              },
            },
            // 图形样式
            itemStyle: {
             
              borderColor: '#f1f5f6',
             
            },

            data: [            
              { value: this.value.Accuracy,chapterId:this.value.ItemId,PaperId:this.value.PaperId},
              { value: (100-Number(this.value.Accuracy)),chapterId:this.value.ItemId,PaperId:this.value.PaperId },
            ],
          },
        ],
      }

      annular.setOption(option)
            annular.on("click", (params)=>{
        console.log(params.data,"33333333332");
        //弹窗需要展示的数据
        this.$emit('ev_getSmallAnnular', params.data);

      });
    },
  },
  mounted() {
    // console.log(this.value.value);
    this.init()
  },
}
</script>

<style lang="less" scoped>
</style>
