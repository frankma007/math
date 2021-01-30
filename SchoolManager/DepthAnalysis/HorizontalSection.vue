<template>
  <div>
    <p class="m-b">
      <span class="f-t">水平及版块质量分析</span>
      <span >
        <!-- 班级 -->
        <a-select v-if="classDefalutId " style="width: 120px" @change="handleChangeClass" :default-value="classDefalutId">
          <a-select-option :value="item.GradeId" v-for="item in classInfor" :key="item.GradeId">
            {{ item.GradeName }}
          </a-select-option>
        </a-select>
      </span>
    </p>
    <p class="m-b-1">
      <!-- <span :class="{ 'chapter-color': firstChapter === ite.ChapterId}" @click="handleChapter(ite.ChapterId)" v-for="ite in chapterList" :key="ite.ChapterId" class="chapter-name" >{{ ite.ChapterName }}</span> -->
      <span @click="handleOperation(0)" class="operation-btn cur" :class="{ 'now-click': defaultBtn === 0}">学习水平</span>
      <span @click="handleOperation(1)" class="operation-btn cur" :class="{ 'now-click': defaultBtn === 1}">题型模板</span>
    </p>

    <div class="pop-info">
      <!--  -->
      <div v-if="showUnit" class="default-unit">
        <img src="@/assets/lack/暂无关注好友.png" alt="">
        <p>暂无相关数据</p>
      </div>
      <!-- 图表 -->
      <div v-if="!showUnit">
        <div class="unit-ana" ref="unitAna">
        </div>
      </div>
      <div class="notes">
        <p>页面注释：</p>
        <p>1.本页面图表为异轴组合图，柱状图及左侧纵轴代表和衡量正确率，折线图和右侧纵轴代表和衡量标准差。</p>
        <p>2.标准差越高，代表班级内学生成绩的离散程度越大。</p>
        <p>3.点击对应班级柱可跳转到该班级的成绩分布分析。</p>
      </div>
    </div>
  </div>
</template>

<script>
import echarts from 'echarts'
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    this.schoolId = localStorage.getItem('SchoolId')
    this.getClassInfo()
  },
  mounted () {
    this.init()
  },
  watch: {
    $route () {
      if (this.$route.fullPath === '/Teacher/TeachingEvaluation/PlateAnalysis') {

      }
    }
  },
  data () {
    return {
      defaultBtn: 0, // 当前操作按钮
      // 缺省
      showUnit: false,
      chapterName: 1,
      classInfor: [],
      // 默认年级
      classDefalutId: '',
      gradeId: '',
      // 章节
      chapterList: [],
      // 第一章节id
      firstChapter: '',
      // 单元质量分析
      unitQualityAna: [],
      // 切换章节id
      ok: '',
      chartColor: '',
      JumpPage: '',
      // 本班id
      nowClassId: '',
      schoolId: ''
    }
  },
  methods: {
    init () {
      const unitAna = echarts.init(this.$refs.unitAna)
      unitAna.setOption({
        // color: ['#F7C974'],
        tooltip: {
          trigger: 'axis',
          axisPointer: { // 坐标轴指示器，坐标轴触发有效
            type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
          }
        },
        legend: {
          x: 'center', // 可设定图例在左、右、居中
          y: 'bottom', // 可设定图例在上、下、居中
          bottom: -20,
          data: ['标准差', '正确率'],
          color: ['#A784ED', '#A784ED', '#A784ED']
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            type: 'category',
            data: [],
            axisTick: {
              alignWithLabel: true
            }
          }
        ],
        yAxis: [
          {
            type: 'value',
            name: '单位（%）',
            max: 100,
            min: 0
          },
          {
            type: 'value',
            scale: true,
            name: '',
            max: 2.5,
            min: 0
          }
        ],
        series: [
          // {
          //   name: '本班级',
          //   type: 'bar',
          //   data: [3.6]
          // },
          // {
          //   name: '标准差',
          //   type: 'line',
          //   itemStyle: {
          //     normal: {
          //       show: true,
          //       position: 'top', // ---相对位置
          //       // rotate: 20, // ---旋转角度
          //       color: '#FF8484',
          //       formatter: function (params) {
          //         console.log(params)
          //       }
          //     }
          //   },
          //   data: (function () {
          //     var res = []
          //     var len = 0
          //     while (len < 10) {
          //       res.push((Math.random() * 10 + 5).toFixed(1) - 0)
          //       len++
          //     }
          //     return res
          //   })()
          // },
          // {
          //   name: '正确率',
          //   type: 'bar',
          //   barWidth: '30',
          //   itemStyle: {
          //     normal: {
          //       show: true,
          //       position: 'top', // ---相对位置
          //       // rotate: 20, // ---旋转角度
          //       color: '#F7C974',
          //       formatter: function (params) {
          //         console.log(params)
          //       }

          //     }
          //   },
          //   data: [60, 52, 60, 33, 39, 50, 49, 80]
          // }
        ]
      })
      const that = this
      unitAna.on('click', function (params, e) {
        console.log('点击', params)
        console.log(e)
        if (that.unitQualityAna[params.dataIndex].BelongToMe === 1) {
          that.$router.push({ path: '/Teacher/TeachingEvaluation/ScoreDistribution', query: { className: params.name, gradeId: that.gradeId, classId: that.nowClassId } })
        } else {
          return false
        }
        // for (let i = 0; i < that.unitQualityAna.length; i++) {
        //   // console.log(that.unitQualityAna[i])
        //   if (that.unitQualityAna[i].BelongToMe === 1) {
        //     that.JumpPage = that.unitQualityAna[i].ClassName
        //   }
        // }
        // if (that.JumpPage === params.name) {
        //   that.$router.push({ path: '/Teacher/TeachingEvaluation/ScoreDistribution', query: { className: params.name, gradeId: that.gradeId, classId: that.nowClassId } })
        // } else {
        //   return false
        // }
        // that.$router.push({ path: '/Teacher/TeachingEvaluation/DepthAnalysis' })
      })
      window.addEventListener('resize', function () {
        unitAna.resize()
      })
    },
    getClassInfo () {
      this.$uwonhttp.post('/Manager/ManagerAnalysis/GetGrade', {
        UserId: this.userId
      }).then(res => {
        console.log('老师带领的年级---', res)

        // this.classInfor = res.data.Data
        this.classInfor = res.data.Data.filter(value => {
          return value.GradeId !== 0
        })
        this.classDefalutId = res.data.Data[1].GradeId
        this.gradeId = +res.data.Data[1].GradeId
        this.getUnitAna()
      })
    },
    // getChapter () {
    //   this.$uwonhttp.post('/Chapter/Chapter/GetFirstChapterByGrade', { grade: this.gradeId, IsWeek: 1 }).then(res => {
    //     console.log('年级对应的章节---', res)
    //     this.chapterList = res.data.Data
    //     this.firstChapter = res.data.Data[0].ChapterId
    //     // debugger
    //     this.getUnitAna()
    //   })
    // },
    // 获取单元质量分析
    getUnitAna () {
      this.$uwonhttp.post('/report/SchoolDeepAnalysis/GetAnalysisLevelPlateTypeList', { GradeId: this.gradeId, SchoolId: this.schoolId, type: this.defaultBtn }).then(res => {
        console.log('单元质量分析---', res)
        if (res.data.Data.length === 0) {
          this.showUnit = true
          return false
        }
        this.unitQualityAna = res.data.Data
        const Accuracy = res.data.Data.map(value => {
          return value.Accuracy
        })
        const Deviation = res.data.Data.map(value => {
          return value.Deviation
        })
        const maxY = Math.max(...Deviation)
        const nowMax = Math.ceil(maxY)
        console.log('折线图---', maxY)

        const TypeName = res.data.Data.map(value => {
          return value.TypeName
        })
        // const ClassId = res.data.Data.map(value => {
        //   return value.ClassId
        // })
        // const stuNum = []
        // for (var i = 0; i < this.unitQualityAna.length; i++) {
        //   stuNum.push({ value: Accuracy[i], 'ID': ClassId[i] })
        // }
        const unitAna = echarts.init(this.$refs.unitAna)
        // const that = this
        unitAna.setOption({
          tooltip: {
            trigger: 'axis'
            // axisPointer: { // 坐标轴指示器，坐标轴触发有效
            //   type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
            // },
            // formatter: (params) => {
            //   // // debugger
            //   // // console.log('提示---', params)
            //   // if (that.unitQualityAna[params[0].dataIndex].BelongToMe === 1) {
            //   //   // console.log(that.unitQualityAna[params[0].dataIndex].ClassId)
            //   //   that.nowClassId = that.unitQualityAna[params[0].dataIndex].ClassId
            //   //   return params[0].name + '<br>' + params[1].seriesName + ': ' + params[1].data + '%' + '<br>' + params[0].seriesName + ': ' + params[0].data
            //   // } else {
            //   //   return ''
            //   // }
            // }
          },
          color: ['#FF8484', '#F7C974', '#A784ED'],
          legend: {
            x: 'center', // 可设定图例在左、右、居中
            y: 'bottom', // 可设定图例在上、下、居中
            // bottom: '20%',
            // padding: [0, 0, 0, 0],
            data: [{
              name: '标准差'
            },
            {
              name: '正确率'
            },
            {
              name: '本班级',
              icon: 'roundRect'
            }]
          },
          grid: { // 修改图例和图的位置
            left: '3%',
            right: '4%',
            bottom: '10%',
            containLabel: true
          },
          xAxis: [
            {
              type: 'category',
              data: TypeName,
              axisTick: {
                alignWithLabel: true
              }
            }
          ],
          yAxis: [
            {
              type: 'value',
              name: '单位（%）',
              max: 100,
              min: 0
            },
            {
              type: 'value',
              scale: true,
              max: nowMax,
              min: 0
            }
          ],
          series: [
            {
              name: '标准差',
              type: 'line',
              data: Deviation,
              yAxisIndex: 1,
              barWidth: '70'
            },
            {
              name: '正确率',
              type: 'bar',
              data: Accuracy,
              barWidth: '70',
              itemStyle: {
                // color: function (params) {
                //   // console.log('颜色---', params)

                //   if (that.unitQualityAna[params.dataIndex].BelongToMe === 1) {
                //     return '#A784ED'
                //   } else {
                //     return '#F7C974'
                //   }
                // }
              },
              label: {
                show: false,
                color: '#000',
                position: 'top',
                formatter: (params) => {
                  console.log('图表---', params)
                  // for (let i = 0; i < )
                }
              }
            }
            // {
            //   name: '本班级',
            //   type: 'line',
            //   data: [],
            //   itemStyle: {
            //     color: '#A784ED'
            //   }
            // }
          ]
        })
      })
    },
    // 选择班级
    handleChangeClass (value) {
      this.gradeId = +value
      console.log(value)
      // this.getChapter()
      this.getUnitAna()
    },
    // 选择章节
    handleChapter (id) {
      this.firstChapter = id
      this.getUnitAna()
    },
    // 选择知识点 / 题型版块
    handleOperation (id) {
      this.defaultBtn = id
      this.getUnitAna()
    }
  }
}
</script>

<style lang="less" scoped>
  .f-t {
    font-size: 22px;
    margin-right: 30px;
  }
  .m-b {
    margin-bottom: 29px;
  }
  .m-b-1 {
    margin-bottom: 15px;
  }
  // 缺省
  .default-unit {
    margin-top: 0;
    padding-top: 60px;
  }
  // 操作按钮
  .operation-btn {
    display: inline-block;
    width: 109px;
    height: 30px;
    line-height: 30px;
    margin-right: 24px;
    text-align: center;
    background-color: #ddd;
    border-radius: 15px;
  }
  // 当前点击
  .now-click {
    color: #fff;
    background-color: #68BB97;
  }
  .unit-ana {
    width: 100%;
    height: 450px;
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
