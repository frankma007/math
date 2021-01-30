<template>
  <div>
    <p class="tit">成绩统计</p>
    <div class="prc-lack" v-if="showLack">
      <img src="@/assets/lack/暂无搜索记录.png" alt="">
      <p>暂无学生提交练习</p>
    </div>
    <div v-if="!showLack">
      <div class="chart" ref="chart"></div>
      <div class="student-answer">
        <a-table
          :columns="columns"
          :data-source="data"
          :pagination="{ pageSize: 20 }"
          :loading="loading"
          :scroll="{ y: 447 }"
          bordered>
          <template slot="cao" slot-scope="text,record">
            <!-- {{ record }} 表格数据 -->
            <span @click="toSeeTack(record.StudentName,record.StudentId)" class="cur" v-if="record.DataState === 1">查看</span>
            <span @click="toSeeTack(record.StudentName,record.StudentId,'1')" class="cur" v-if="record.DataState === 2">待订正</span>
            <span @click="toCorrection" class="cur" v-if="record.DataState === 3">订正批改</span>
          </template>
        </a-table>
      </div>
    </div>
  </div>
</template>

<script>
import echarts from 'echarts'
const columns = [
  {
    title: '学号',
    dataIndex: 'StudentNo',
    align: 'center',
    key: 'StudentNo',
    width: 60
  },
  {
    title: '学生',
    dataIndex: 'StudentName',
    align: 'center',
    key: 'Index',
    scopedSlots: { customRender: 'Index' },
    width: 60
  },
  {
    title: '等第评价',
    dataIndex: 'Epilogue',
    align: 'center',
    width: 80
  },
  {
    title: '提交时间',
    dataIndex: 'SubmitDateTime',
    align: 'center',
    // ellipsis: true,
    width: 80
  },
  {
    title: '详情',
    dataIndex: 'DoTime',
    align: 'center',
    key: 'DoTime',
    width: 80,
    scopedSlots: { customRender: 'cao' }
  }
  // {
  //   title: '分数等级',
  //   dataIndex: 'Rank',
  //   align: 'center',
  //   key: 'Rank',
  //   width: 80
  // },
  // {
  //   title: '订正状态',
  //   dataIndex: 'RevisionStates',
  //   align: 'center',
  //   key: 'RevisionStates',
  //   width: 80,
  //   scopedSlots: { customRender: 'RevisionStates' }
  // },
  // {
  //   title: '提交时间',
  //   dataIndex: 'CreateTime',
  //   align: 'center',
  //   key: 'CreateTime',
  //   width: 80
  // },
  // {
  //   title: '操作',
  //   dataIndex: '5',
  //   align: 'center',
  //   width: 150,
  //   scopedSlots: { customRender: 'cao' }
  // }
]
const data = [

]
export default {
  created () {
    this.classId = this.$route.query.classId
    this.customId = this.$route.query.customId
    this.getColumnar()
    this.getFormData()
  },
  mounted () {
    this.ring()
  },
  watch: {
    $route () {
      this.classId = this.$route.query.classId
      this.customId = this.$route.query.customId
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/IntelligentCorrection/CompletePractive?' + ChangId) {
        this.getColumnar()
        this.getFormData()
      }
    }
  },
  data () {
    return {
      showLack: false, // 缺省
      classId: '',
      customId: '', // 自定义练习id
      columns, // 行
      data, // 列
      loading: true, // 加载表格
      rank: '优秀'
      // formList: []
    }
  },
  methods: {
    ring (datas) {
      const myChart = echarts.init(this.$refs.chart)

      myChart.setOption({
        xAxis: {
          type: 'category',
          data: ['优秀', '良好', '合格', '须努力']
        },
        yAxis: {
          type: 'value'
        },
        color: ['#B0CB5C', '#91CDB3', '#28B7FF', '#28B7FF'],
        series: [{
          data: [1, 2, 2, 2],
          type: 'bar',
          itemStyle: {
            normal: {
              label: { // ---图形上的文本标签
                formatter: function (params) {
                  // console.log(params)
                  // console.log(params.value)
                  // if (params.name === '100') {
                  //   return params.value + '人优秀' + '{img1|}'
                  // } else if (params.name === '90-99') {
                  //   return `${params.value}人优秀`
                  // } else if (params.name === '80-89') {
                  //   return `${params.value}人良好`
                  // } else if (params.name === '60-79') {
                  //   return `${params.value}人合格`
                  // } else if (params.name === '0-59') {
                  //   return `${params.value}人须努力`
                  // } else {
                  //   return `${params.value}人未做`
                  // }
                },
                rich: {
                  // img1: {
                  //   backgroundColor: {
                  //     image: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1605074303494&di=3871f451c5a232233ddd64cb07969063&imgtype=0&src=http%3A%2F%2Fbpic.588ku.com%2Felement_origin_min_pic%2F18%2F02%2F24%2Fee79cb33232a6f2bcce1512cfa251993.jpg',
                  //   },
                  //   height: 20,
                  //   borderRadius: 5
                  // }
                },
                show: true,
                position: 'top', // ---相对位置
                // rotate: 0, // ---旋转角度
                color: '#000'
              },
              color: function (params) {
                // 注意，如果颜色太少的话，后面颜色不会自动循环，最好多定义几个颜色
                var colorList = ['#B0CB5C', '#91CDB3', '#28B7FF', '#FF9166']
                return colorList[params.dataIndex]
              }
            }
          },
          barWidth: '45',				// ---柱形宽度
          barCategoryGap: '30%' // ---柱形间距
        }],
        axisLabel: {					// ---坐标轴 标签
          show: true,					// ---是否显示
          inside: false,				// ---是否朝内
          rotate: 0,					// ---旋转角度
          margin: 5					// ---刻度标签与轴线之间的距离
          // color:'red',				//---默认取轴线的颜色
        }
      })
      var that = this
      myChart.on('click', function (params) {
        this.loading = true
        that.rank = params.name
        console.log('点击图表', params.name)
        that.getFormData()
      })
      // 设置图表随窗口大小变化
      window.addEventListener('resize', function () {
        myChart.resize()
      })
    },
    // 获取柱状图数据
    getColumnar () {
      this.$uwonhttp.post('/Customer/CustomExercise/GetCustomExerciseScoreReport', { classid: this.classId, customExerciseId: this.customId }).then(res => {
        console.log('柱状图', res)
        const lack = res.data.Data.Histogram.every(value => {
          return value.StudentCnt === 0
        })
        this.showLack = lack
        const Etc = res.data.Data.Histogram.map(value => {
          return value.StudentCnt
        })
        const myChart = echarts.init(this.$refs.chart)
        myChart.setOption({
          series: [{
            data: Etc
          }]
        })
      })
    },
    // 表格
    getFormData () {
      this.$uwonhttp.post('/Customer/CustomExercise/GetCustomExerciseStudentScoreReport', { classid: this.classId, customExerciseId: this.customId, epilogue: this.rank }).then(res => {
        console.log('表格', res)
        this.data = res.data.Data
        this.loading = false
      })
    },
    // 查看
    toSeeTack (name, id, waitRevision) {
      this.$router.push({ path: '/Teacher/IntelligentCorrection/TaskSituation',
        query: {
          studentName: name,
          studentId: id,
          customId: this.customId,
          waitRevision
        }
      })
    },
    // 去批改
    toCorrection () {
      this.$router.push({ path: '/Teacher/IntelligentCorrection/IntelligentCorrection',
        query: {
          customId: this.customId,
          classId: this.classId
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .tit {
    font-size: 18px;
  }
  // 缺省
  .prc-lack {
    margin-top: 200px;
    text-align: center;
    color: #ccc;
  }
  // 图表
  .chart {
      height: 409px;
      margin-bottom: 30px;
      background-color: #fff;
      border-radius: 5px;
    }
  // 表格
  /deep/.ant-table-thead {
    /deep/.ant-table-row-cell-break-word {
      color: #fff;
      background-color: #68BB97;
    }
  }
</style>
