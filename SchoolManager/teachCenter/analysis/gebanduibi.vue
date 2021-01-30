<template>
    <div>
        <div id="paperTitles">
            <br>
            {{titles}}
            <br>
            <br>
        </div>
        
        <div id="eachrts" ref="mychart1">
            
        </div>
        <a-divider />
        <div id="classDetail">
            {{classTitile}}
        </div>
        <div id="eachrts2" ref="mychart2">
            
        </div>
        <br><br>
    </div>
</template>
<script>
import echarts from 'echarts'
export default {
    data(){
        return{
            paid:'',
            //本校正确率
            schoolAccuracy:[],
            //全区正确率
            areaAccuracy:[],
            //点击时获取的班级
            classId:'',
            classTitile:'',
            className:'',
            userid:'',
            underEcShow:false,
        }
    },
    props:{
        paids:String,
        titles:String,
    },
    computed:{
        RoleType:function(){
            return this.$store.getters.getRoleType
        }
    },
    created(){
        this.userid = localStorage.getItem('UserId')
        this.getDetail()
    },
    mounted(){
        this.init()
    },
    watch:{
        paids:function(newval,oldval){
            this.paids=newval
            echarts.init(this.$refs.mychart1).dispose()
                this.init()
                this.getDetail()
        },
        titles:function(newval,oldval){
            this.titles=newval
            console.log(this.titles);
        },
    },
    methods:{
        getDetail(){
            this.$uwonhttp.post('/Paper/TeacherPaper/GetMyClassScoreInfo', {
                PaperId:this.paids,
                UserId:this.userid
            }).then(res=>{
                var Data=res.data.Data[0]
                console.log(Data);
                for(let i=0;i<Data.MyClass.length+1;i++){
                    this.schoolAccuracy.push(Data.SchoolAccuracy)
                    this.areaAccuracy.push(Data.AvgScore)
                }
                console.log(Data.MyClass.length);
                console.log(this.schoolAccuracy,this.areaAccuracy);
                const classFinsh=Data.MyClass.map(item=>{
                    return item.FinishPer
                })
                const classAccuracy=Data.MyClass.map(item=>{
                    return item.AvgScore
                })
                const classId=Data.MyClass.map(item=>{
                    return item.ClassNo
                })
                this.classId=classId
                const classname=Data.MyClass.map(item=>{
                    return item.ClassName
                })
                this.className=classname
                
                console.log(this.RoleType);
                this.$nextTick(()=>{
                    if(this.RoleType==0){
                        console.log(this.schoolAccuracy,this.areaAccuracy);
                        const myChart = echarts.init(this.$refs.mychart1)
                        myChart.setOption({
                        color: ['#b0cb5c', '#7ec5a6','#f4c55f','#f87679'],
                        title: {
                            subtext: '(单位：%)',
                            left:"5%"
                        },
                        tooltip: {
                            trigger: 'axis',
                        },
                        legend: {
                            data: ['完成率', '正确率','本校正确率','全区正确率']
                        },
                        xAxis: [
                            {
                                type: 'category'
                            }
                        ],
                        yAxis: [{
                                type: 'value',
                            }],
                            series:[
                                {type: 'bar',data:classFinsh},
                                {type: 'bar',data:classAccuracy},
                                {name:'本校正确率',type: 'line',data:this.schoolAccuracy},
                                {name:'全区正确率',type: 'line',data:this.areaAccuracy}
                            ],
                            xAxis: [
                                {
                                    type: 'category',
                                    data: this.className
                                }
                            ],
                        })
                    }else{
                        console.log(classFinsh,classAccuracy);
                        const myChart = echarts.init(this.$refs.mychart1)
                        myChart.setOption({
                            series:[
                                {type: 'bar',data:classFinsh},
                                {type: 'bar',data:classAccuracy},
                                
                            ],
                            xAxis: [
                                {
                                    type: 'category',
                                    data: this.className
                                }
                            ],
                        })
                    }
                })
                    
                
            })
        },
        init(){
            if(this.RoleType==0){
                const myChart1 = echarts.init(this.$refs.mychart1)
                myChart1.setOption({
                color: ['#b0cb5c', '#7ec5a6','#f4c55f','#f87679'],
                title: {
                    subtext: '(单位：%)',
                    left:"5%"
                },
                tooltip: {
                    trigger: 'axis',
                },
                legend: {
                    data: ['完成率', '正确率','本校正确率','全区正确率']
                },
                xAxis: [
                    {
                        type: 'category'
                    }
                ],
                yAxis: [{
                        type: 'value',
                    }],
                series: [
                    {
                        name: '完成率',
                        type: 'bar',
                        label: {
                        normal: {
                            position: 'top',
                            show: true
                            }
                        },
                        barWidth: 45,
                        barGap: "0%",
                    },
                    {
                        name: '正确率',
                        type: 'bar',
                        label: {
                        normal: {
                            position: 'top',
                            show: true
                            }
                        },
                        barWidth: 45,
                        barGap: "0%",
                    },{
                        name: '本校正确率',
                        type: 'line',
                        data:[0]
                    },{
                        name: '全区正确率',
                        type: 'line',
                        data:[0]
                    },
                ]
                })
                const myChart2 = echarts.init(this.$refs.mychart2)
                myChart2.setOption({
                title: {
                    subtext: '(单位：人)',
                    left:"5%"
                },
                // color: [ '#7ec5a6','#28b7ff','#A2C240','#ffc23d','#ff9166'],
                // color: ['#B0CB5C', '#91CDB3', '#28B7FF', '#B0CB5C', '#FFC23D', '#FF9166'],
                
                calculable: true,
                xAxis: [
                    {
                        type: 'category',
                        data: ['100', '90-99', '80-89', '60-79', '0-59','未做']
                    }
                ],
                yAxis: [{
                        type: 'value',
                    }],
                series:[{
                        name: '完成率',
                        type: 'bar',
                        label: {
                            normal: {
                                position: 'top',
                                show: true
                            }
                        },
                        itemStyle: {
                            normal: {
                            label: { // ---图形上的文本标签
                                formatter: function (params) {
                                // console.log(params)
                                // console.log(params.value)
                                if (params.name === '100') {
                                    return params.value + '人优秀' + '{img1|}'
                                } else if (params.name === '90-99') {
                                    return `${params.value}人优秀`
                                } else if (params.name === '80-89') {
                                    return `${params.value}人良好`
                                } else if (params.name === '60-79') {
                                    return `${params.value}人合格`
                                } else if (params.name === '0-59') {
                                    return `${params.value}人须努力`
                                } else {
                                    return `${params.value}人未做`
                                }
                                },
                                rich: {
                                    img1: {
                                        backgroundColor: {
                                            image: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1605074303494&di=3871f451c5a232233ddd64cb07969063&imgtype=0&src=http%3A%2F%2Fbpic.588ku.com%2Felement_origin_min_pic%2F18%2F02%2F24%2Fee79cb33232a6f2bcce1512cfa251993.jpg',
                                        },
                                        height: 20,
                                        borderRadius: 5
                                    }
                                },
                                show: true,
                                position: 'top', // ---相对位置
                                // rotate: 0, // ---旋转角度
                                color: '#000'
                            },
                            color: function (params) {
                                // 注意，如果颜色太少的话，后面颜色不会自动循环，最好多定义几个颜色
                                var colorList = ['#B0CB5C', '#91CDB3', '#28B7FF', '#B0CB5C', '#FFC23D', '#FF9166']
                                return colorList[params.dataIndex]
                            }
                            }
                        },
                        barWidth: '45',				// ---柱形宽度
                        barCategoryGap: '30%'
                    }]
                })
                var that = this
                myChart1.on('click', function (params) {
                    that.classTitile=params.name+'详细情况'
                    var classNo=that.classId[params.dataIndex]
                    console.log(params)
                    that.getClassDetail(classNo)
                })
                
                window.addEventListener('resize',function(){
                    myChart1.resize()
                    myChart2.resize()
                })
            }else{
                
                const myChart1 = echarts.init(this.$refs.mychart1)
                myChart1.setOption({
                color: ['#b0cb5c', '#7ec5a6'],
                title: {
                    subtext: '(单位：%)',
                    left:"5%"
                },
                tooltip: {
                    trigger: 'axis',
                    axisPointer: {            // 坐标轴指示器，坐标轴触发有效
                        type: 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
                    }
                },
                legend: {
                    data: ['完成率', '正确率']
                },
                
                calculable: true,
                xAxis: [
                    {
                        type: 'category'
                    }
                ],
                yAxis: [{
                        type: 'value',
                    }],
                series: [
                    {
                        name: '完成率',
                        type: 'bar',
                        label: {
                        normal: {
                            position: 'top',
                            show: true
                            }
                        },
                        barWidth: 45,
                        barGap: "0%",
                    },
                    {
                        name: '正确率',
                        type: 'bar',
                        label: {
                        normal: {
                            position: 'top',
                            show: true
                            }
                        },
                        barWidth: 45,
                        barGap: "0%",
                    }
                ]
                })
                const myChart2 = echarts.init(this.$refs.mychart2)
                myChart2.setOption({
                title: {
                    subtext: '(单位：人)',
                    left:"5%"
                },
                // color: [ '#7ec5a6','#28b7ff','#A2C240','#ffc23d','#ff9166'],
                // color: ['#B0CB5C', '#91CDB3', '#28B7FF', '#B0CB5C', '#FFC23D', '#FF9166'],
                
                calculable: true,
                xAxis: [
                    {
                        type: 'category',
                        data: ['100', '90-99', '80-89', '60-79', '0-59','未做']
                    }
                ],
                yAxis: [{
                        type: 'value',
                    }],
                series:[{
                        name: '完成率',
                        type: 'bar',
                        label: {
                            normal: {
                                position: 'top',
                                show: true
                            }
                        },
                        itemStyle: {
                            normal: {
                            label: { // ---图形上的文本标签
                                formatter: function (params) {
                                // console.log(params)
                                // console.log(params.value)
                                if (params.name === '100') {
                                    return params.value + '人优秀' + '{img1|}'
                                } else if (params.name === '90-99') {
                                    return `${params.value}人优秀`
                                } else if (params.name === '80-89') {
                                    return `${params.value}人良好`
                                } else if (params.name === '60-79') {
                                    return `${params.value}人合格`
                                } else if (params.name === '0-59') {
                                    return `${params.value}人须努力`
                                } else {
                                    return `${params.value}人未做`
                                }
                                },
                                rich: {
                                    img1: {
                                        backgroundColor: {
                                            image: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1605074303494&di=3871f451c5a232233ddd64cb07969063&imgtype=0&src=http%3A%2F%2Fbpic.588ku.com%2Felement_origin_min_pic%2F18%2F02%2F24%2Fee79cb33232a6f2bcce1512cfa251993.jpg',
                                        },
                                        height: 20,
                                        borderRadius: 5
                                    }
                                },
                                show: true,
                                position: 'top', // ---相对位置
                                // rotate: 0, // ---旋转角度
                                color: '#000'
                            },
                            color: function (params) {
                                // 注意，如果颜色太少的话，后面颜色不会自动循环，最好多定义几个颜色
                                var colorList = ['#B0CB5C', '#91CDB3', '#28B7FF', '#B0CB5C', '#FFC23D', '#FF9166']
                                return colorList[params.dataIndex]
                            }
                            }
                        },
                        barWidth: '45',				// ---柱形宽度
                        barCategoryGap: '30%'
                    }]
                })
                var that = this
                myChart1.on('click', function (params) {
                    that.classTitile=params.name+'详细情况'
                    var classNo=that.classId[params.dataIndex]
                    console.log(params)
                    that.getClassDetail(classNo)
                })
                
                window.addEventListener('resize',function(){
                    myChart1.resize()
                    myChart2.resize()
                })
            }
        },
        //获取全校和全区正确率对比
        getClassDetail(classid){
            this.$uwonhttp.post('/Paper/TeacherPaper/GetPaperStudentScoreForChartWeb', {
                PaperId:this.paids,
                ClassId:classid,
            }).then(res=>{
                var Data=res.data.Data
                console.log(Data);
                const myChart2 = echarts.init(this.$refs.mychart2)
                myChart2.setOption({
                    series:[
                        {data:Data}
                    ]
                })
            })
        },
    },
}
</script>
<style lang="less">
#paperTitles{
    width: 100%;
    text-align: center;
    background-color: white;
    font-size: 18px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #4D5753;
}
#eachrts{
    width: 100%;
    height: 500px;
    position: relative;
    top: 0px;
    bottom: 0px;
    left: 0px;
    right: 0px;
    margin:auto;
    background-color: white;
}
#eachrts2{
    width: 100%;
    height: 500px;
    position: relative;
    top: 0px;
    bottom: 0px;
    left: 0px;
    right: 0px;
    margin:auto;
    background-color: white;
}
#classDetail{
    width: 100%;
    height: 50px;
    position: relative;
    background-color: white;
    text-align: center;
    line-height: 50px;
}
</style>