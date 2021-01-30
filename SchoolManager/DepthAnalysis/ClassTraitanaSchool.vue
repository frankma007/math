<template>
  <div>
    <p class="m-b">
      <span class="f-t">班级特点分析</span>
      <span >
        <!-- 班级 -->
        <a-select v-if="gradeDefalutId" style="width: 120px" @change="handleChangeGrade" :default-value="gradeDefalutId">
          <a-select-option :value="item.GradeId" v-for="item in classInfor" :key="item.GradeId">
            {{ item.GradeName }}
          </a-select-option>
        </a-select>
      </span>
    </p>
   <div class="pop-info">
      <div class="student-traitana" ref="studentTraitana">

    </div>
    <div class="notes">
      <p>页面注释：</p>
      <p>1.本页面按照班级综合正确率和答题速度在全区同年级的排名构建，旨在帮助管理者发现班级特性并进行针对性教学规划。</p>
      <p>2.图中各点代表各班级，点击显示其正确率得分和答题速度得分。</p>
      <p>3.第一象限内为做题又快又好的班级，第二象限内为做题较差但速度很快的班级，第三象限为做题很差速度也很慢的班级，第四象限为做题很好但速度很慢的班级。</p>
    </div>
   </div>
  </div>
</template>

<script>
import echarts from 'echarts'
import ecStat from 'echarts-stat'
export default {
  components: {
    ecStat
  },
  created () {
    this.schoolId = localStorage.getItem('SchoolId')
    this.userId = localStorage.getItem('UserId')
    this.getClassInfo()
    
  },
  mounted () {
    this.init()
  },
  data () {
    return {
      schoolId: '',
      userId: '',
      classInfor: [], // 年级的信息
      gradeDefalutId: '', // 默认年级id
      // arr: []
    }
  },
  methods: {
    init () {
      const studentTraitana = echarts.init(this.$refs.studentTraitana)

      const option1 = {
       
        grid: {
          top: '10%',
          left: '3%',
          right: '7%',
          bottom: '1%',
          containLabel: true,
        },
        tooltip: {
          showDelay: 0,
          formatter: (params) => {
            if (params.value.length > 1) {
              return `${params.seriesName}:<br/>正确率得分：${params.value[0]}<br/>答题速度得分：${params.value[1]}`;
            }
            return false;
          },

        },
        legend: [
          {
            orient: 'horizontal',
            x: '89%',
            y: '7%',
            align: 'left',
            data: ['二'],
            textStyle: {
              fontSize: 14,
            },
          },
          {
            orient: 'horizontal',
            x: '94%',
            y: '7%',
            align: 'left',
            data: ['一'],
            textStyle: {
              fontSize: 14,
            },
          },
          {
            orient: 'horizontal',
            x: '89%',
            y: '10%',
            align: 'left',
            data: ['四'],
            textStyle: {
              fontSize: 14,
            },
          },
          {
            orient: 'horizontal',
            x: '94%',
            y: '10%',
            align: 'left',
            data: ['三'],
            textStyle: {
              fontSize: 14,
            },
          },
        ],
        xAxis: [{
          type: 'value',
          name: '正确率得分',
          min: 0,
          max: 10,
          interval:1, // 步长
          scale: true,
          splitLine: {
            show: false,
          },
          data: [0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1],
        }],
        yAxis: [{
          type: 'value',
          scale: true,
          min: 0,
          max: 10,
          interval:1, // 步长
          name: '答题速度得分',
          splitLine: {
            show: false,
          },
        }],
        series: [
          {
            name: '1',
            z: 3,
            type: 'scatter',
            symbolSize: 12,
            itemStyle: {
              normal: {
                color: '#EF535C',
              },
            },
            data: [
              [4, 6.5], [1, 5.9], [3, 6.3], [2.6, 7.90],
            ],
          },
          {
            name: '2',
            z: 3,
            type: 'scatter',
            symbolSize: 12,
            itemStyle: {
              normal: {
                color: '#6FE12F',
              },
            },
            data: [[6, 8], [7, 6.6], [8, 5.4], [8.9, 9],
            ],
          },

          {
            name: '3',
            type: 'scatter',
            data: [[3, 2], [4, 2.8]],
            itemStyle: {
              normal: {
                color: '#8B4DF6',
              },
            },
          },
          {
            name: '4',
            type: 'scatter',
            data: [[6, 3]],
            itemStyle: {
              normal: {
                color: '#05D6DE',
              },
            },

          },
          {
            name: 'wu',
            type: 'scatter',
            data: [[5, 0]],
            itemStyle: {
              normal: {
                color: '#999',
              },
            },
            markLine: {
              data: [
                {
                  type: 'average',
                  name: 'wu',
                  valueIndex: 0,
                },
              ],
              label: {
                show: false,
              },
            },

          },
          {
            name: 'six',
            type: 'scatter',
            data: [[0, 5]],
            itemStyle: {
              normal: {
                color: '#999',
              },
            },
            markLine: {
              data: [
                {
                  type: 'average',
                  name: 'six',
                },
              ],
              label: {
                show: false,
              },
            },
          },
        ],
      }
      studentTraitana.setOption(option1)
      window.addEventListener('resize', function () {
        studentTraitana.resize()
      })
    },
    // 获取老师所带班级
    getClassInfo () {
      this.$uwonhttp.post('/Manager/ManagerAnalysis/GetGrade', { UserId: this.userId }).then(res => {
        console.log('学校年级---', res)
        this.classInfor = res.data.Data.filter(value => {
          return value.GradeId !== 0
        })
        this.gradeDefalutId = res.data.Data[1].GradeId
        console.log('学校年级', this.classInfor)
        this.studentTraitanaAna()
      })
    },
    // 学生特点分析
    studentTraitanaAna () {
      this.$uwonhttp.post('/report/SchoolDeepAnalysis/StudentDoItemAttribute', { schoolid: this.schoolId,greadid: this.gradeDefalutId }).then(res => {
        console.log('学生特点分析---', res)
        const studentTraitana = echarts.init(this.$refs.studentTraitana)
          let option = {}
          option.series = []
        res.data.Data.map( (value, index) => {
          // console.log(value)
          // debugger
          const arr = []
          const arr1 = []
          const arr2 = []
          const arr3 = []
          const arr4 = []
          arr.push([value.AccuracyScore, value.AnswerSpendTimeScore],value.Studentlist)
          console.log(arr)
         
          if (value.AnswerSpendTimeScore < 5 && value.AccuracyScore > 5) {
            arr1.push([value.AccuracyScore, value.AnswerSpendTimeScore],value.Studentlist)
          } else if (value.AnswerSpendTimeScore > 5 && value.AccuracyScore > 5) {
            arr2.push([value.AccuracyScore, value.AnswerSpendTimeScore],value.Studentlist)
          } else if (value.AnswerSpendTimeScore < 5 && value.AccuracyScore < 5 ) {
            arr3.push([value.AccuracyScore, value.AnswerSpendTimeScore],value.Studentlist)
          } else if (value.AnswerSpendTimeScore > 5 && value.AccuracyScore < 5) {
            arr4.push([value.AccuracyScore, value.AnswerSpendTimeScore],value.Studentlist)
          }
          console.log('数据', arr1[0])
          // const revArr1 = arr1[0].reverse()
          // console.log('翻转数据', revArr1)
          // debugger
           option.series.push(
              {
                name: value.Studentlist,
                type: 'scatter',
                data: [arr1[0]],
                 itemStyle: {
                  normal: {
                    color: '#73B3DF',
                  }
                }
              },
              {
                name: value.Studentlist,
                type: 'scatter',
                data: [arr2[0]],
                 itemStyle: {
                  normal: {
                    color: '#EFC47A',
                  }
                }
              },
              {
                name: value.Studentlist,
                type: 'scatter',
                data: [arr3[0]],
                 itemStyle: {
                  normal: {
                    color: '#FF9797',
                  }
                }
            },
            {
              name: value.Studentlist,
              type: 'scatter',
              data: [arr4[0]],
              itemStyle: {
                  normal: {
                    color: '#7ED88B',
                  }
                }
            },
            {
              name: 'wu',
              type: 'scatter',
              data: [[5, 0]],
            },
            {
              name: 'six',
              type: 'scatter',
              data: [[0, 5]],
            }
           )
           return option.series
           
        })
        console.log('OPTION===', option)
       studentTraitana.setOption(option)
      })
    },
    // 选择年级
    handleChangeGrade (value) {
      this.gradeDefalutId = value
      this.studentTraitanaAna()
      console.log(value)
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
  .student-traitana {
    width: 100%;
    height: 450px;
  }
  .notes {
    margin-top: 20px;
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
</style>
