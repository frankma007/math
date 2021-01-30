<template>
  <div>
    <p style="margin-bottom: 30px;">
      <span class="f-t">版块环比分析</span>
      <span >
        <!-- 班级 -->
        <a-select v-if="gradeDefalutId" style="width: 120px" @change="handleChangeGrade" :default-value="gradeDefalutId">
          <a-select-option :value="item.GradeId" v-for="item in classInfor" :key="item.GradeId">
            {{ item.GradeName }}
          </a-select-option>
        </a-select>
      </span>
    </p>
    <div style="position: relative;">
      <!-- <span class="left-arrow cur" slot="button-prev" @click="prevPhoto" >
        <img class="arrow-img" src="@/assets/student/returnleft.png" />
      </span> -->
      <!-- <swiper class="swiper" ref="swiper" :options="swiperOption">
        <swiper-slide
          style="text-align:center;"
          v-for="(item, index) in plateData"
          :key="index">
          <div > -->
      <span v-for="(item, index) in plateData" :key="index" @click="handleStudnet(item.ObjectId)" :class="{'student-select': firstStu === item.ObjectId}" class="student-list" >{{ item.ObjectName }}</span>
      <!-- </div> -->
      <!-- </swiper-slide>
      </swiper> -->

      <!-- <span class="right-arrow cur" slot="button-next" >
        <img @click="nextPhoto" class=" arrow-img cur" src="@/assets/student/returnbtn.png" />
      </span> -->
    </div>

    <div class="pop-info">
      <div v-if="showPlate" class="default-unit">
        <img src="@/assets/lack/暂无关注好友.png" alt="">
        <p>暂无版块环比分析数据</p>
      </div>
      <!--  -->
      <div v-show="!showPlate">
        <div class="plate-analysis" ref="plateAnalysis"></div>
      </div>
      <div class="notes">
        <p>页面注释：</p>
        <p>1.本页各班排名计算方式为各班平均综合得分在所有班级的排名，其中高位指排名前20%的班级，中位指排名20%-60%的班级，低位指排名60%以后的班级。</p>
        <p>2.综合得分计算方法为学生在各题型版块下整体正确率及答题速度在全年级排名得分的加权平均值。</p>
      </div>
    </div>
  </div>
</template>

<script>
import echarts from 'echarts'
import { Swiper, SwiperSlide } from 'vue-awesome-swiper'
export default {
  components: {
    Swiper,
    SwiperSlide
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    this.schoolId = localStorage.getItem('SchoolId')
    this.getClassInfo()
  },
  mounted () {
    this.init()
  },
  data () {
    return {
      schoolId: '',
      // 缺省
      showPlate: false,
      userId: '',
      classInfor: [], // 年级信息
      plateData: [],
      // 默认年级id
      gradeDefalutId: '',
      swiperOption: {
        slidesPerView: 2,
        spaceBetween: 30,
        direction: 'horizontal',
        navigation: {
          nextEl: '.swiper-button-next',
          prevEl: '.swiper-button-prev',
          hideOnClick: true
        }

        // on: {
        //   resize: () => {
        //     this.$refs.swiper.changeDirection(
        //       window.innerWidth <= 960
        //         ? 'vertical'
        //         : 'horizontal'
        //     )
        //   }
        // }
      },
      firstStu: '',
      arr: []
    }
  },
  methods: {
    prevPhoto () {
      // console.log('左')
      this.$refs.swiper.$swiper.slidePrev()
    },
    nextPhoto () {
      // console.log('右')
      this.$refs.swiper.$swiper.slideNext()
    },
    init () {
      const plateAna = echarts.init(this.$refs.plateAnalysis)
      var hidddenStyle = {
        normal: {
          color: 'rgba(0,0,0,0)'
        },
        emphasis: {
          color: 'rgba(0,0,0,0)'
        }
      }
      const option = {
        toolbox: {

        },
        legend: {
          x: 'center', // 可设定图例在左、右、居中
          y: 'bottom', // 可设定图例在上、下、居中
          data: ['上学期', '下学期']
        },
        xAxis: [
          {
            type: 'category',
            data: ['数与运算', '图形与几何', '统计与概率', '方程与代数 '],
            axisPointer: {
              type: 'shadow'
            }
          }
        ],
        yAxis: [
          {
            type: 'value',
            name: '综合得分',
            min: 0,
            max: 10,
            interval: 2,
            axisLabel: {
              formatter: '{value}'
            }
          }
        ],
        series: [
          {
            name: '上学期',
            type: 'bar',
            data: [2.0, 4.9, 7.0, 4.9],
            barWidth: 30,
            itemStyle: {
              normal: {
                color: '#A4A7C9',
                label: { // ---图形上的文本标签
                  // formatter: function (params) {
                  //   // console.log(params)
                  // },
                  // show: true,
                  // position: 'top', // ---相对位置
                  // // rotate: 0, // ---旋转角度
                  // textStyle: { // 数值样式
                  // color: 'red'
                  // padding: '10px',
                  // border: '1px solid #A4A7C9'
                  // }
                }
              }
            }
          },
          {
            name: '下学期',
            type: 'bar',
            data: [2.6, 5.9, 9.0, 4.9],
            barWidth: 30,
            itemStyle: {
              normal: {
                show: true,
                position: 'top', // ---相对位置
                // rotate: 20, // ---旋转角度
                color: '#F77474',
                formatter: function (params) {
                  console.log(params)
                }

              }
            }
          }
        ]
      }
      const seriesAdd = {
        type: 'line',
        itemStyle: hidddenStyle,
        symbol: 'circle', // 将线图数据点设置为实心圆，因为空心会留下白点……
        silent: true, // 使线图不触发鼠标事件
        label: {
          show: true,
          color: '#000',
          position: 'top',
          // formatter: (params) => {
          //   console.log('图表---', params)
          //   return params.value + '高位保持'
          // },
          textStyle: {
            color: '#A4A7C9',
            textBorderWidth: '1px solid #ddd'
          },
          borderStyle: {
            border: '1px solid #ddd'
          }
        },
        data: []
      }
      option.series.push(seriesAdd)
      plateAna.setOption(option)
      window.addEventListener('resize', function () {
        plateAna.resize()
      })
    },
    getClassInfo () {
      this.$uwonhttp.post('/Manager/ManagerAnalysis/GetGrade', { UserId: this.userId }).then(res => {
        console.log('学校年级---', res)
        this.classInfor = res.data.Data.filter(value => {
          return value.GradeId !== 0
        })
        // this.classInfor = res.data.Data
        this.gradeDefalutId = res.data.Data[1].GradeId
        // console.log('默认id', this.gradeDefalutId)
        this.getPlateAna()
      })
    },
    getPlateAna () {
      this.$uwonhttp.post('/report/SchoolDeepAnalysis/SequentialPlateCompar', { schoolid: this.schoolId, greadid: this.gradeDefalutId }).then(res => {
        console.log('环形对比---', res)
        this.showPlate = false
        if (res.data.Data.length === 0) {
          this.showPlate = true
          return
        }
        this.plateData = res.data.Data
        // 默认学生
        this.firstStu = res.data.Data[0].ObjectId
        console.log('默认学生', this.firstStu)
        this.switchStu = res.data.Data[0]
        const chapter = res.data.Data[0].Plates.map(value => {
          return value.PlatName
        })
        const position = res.data.Data[0].Plates.map(value => {
          return value.ScoreResult
        })
        const arr = res.data.Data[0].Plates

        res.data.Data[0].Plates.forEach(value1 => {
          this.arr.push(value1.TermScores.filter(item => {
            return item.Term === 1 || item.Term === 2
          }))
        })
        const ob = this.arr.map(item => {
          return item[0]
        })
        const ob1 = this.arr.map(item => {
          return item[1]
        })
        // console.log('=====', chapter)
        console.log('=====', this.arr)
        console.log('======', ob)
        // console.log('======', ob1)
        // 获取最大数
        const obc = []
        for (let index = 0; index < this.arr.length; index++) {
          if (this.arr[index][0].Score > this.arr[index][1].Score) {
            obc.push(this.arr[index][0].Score)
          } else {
            obc.push(this.arr[index][1].Score)
          }
        }
        console.log(obc)

        const upsemester = ob.map(value => {
          return value.Score
        })
        const downsemester = ob1.map(value => {
          return value.Score
        })
        console.log('上学期---', upsemester)
        console.log('下学期---', downsemester)

        // 获取位置关系
        const stuNum = []
        for (let i = 0; i < arr.length; i++) {
          console.log('哈哈哈', arr[i])
          stuNum.push({ value: upsemester[i], name: position[i] })
        }
        const plateAna = echarts.init(this.$refs.plateAnalysis)
        const option = {
          xAxis: [
            {
              type: 'category',
              data: chapter,
              axisPointer: {
                type: 'shadow'
              }
            }
          ],
          series: [
            {
              name: '上学期',
              type: 'bar',
              data: upsemester
            },
            {
              name: '下学期',
              type: 'bar',
              data: downsemester
            }
          ]
        }
        const seriesAdd = {
          type: 'line',
          label: {
            show: true,
            color: '#000',
            position: 'top',
            formatter: (params) => {
              // console.log('图表---', params, position)
              // console.log('图表---', position)

              // this.firstStu
              for (let i = 0; i < this.plateData.length; i++) {
                const plates = this.plateData[i].Plates
                for (let x = 0; x < plates.length; x++) {
                  const plate = plates[x]
                  if (plate.PlatName === params.name) {
                    return plate.ScoreResult
                  }
                }
              }
            },
            textStyle: {
              color: '#A4A7C9',
              textBorderWidth: '1px solid #ddd'
            },
            borderStyle: {
              border: '1px solid #ddd'
            }
          },
          data: obc
        }
        option.series.push(seriesAdd)
        plateAna.setOption(option)
      })
    },
    // 选择年级
    handleChangeGrade (value) {
      console.log(value)
      this.gradeDefalutId = value
      this.getPlateAna()
    },
    // 选择学生
    handleStudnet (id) {
      console.log('学生id---', id)
      this.firstStu = id
      this.switchStudent(id)
    },
    // 切换学生图表
    switchStudent (id) {
      this.arr = []
      // 默认学生
      // debugger
      const plateList = this.plateData.filter(value => {
        // debugger
        if (value.ObjectId === id) {
          return value
        }
      })
      console.log('最新value', plateList)
      const chapter = plateList[0].Plates.map(value => {
        return value.PlatName
      })
      const position = plateList[0].Plates.map(value => {
        return value.ScoreResult
      })
      const arr = plateList[0].Plates

      plateList[0].Plates.forEach(value1 => {
        this.arr.push(value1.TermScores.filter(item => {
          return item.Term === 1 || item.Term === 2
        }))
      })
      const ob = this.arr.map(item => {
        return item[0]
      })
      const ob1 = this.arr.map(item => {
        return item[1]
      })
      // console.log('=====', chapter)
      console.log('=====', this.arr)
      console.log('======', ob)
      // console.log('======', ob1)
      // 获取最大数
      const obc = []
      for (let index = 0; index < this.arr.length; index++) {
        if (this.arr[index][0].Score > this.arr[index][1].Score) {
          obc.push(this.arr[index][0].Score)
        } else {
          obc.push(this.arr[index][1].Score)
        }
      }
      console.log(obc)

      const upsemester = ob.map(value => {
        return value.Score
      })
      const downsemester = ob1.map(value => {
        return value.Score
      })
      // 获取位置关系
      const stuNum = []
      for (let i = 0; i < arr.length; i++) {
        console.log('哈哈哈', arr[i])
        stuNum.push({ value: upsemester[i], name: position[i] })
      }
      const plateAna = echarts.init(this.$refs.plateAnalysis)
      const option = {
        xAxis: [
          {
            type: 'category',
            data: chapter,
            axisPointer: {
              type: 'shadow'
            }
          }
        ],
        series: [
          {
            name: '上学期',
            type: 'bar',
            data: upsemester
          },
          {
            name: '下学期',
            type: 'bar',
            data: downsemester
          }
        ]
      }
      const seriesAdd = {
        type: 'line',
        label: {
          show: true,
          color: '#000',
          position: 'top',
          formatter: (params) => {
            console.log('图表---', params, position)
            // console.log('图表---', position)
            for (let i = 0; i < plateList.length; i++) {
              // debugger
              // params.value + this.plateData[i]
              const plates = plateList[i].Plates
              for (let x = 0; x < plates.length; x++) {
                const plate = plates[x]
                if (plate.PlatName === params.name) {
                  return plate.ScoreResult
                }
              }
            }
          },
          textStyle: {
            color: '#A4A7C9',
            textBorderWidth: '1px solid #ddd'
          },
          borderStyle: {
            border: '1px solid #ddd'
          }
        },
        data: obc
      }
      option.series.push(seriesAdd)
      plateAna.setOption(option)
    }
  }
}
</script>

<style lang="less" scoped>
  .f-t {
    font-size: 22px;
    margin-right: 30px;
  }
  .default-unit {
    margin-top: 0;
    padding-top: 60px;
  }
  .plate-analysis {
    width: 100%;
    height: 450px;
  }
  .student-list {
    display: inline-block;
    margin: 0 30px 15px 0;
    padding: 5px 8px;
    background-color: #ddd;
    border-radius: 20px;
    cursor: pointer;
  }
  // .swiper {
    .student-select {
      color: #fff;
      background-color: #68BB97;
    }
  // }

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

  .swiper-slide {
    width: 100px !important;
  }
  .left-arrow {
    position: absolute;
    left: -23px;
    top: 1px;
    z-index: 2;
  }
  .right-arrow {
    position: absolute;
    right: -47px;
    z-index: 12;
    top: -4px;
  }
</style>
