<template>
  <div id="app">
    <div id="maxoutd">
      <div id="time-d">
        <img src="@/assets/schoolmamager/homepage/编组.png">&nbsp;&nbsp;&nbsp;
        <span>{{ data }}</span>&nbsp;&nbsp;&nbsp;
        <span>{{ time }}</span>&nbsp;&nbsp;&nbsp;
        <span>{{ week }}</span>
      </div>
      <div class="lay-d" id="laydfirset">
        <div id="gradeLevel">
          <!-- 年级图标 -->
          <div class="gradeColor" style="background-color:#5CC0FF;"></div>&nbsp;一年级&nbsp;&nbsp;&nbsp;
          <div class="gradeColor" style="background-color:#FACC14;"></div>&nbsp;二年级&nbsp;&nbsp;&nbsp;
          <div class="gradeColor" style="background-color:#448870;"></div>&nbsp;三年级&nbsp;&nbsp;&nbsp;
          <div class="gradeColor" style="background-color:#79BEBD;"></div>&nbsp;四年级&nbsp;&nbsp;&nbsp;
          <div class="gradeColor" style="background-color:#F57579;"></div>&nbsp;五年级&nbsp;&nbsp;&nbsp;
          <br>
          <br>
          <div id="showEcharts">
            <!-- 展示正确率 -->
            <p class="accuracy">   正确率</p><p class="finish">   完成率</p>
            <div id="echarts1" ref="echarts1">
              
            </div>
            <!-- 展示完成率 -->
            <div id="echarts2" ref="echarts2">
              
            </div>
            <div id="turePrecent"  class="Precent">
              <div class="showPrecent" style="color:#5CC0FF;">{{this.Accuracy[0]}}%</div>
              <div class="showPrecent" style="color:#FACC14;">{{this.Accuracy[1]}}%</div>
              <div class="showPrecent" style="color:#448870;">{{this.Accuracy[2]}}%</div>
              <div class="showPrecent" style="color:#79BEBD;">{{this.Accuracy[3]}}%</div>
              <div class="showPrecent" style="color:#F57579;">{{this.Accuracy[4]}}%</div>
            </div>
            <div id="finshPrecent" class="Precent">
              <div class="showPrecent" style="color:#5CC0FF;">{{this.Finish[0]}}%</div>
              <div class="showPrecent" style="color:#FACC14;">{{this.Finish[1]}}%</div>
              <div class="showPrecent" style="color:#448870;">{{this.Finish[2]}}%</div>
              <div class="showPrecent" style="color:#79BEBD;">{{this.Finish[3]}}%</div>
              <div class="showPrecent" style="color:#F57579;">{{this.Finish[4]}}%</div>
            </div>
          </div>
          <!-- 登录和做卷信息 -->
          <div class="allMessageShow">
            <div class="mess-top">
              <div class="message-div" style="top:-16px;width:50px"><img src="@/assets/schoolmamager/homepage/icon-登录人数.png" alt=""></div>
              <div class="message-div">今日登陆人数<br><span class="messageNum">{{this.loginMessage.TodayLogin}}</span></div><!--span 使用v-text或v-html绑定值 -->
              <div class="message-div">今日做卷人数<br><span class="messageNum">{{this.loginMessage.TodayDoPeopleCount}}</span></div>
              <div class="message-div">注册总人数<br><span class="messageNum">{{this.loginMessage.TotalRegister}}</span></div>
            </div>
             <div class="mess-top">
              <div class="message-div" style="top:-16px;width:50px"><img src="@/assets/schoolmamager/homepage/icon-注册总人数.png" alt=""></div>
              <div class="message-div">总做卷人数<br><span class="messageNum">{{this.loginMessage.TotalDoPeopleCount}}</span></div>
              <div class="message-div">今日做卷张数<br><span class="messageNum">{{this.loginMessage.TodayDoPaperCount}}</span></div>
              <div class="message-div">总做卷次数<br><span class="messageNum">{{this.loginMessage.TotalDoPaperCount}}</span></div>
            </div> 
          </div>
        </div>
      </div>
      <div class="lay-d" id="laydsecond">
        <p class="warnTitle">教学实时诊断预警</p>
        <br>
        <span class="waitToOver">
          待解决问题
          <p class="badge" style="color:white">
            {{warnCount}}
          </p>
        </span>
        <span id="ckeckAll" @click="toCheckAllWarn">查看全部>></span>
        <div id="biaoQianYe">
            <div v-for="(item,index) in warnList" :key="index" class="warnList">
                <div class="warnContent" style="font-size: 12px;font-family: PingFangSC-Regular, PingFang SC;font-weight: 400;color: #4D5753;" @click="toDetailsWarn(warnId[index])">{{item}}</div>
                <span class="time">{{warnTime[index]}}</span>
            </div>
            <p v-if="!warnCount" style="text-align:center;"><a-empty  :description="false"/>暂无预警</p>
        </div>
         
        <div id="echarts3" ref="echarts3">
          
        </div>
        <p class="circle">*&nbsp;每个圆点代表一个班</p>
      </div>
      <!--  练习趋势图和本学期做题时段数据 --> 
      
      <div>
        <a-spin :spinning="loading">
        <div class="lay-d" id="laydthird">
            <p class="termCharts"> <span class="echartTitle"> 练习情况趋势图 </span>
               <a-select class="fg" default-value="week" style="width: 90px;" @change="getDoExerciseCondTime">
                  <a-select-option value="week">
                    近一周
                  </a-select-option>
                  <a-select-option value="month">
                    近一月
                  </a-select-option>
                  <a-select-option value="xq">
                    本学期
                  </a-select-option>
                </a-select>
                <a-select class="fg" default-value="0" style="width: 90px;" @change="getExerciseGrade"> <!--  -->
                  <a-select-option value="0">
                    全校
                  </a-select-option>
                  <a-select-option value="1">
                    一年级
                  </a-select-option>
                  <a-select-option value="2">
                    二年级
                  </a-select-option>
                  <a-select-option value="3">
                    三年级
                  </a-select-option>
                  <a-select-option value="4">
                    四年级
                  </a-select-option>
                  <a-select-option value="5">
                   五年级
                  </a-select-option>
                </a-select>
            </p>
            <div ref="echarts4" id="echarts4">

            </div>
            <div id="ech4Title">
                <div class='tip' style="background-color:#FF8B7F;"></div>登录人数
                <div class='tip' style="background-color:#088767;"></div>做练习张数
                <div class='tip' style="background-color:#52B4FF;"></div>做练习人数
            </div>
        </div>
        <div class="lay-d" id="laydfour">
            <p class="termCharts" id="laydfourP"> <span class="echartTitle"> 本学期集中做题时段</span>
              <a-select class="fg" default-value="week" style="width: 90px" @change="getDoExerciseTime"><!--  -->
                  <a-select-option value="week">
                    近一周
                  </a-select-option>
                  <a-select-option value="month">
                    近一月
                  </a-select-option>
                  <a-select-option value="xq">
                    本学期
                  </a-select-option>
                </a-select>
                <a-select class="fg" default-value="0" style="width: 90px" @change="getDoExerciseGrade"><!--  -->
                  <a-select-option value="0">
                    全校
                  </a-select-option>
                  <a-select-option value="1">
                    一年级
                  </a-select-option>
                  <a-select-option value="2">
                    二年级
                  </a-select-option>
                  <a-select-option value="3">
                    三年级
                  </a-select-option>
                  <a-select-option value="4">
                    四年级
                  </a-select-option>
                  <a-select-option value="5">
                   五年级
                  </a-select-option>
                </a-select>
            </p>
            <div ref="echarts5" id="echarts5">
            </div>  
            <!-- <div style="width:390px;height:340px;position:relative;left:0;right:0;margin:0 auto;z-index:-10px;">
              <canvas ref="myCanvas" id="myCanvas" width="390" height="340" style="position:absolute;">
                Your browser does not support the HTML5 canvas tag.
              </canvas>
            </div> -->
        </div>
        </a-spin>
      </div>
      
    </div>
  </div>
</template>
<script>
import moment from 'moment'
import echarts from 'echarts'
import store from '../../store/index.js'
export default {
  data(){
    return{
      schoolOrArea:'',
      //用户id
      userId:'',
      //日期
      data:'',
      time: '',
      timer: '',
      week:'',
      resPrecentData:['first','second','third','four','five'],
       //登录和做题信息
      loginMessage:'',
      //百分比信息
      gradePrecentMessage:[],
      //正确率
      Accuracy:[],
      //完成率
      Finish:[],
      //正确率总数
      allAccuracy:[],
      //完成率总数
      allFinish:[],
      questionCount:19,
      //预警列表数据
      // warndata,
      warnList:'',
      //预警发送时间
      warnTime:'',
      // columns,
      //练习趋势时间和年级
      doExerciseCondGrade:0,
      doexerciseCondTime:'week',
      //做题时段班级和时间
      doTimeGrade:0,
      doTimeWhen:'week',
      //做题趋势加载图表
      loading:false,
      //预警数量
      warnCount:'',
      //预警唯一标识
      warnId:'',
    }
  },
  computed:{
        schoolOrAreas:function(){
            return this.$store.getters.getRoleType
        }
  },
  created(){
    this.userId = localStorage.getItem('UserId')
    const data = moment(new Date()).format('YYYY年MM月DD日')
    this.timer =window.setInterval(() => {
    this.time = moment(new Date()).format('a h:mm:ss')
    }, 1000)
    const week = moment(new Date()).format('dddd')
    this.data = data
    this.week = week
    //调用所有接口并初始化图表
    //登录信息
    this.GetSchoolLoginInfo()
    //获取页面头部百分比数据
    this.getGradePrecentMessage()
    //获取预警列表
    this.getWarnList()
    //获取预警地图数据
    this.getWarnMap()
    //获取做题趋势
    this.getExerciseCond()
    //获取做题时段
    this.getDoTime()
  },
  mounted(){
    this.init()
    // this.writecanvas()
  },
  computed:{
      RoleType:function(){
          var roleType = this.$store.getters.getRoleType
          return roleType
      }
  },
  methods:{
    //改变时间和年级时，重新渲染
    //选择趋势图时间
    getDoExerciseCondTime(value){
      this.doexerciseCondTime=value
      this.getExerciseCond()
    },
    //选择趋势图年级
    getExerciseGrade(value){
      this.doExerciseCondGrade=value
      this.getExerciseCond()
    },
    //选择做题时间
    getDoExerciseTime(value){
      this.doTimeWhen=value
      this.getDoTime()  
    },
    //选择做题时间段的年级
    getDoExerciseGrade(value){
      this.doTimeGrade=value
      this.getDoTime()
    },
    
    //选择获取正确率和完成率
    getGradePrecentMessage(){
      this.$http.post('/HomePageDataView/TeacherManage/GetTeacherManageTop',
        {
          userId:this.userId,
        }).then(res => {
          this.gradePrecentMessage=res.Data;
          for(let i=0;i<res.Data.length;i++){
            this.Accuracy.push(res.Data[i].Accuracy)
            this.Finish.push(res.Data[i].Finish)           
          }
          //echarts极坐标数据展示方向不同，需发展
          for(let i=res.Data.length-1;i>-1;i--){
            this.allAccuracy.push(res.Data[i].Accuracy)
            this.allFinish.push(res.Data[i].Finish)           
          }
          const myChart1 = echarts.init(this.$refs.echarts1)
          myChart1.setOption({
            series:[{
              data:this.allAccuracy
            }]
          })
          const myChart2 = echarts.init(this.$refs.echarts2)
          myChart2.setOption({
            series:[{
              data:this.allFinish
            }]
          })
          if (res.Data.length === 0) {
            this.$message.error('数据获取错误')
          }
      })
    },
    //获取学校登录人数和做题信息
    GetSchoolLoginInfo(){
      this.$http.post('/HomePageDataView/TeacherManage/GetSchoolLoginInfo',
          {
            userId:this.userId,
          }).then(res => {
            console.log(res);
            this.loginMessage=res.Data;
        })
    },
    //获取预警列表
    getWarnList(){
      this.$http.post('/HomePageDataView/TeacherManage/GetEarlyWarningPush',
        {
          userId:this.userId
        }).then(res => {
          var Data=res.Data
          console.log(Data);
          this.warnList=Data.map(item=>{
            return item.Content
          })
          this.warnTime=Data.map(item=>{
            return item.CreateTime
          })
          this.warnId=Data.map(item=>{
            return item.Id
          })
          this.warnCount=Data.length
        })
    },
    //跳转查看全部预警页面
    toCheckAllWarn(){
      this.$router.push('/SchoolManager/allWarn')
    },
    //跳转详情预警
    toDetailsWarn(value){
      console.log(value);
      this.$router.push({path:'/SchoolManager/warnDetail',query: {Id:value}})
    },
    //获取预警地图
    getWarnMap(){
      var arr=[];
      this.$http.post('/HomePageDataView/TeacherManage/GetClassFinishAndAccuracy',
          {
            userId:this.userId,
          }).then(res => {
            //处理预警地图数据
            var Data=res.Data
            var grade1
            var grade2
            var grade3
            var grade4
            var grade5
            // console.log(data);
           for(var i=0;i<Data.length;i++){
            if(Data[i].GradeId=="1"){
              grade1=Data[i].list
            }else if(Data[i].GradeId=="2"){
               grade2=Data[i].list
            }else if(Data[i].GradeId=="3"){
               grade3=Data[i].list
            }else if(Data[i].GradeId=="4"){
               grade4=Data[i].list
            }else if(Data[i].GradeId=="5"){
               grade5=Data[i].list
            }
           }
            var warn1 = grade1.map(item=>{
              return [item.Finish,item.Accuracy,item.ClassName]
            })
            var warn2 = grade2.map(item=>{
              return [item.Finish,item.Accuracy,item.ClassName]
            })
            var warn3 = grade3.map(item=>{
              return [item.Finish,item.Accuracy,item.ClassName]
            })
            var warn4 = grade4.map(item=>{
              return [item.Finish,item.Accuracy,item.ClassName]
            })
            var warn5 = grade5.map(item=>{
              return [item.Finish,item.Accuracy,item.ClassName]
            })
            
            const myChart3 = echarts.init(this.$refs.echarts3)
            myChart3.setOption({
              series: [{data: warn1 },
                      { data: warn2 },
                      { data: warn3 },
                      { data: warn4 },
                      { data: warn5 }
                    ]
            })
        })
    },
    //练习情况趋势图
    getExerciseCond(){
      this.loading=true
      var timeArr=[]
      this.$http.post('/HomePageDataView/TeacherManage/GetExercises',
          {
            userId:this.userId,
            whereTime:this.doexerciseCondTime,
            grade:this.doExerciseCondGrade
          }).then(res=>{
            var data=res.Data
            console.log(data);
            const LoginCount=data.map(item=>{
              return item.LoginCount
            })
            const DoPaperCount=data.map(item=>{
              return item.DoPaperCount
            })
            const DoPeopleCount=data.map(item=>{
              return item.DoPeopleCount
            })
            const timearr=data.map(item=>{
              return item.Time
            })
            this.loading=false
            const myChart4 = echarts.init(this.$refs.echarts4)
            myChart4.setOption({
            xAxis:[{
              data:timearr
            }],
            series: [{data:LoginCount},
            {data:DoPaperCount},
            {data:DoPeopleCount}
            ]
        })
      })
    },
    //做题时段
    getDoTime(){
      this.$http.post('/HomePageDataView/TeacherManage/GetTimeIntervalBySchool',
          {
            userId:this.userId,
            whereTime:this.doTimeWhen,
            grade:this.doTimeGrade
          }).then(res=>{
            var Data = res.Data
            const doDtime=Data.map(item=>{
              return item.StuCount
            })
            const myChart5 = echarts.init(this.$refs.echarts5)
            myChart5.setOption({
            series: [
              {data:doDtime}
            ]
            })
            
      })
    },
    //canvas
    writecanvas(){
      var c = this.$refs.myCanvas
      var ctx = c.getContext('2d')
      ctx.beginPath()
      ctx.lineWidth = 2
      ctx.strokeStyle = '#eee'
      ctx.arc(204, 159, 135, 0, 2 * Math.PI)
      ctx.stroke()

      // 时刻度
      ctx.save()
      ctx.translate(204, 159)

      var hour = [6, '', 8, '', 10, '', 12, '', 14, '', 16, '', 18, '', 20, '', 22, '', 0, '', 2, '', 4, '']
      for (var i = 0; i < 24; i++) {
        const v = Math.PI / 12
        ctx.rotate(v)
        ctx.beginPath()
        ctx.moveTo(0, -50)
        ctx.lineTo(0, -135)
        ctx.closePath()
        ctx.lineWidth = 0.5
        ctx.setLineDash([3, 3])
        ctx.strokeStyle = '#ccc'
        ctx.stroke()
      }
      ctx.restore()

      hour.forEach(function (num, i) {
        var rad = 2 * Math.PI / 24 * i
        var x = Math.cos(rad) * 150
        var y = Math.sin(rad) * 150
        ctx.font = '12px sans-serif'
        ctx.textAlign = 'center'
        ctx.textBaseline = 'middle'
        ctx.fillStyle = '#000'
        ctx.fill()
        ctx.fillText(num, x + 204, y + 159)
      })
    },

    init(){
      const myChart1 = echarts.init(this.$refs.echarts1)
      myChart1.setOption({
        angleAxis: {
          //百分比总数
          max:100,
          startAngle: -90,
          axisLine: {
            show: false
          },
          axisTick:{
              show:false
          },
          axisLabel:{
              show:false
          },
          splitLine: {
              show: false
          }
        },
        radiusAxis: {
          data:this.allAccuracy,
          axisLabel:{
              show:false
          },
          axisTick:{  
              show:false
          },
          axisLine: {
              show: false
          },
        },
        //设置内外半径
        polar: {
          radius: ["12%", "100%"]
        },      
        series: [{
            type: 'bar',
            data:[0,0,0,0,0],
            showBackground: true,
            coordinateSystem: 'polar',
            roundCap: true,
            itemStyle: {
            normal: {
              color: function(params) {
                  var colorList = ['#F57579','#79BEBD', '#448870', '#FACC14', '#5CC0FF'];
                  return colorList[params.dataIndex]
                  }
                }
              },
              barWidth: '50%',	
              barCategoryGap: '50%'
            }]
          })
    const myChart2 = echarts.init(this.$refs.echarts2)
    myChart2.setOption({
        angleAxis: {
        max:100,
        startAngle: -90,
        axisLine: {
         show: false
        },
        axisTick:{
            show:false
        },
        axisLabel:{
            show:false
        },
        splitLine: {
            show: false
        }
        },
        radiusAxis: {
            data: [5,4,3,2,1],
            axisLabel:{
            show:false
          },
          axisTick:{
              show:false
          },
          axisLine: {
              show: false
          },
        },
        //设置内外半径
        polar: {
          radius: ["12%", "100%"]
        },
        xAxis: {
            show: false,
            gridIndex:100,
        },
        series: [{
            type: 'bar',
            showBackground: true,
            data: [0,0,0,0,0],
            coordinateSystem: 'polar',
            roundCap: true,
            itemStyle: {
            normal: {
                color: function(params) {
                    var colorList = ['#F57579','#79BEBD', '#448870', '#FACC14', '#5CC0FF'];
                    return colorList[params.dataIndex]
                      }
                  }
              },
              barWidth: '50%',	
              barCategoryGap: '50%'
        }]
      })
      //散点图
      const myChart3 = echarts.init(this.$refs.echarts3)
      myChart3.setOption({
        title: {
          text: '多色散点图',
          textStyle: {
            fontSize: 16
          },
        },
        grid: {
            left: '3%',
            right: '7%',
            bottom: '3%',
            containLabel: true
        },
        tooltip: {
            // trigger: 'axis',
            showDelay: 0,
            formatter: function (params) {
                    return params.value[2]+' :<br/>'
                    + '完成率：'+ params.value[0] +'%<br/>'
                    + '正确率：'+ params.value[1] +'%';
            },
            axisPointer: {
                show: true,
                type: 'cross',
                lineStyle: {
                    type: 'solid',
                    width: 1
                }
            }
        },
        legend: {
            data: ['一年级', '二年级', '三年级', '四年级', '五年级'],
            right: 'center',
            itemWidth: 11,
            itemHeight: 11,
             width: "100%",
        },
        xAxis: [{
                type: 'value',
                max:100,
                scale:false,
                name: '(完成率)%',
                nameLocation: 'end',
                nameGap: 20,
                axisLabel: {
                    formatter: '{value}'
                },
                splitLine: {
                    show: true
                },
                nameTextStyle: {
                align: "center",
                rich: {
                  "<style_name>": {
                    align: "center"
                  }
                }
              }
            }],
        yAxis: [
            {
                type: 'value',
                max:100,
                scale:true,
                name: '（正确率）%',
                nameLocation: 'end',
                axisLabel: {
                    formatter: '{value}'
                },
                splitLine: {
                    show: true,
                    lineStyle:{
                        type:'dashed'
                    }
                }
            }
        ],
        color:['#5CC0FF','#FACC14','#448870','#79BEBD','#F57579'],
        series: [
            {
                name: "一年级",
                type: 'scatter',
                // data: [
                //         [61.2, 51.6,"一年级"], [67.5, 59.0,"一年级"], [59.5, 49.2,"一年级"], [57.0, 63.0,"一年级"], [55.8, 53.6,"一年级"],
                //         [0, 0], [0,0], [0, 69.8], [0, 66.8], [0, 75.2]
                // ],
            },
            {
                name: '二年级',
                type: 'scatter',
                // data: [[14.0, 65.6], [75.3, 71.8], [93.5, 80.7], [86.5, 72.6], [87.2, 78.8],
                //     [81.5, 74.8], [84.0, 86.4], [184.5, 78.4], [75.0, 62.0], [84.0, 81.6]
                // ],
            },
            {
                name: '三年级',
                type: 'scatter'
            },
            {
                name: '四年级',
                type: 'scatter'
            },
            {
                name: '五年级',
                type: 'scatter'
            },
        ]
      })
      
      const myChart4 = echarts.init(this.$refs.echarts4)
      myChart4.setOption({
        title: {
          text: '(次数/人数)', // 正确率
          textStyle: {
            fontSize: 10.32
          },
        },
        // legend: {
        //   show: true,
        //   bottom: "-0%"
        // },
        dataZoom: [
            {
                start: 0, // 默认为0
                end: 100 - 1500 / 50, // 默认为100
                type: 'slider',
                show: true,
                xAxisIndex: [0],
                handleSize: 0, // 滑动条的 左右2个滑动条的大小
                height: 14, // 组件高度
                bottom: 0, // 底部的距离
                border: 'none',
                borderColor: '#fff',
                fillerColor: '#B5BEB9',
                borderRadius: 7,
                backgroundColor: '#F8F8F8', // 两边未选中的滑动条区域的颜色
                showDataShadow: false, // 是否显示数据阴影 默认auto
                showDetail: false, // 即拖拽时候是否显示详细数值信息 默认true
                realtime: true, // 是否实时更新
                filterMode: 'filter'
            }
        ],
        // 提示框内容
        tooltip: {
          trigger: 'axis'
        },
        // 自定义颜色
        color: ['#FF8B7F', '#71B6A1', '#52B4FF'],
        grid: {
          left: '3%',
          right: '4%',
          bottom: '10%',
          containLabel: true
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: ['']
        },
        yAxis: {
          type: 'value',
          axisLabel: {
            formatter: '{value}'
          },
          triggerEvent: true,
        },
        series: [
          {
            name: '登陆人数',
            type: 'line',
            // stack: '总量',
            // data: [65, 78, 82, 75, 69, 73, 98, 89, 73, 98, 73, 98, 73, 78, 82, 75],
            itemStyle: {
              normal: {
                lineStyle: {
                  width: 1, // 设置线宽
                  type: 'solid' // 'dotted'虚线 'solid'实线
                }
              }
            },
            // 背景阴影渐变
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: 'rgba(250, 69, 56, 0.5)'
              }, {
                offset: 1,
                color: 'rgba(255, 255, 255, 1)'
              }])
            }
          },
          {
            name: '做练习张数',
            type: 'line',
            // stack: '总量',
            // data: [73, 98, 89, 73, 98, 73, 70, 85, 89, 73, 98, 73, 98, 73, 98, 73, 98],
            itemStyle: {
              normal: {
                lineStyle: {
                  width: 1, // 设置线宽
                  type: 'solid' // 'dotted'虚线 'solid'实线
                }
              }
            },
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: 'rgba(113, 182, 161, 1)'
              }, {
                offset: 1,
                color: 'rgba(255, 255, 255, 1)'
              }])
            }
          },
          {
            name: '做练习人数',
            type: 'line',
            // stack: '总量',
            // data: [15, 68, 12, 33, 68, 43, 98, 73, 19, 33, 68, 73, 28, 73, 98, 63, 98],
            itemStyle: {
              normal: {
                lineStyle: {
                  width: 1, // 设置线宽
                  type: 'dotted' // 'dotted'虚线 'solid'实线
                }
              }
            }
          }
        ]
      })
      const myChart5 = echarts.init(this.$refs.echarts5)
      myChart5.setOption({
         tooltip: {
              trigger: 'axis',
             axisPointer: {
              lineStyle: {
                color: "#a5c244",
                width: 2
              }
            }
          },
          angleAxis: {
            type: "category",
            data: ["0","2", "4", "6", "8", "10", "12", "14", "16", "18", "20", "22"],
            boundaryGap: false,
            axisLabel: {
              rich: {
                "<style_name>": {
                  align: "center"
                }
              },
              show: true
            },
            axisLine: {
                show: false
              },
              axisTick: {
                lineStyle: {
                  width: 0.5
                },
                length: 3
              }
          },
          radiusAxis: {
            axisLine: {
              show: false
            },
            axisTick: {
              show: false
            },
            axisLabel: {
              show: false
            }
          },
          polar: {
          },
          series: [{
              type: 'bar',
              data: [0], 
              coordinateSystem: 'polar',
              stack: 'a',
              itemStyle: { color: '#43715A' }
          }],
          legend: {
          },
      })
      
      window.addEventListener('resize', function () {
        myChart1.resize()
        myChart2.resize()
        myChart3.resize()
        myChart4.resize()
        myChart5.resize()
      })
      
    },
  },
}
</script>
<style lang="less" scoped>
body,html{
  font-size: 12px;
  margin: 0px;
  padding: 0px;
}
#maxoutd{
  margin-top: -25px;
  background-color:#f1f2f2;
  position: relative;
  width:103%;
  height:1530px;
  #time-d{
    position: relative;
    height: 30px;
    font-size: 12px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #A8B3B6;
  }
}
//所有布局div样式
.lay-d{
  border-radius: 10px;
  padding-top: 1%;
  padding-left: 1%;
  background-color: white;
  width: 97%;
}

#laydsecond :nth-child(1){
  position: relative;
  font-size: 18px;
  font-family: PingFangSC-Regular, PingFang SC;
  font-weight: 400;
  color: #4D5753;
}
#laydfirset{
  top: 2%;
  height: 265px;
  position: absolute;
  margin: 0px;
  overflow: hidden;
  }
#laydsecond{
  position: relative;
  margin-top: 280px;
  height: 800px;
}
#laydthird{
  display: inline-block;
  margin-top: 20px;
  position: relative;
  width: 60%;
  height: 450px;
}
#laydfour{
  padding: 0px;
  margin-left:2%;
  display: inline-block;
  width: 35%;
  height: 450px;
  margin-top: 20px;
  position: absolute;
}
.gradeColor{
  display: inline-block;
  width: 11px;
  height: 11px;
  border-radius: 15px;
}
#showEcharts{
  position: absolute;
  height: 80%;
  width: 30%;
  display: inline-block;
}
#echarts1{
  position: absolute;
  width: 30%;
  height: 80%;
}
#echarts2{
  margin-left: 68%;
  position: absolute;
  width: 30%;
  height: 80%;
}
#echarts3{
  width: 70%;
  height:78%;
  margin-left: 30%;
  display: inline-block;
}
//练习情况趋势图echarts div
#echarts4{
  position: relative;
  width: 100%;
  height: 80%;
}
#echarts5{
  width: 100%;
  height: 80%;
  position: absolute;
}
//所有百分比
.Precent{
  width: 10%;
  position: absolute;
  height: 80%;
}
.showPrecent{
  font-size: 12px;
  font-family: PingFangSC-Semibold, PingFang SC;
  font-weight: 600;
  width: 100%;
  height: 20%;
}
#turePrecent{
  position: absolute;
  margin-left: 32%;
}
#finshPrecent{
  position: absolute;
  margin-left: 100%;
}
.allMessageShow{
  position: absolute;
  display: inline-block;
  width:60%;
  height:15%;
  margin-top: 50px;
  margin-left: 38%;
  font-size: 12px;
  font-family: PingFangSC-Regular, PingFang SC;
  font-weight: 400;
  text-align: center;
}
.message-div{
  white-space: nowrap;
  position: relative;
  display: inline-block;
  height: 100%;
  // background: brown;
  margin-left:13px;
  margin-top: 0px;
  font-size: 12px;
  font-family: PingFangSC-Regular, PingFang SC;
  font-weight: 400;
  color: #4D5753;
}
#biaoQianYe{
  position: absolute;
  // border: 1px solid;
  width: 35%;
  height: 80%;
  
  text-overflow: ellipsis;
  overflow: hidden;
}
#ckeckAll{
  position: relative;
  cursor: pointer;
  margin-left: 13%;
  font-size: 14px;
  font-family: PingFangSC-Regular, PingFang SC;
  font-weight: 400;
  color: #B5B5B5;
}
.item{
  width: 100%;
  height: 100%;
  border-radius: 8px;
  background-color:#F6F6F6;
  // border: 1px solid black;
}
/deep/ .a-tabs{
  background-color: green;
}
.termCharts{
  width: 90%;
  height: 30px;
  font-size: 14px;
  font-family: PingFangSC-Regular, PingFang SC;
  font-weight: 400;
  color: #4D5753;
}
#laydfourP{
  margin-left: 2%;
  margin-top: 2%;
}
//登录人数 span
.messageNum{
  font-size: 18px;
  font-family: Roboto-Regular, Roboto;
  font-weight: 400;
  color: #3CBB8F;
}
//练习情况趋势图提醒字段
.tip{
  text-decoration:underline;
  width: 20px;
  height: 2px;
  display: inline-block;
  margin-bottom:3px;
}
.tip:nth-of-type(1){
  margin-left: 25%;
}
.tip:nth-of-type(2){
  margin-left: 12%;
}
.tip:nth-of-type(3){
  margin-left: 6%;
}
  #myCanvas{
    left: -10px;
    top: -3px;
  }
/deep/ .fg div{
  border:none;
}
.warnList{
  position: relative;
  width: 100%;
  margin: 20px 0px;
  border-radius: 5px;
  height: 70px;
  margin: 10px auto;
  padding-left: 20px;
  padding-top:10px;
  display: block;
  background-color:#f6f6f6;
  border-radius: 10px;
  max-width: 430px;
  margin-left: 0px;
}
.warnCont{
    margin-left: 24px;
    height:80%;
    display:inline-block;
    font-size: 12px;
  }
.warnContent{
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  position: relative;
  height: 30px;
  cursor: pointer;
}
.warnContent:hover{
  color: red;
  text-decoration:underline;
}
.time{
  position: absolute;
  font-size: 12px;
  font-family: PingFangSC-Regular, PingFang SC;
  font-weight: 400;
  color: #B5B5B5;
  }
  .ant-select-selection--single.ant-select-selection{
    background-color: #ffffff;
    border: none;
    width: 100px;
  }
  
  .accuracy{
    display:inline;
    margin-top:120px;
    color: #989898;
    width: 40px;
  }
  .finish{
    display:inline;
    margin-left:55%;
    margin-top:120px;
    color: #989898;
     width: 40px;
  }
  .mess-top{
    display: inline-block;
    margin: 0px 5%;
  }
  .mess-top:nth-of-type(2){
    margin-left: 4%;

  }
  .circle{
    font-size: 12px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #b5b5b5;
    position: absolute;
    left:68%;
    bottom: 5%;
  }
  .warnTitle{
  font-size: 18px;
  font-family: PingFangSC-Regular, PingFang SC;
  font-weight: 400;
  color: #4D5753;
  }
.echartTitle{
  font-size: 14px;
  font-family: PingFangSC-Regular, PingFang SC;
  font-weight: 400;
  color: #4D5753;
}
.waitToOver{
  position: relative;
  display: inline-block;
  font-size: 14px;
  font-family: PingFangSC-Medium, PingFang SC;
  font-weight: 500;
  color: #4D5753;
}
.badge{
  text-align: center;
  display: inline-block;
  width: 30px;
  height: 25px;
  line-height: 25px;
  border-radius: 20px;
  position: absolute;
  top: -10px;
  right: 0px;
  background-color: red;
}
#ech4Title{
  margin-bottom: 0px;
}
  </style>