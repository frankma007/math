<template>
    <div>
        
        <div id="title">
            <br>
            {{titles}}
            <br>
            <br>
        </div>
        <a-spin :spinning="load">
        <div id="eachrts" ref="mycharts1">
            
        </div>
        </a-spin>
        <br>
        <div class="bottom">
            <p class="chapterTitle">{{titles}}</p>
            <br>
            <div class="showProcress">
                <div class="process"  v-for="(item,index) in allDatas" :key="index">
                    <div class="proce">
                        <a-progress type="circle" :percent="item.SchoolPer" :width="50" :format="(percent) =>{if(percent=='100'){return '100%'}else{return percent+'%'}}"/>
                    </div>
                    <div class="number">
                        {{index+1}}
                    </div>
                </div>
            </div>
            <br>
            <br>
            <div class="showQuestion">
                <div class="type" v-for="(item,index) in allDatas" :key="index">
                    <div class="question">
                        {{index+1+'.'}}
                        <div class="anserTitle" v-html="item.OriItemTitle"></div>
                        <span class="options" v-for="(ite,ind) in item.Opts" :key="ind" v-show="item.ItemTypeId !== 5">
                            <br>
                            <!-- 选项内容 -->
                            <span v-if="item.ItemType === 2 || item.ItemType === 10">{{ ite.Opt }}.</span>
                            <span v-html="ite.OptContent" style="margin-right: 25px"></span>
                        </span>
                    </div>

                    <div class="answer">
                        <p class="anserTitle"><span>【答案:】 </span><span v-html="item.Answer"></span></p>
                        <br>
                        <p class="anserTitle"><span>【解析:】 </span><span v-html="item.Analyze"></span></p>
                        <br>
                        <p class="AccuracyPrecent"> 本校正确率：<span class="Accuracy" v-html="item.SchoolPer+'%'"></span>&nbsp;&nbsp;&nbsp;&nbsp;本区正确率：<span class="Accuracy" v-html="item.AllPer+'%'"></span> </p>
                        <br>
                        <p class="anserTitle" >
                            正确<span v-html="item.SchoolRight"></span>人 
                            <span class="progressSpan"><a-progress :success-percent="item.SchoolRight/(item.SchoolRight+item.SchoolError)*100" :showInfo="false" :strokeWidth="15" /></span>  
                            错误<span v-html="item.SchoolError"></span>人&nbsp;&nbsp;&nbsp;&nbsp;
                            <span v-if="item.SchoolRight !== 0 || item.SchoolError !== 0" class="packInfo" @click="questionDetail(item.ItemID,index,$event)">展开详情↓</span>&nbsp;
                        </p>
                        <div class="answerDetail" ref="answerDetail">
                            <div class="allAnswer" v-for="(m,ind) in allAnswerDetail[index]" :key="ind">
                                <span class="detailOpt ansers" v-html="m.Answer"></span>&nbsp;&nbsp;
                                <!-- <span class="detailOptProgress" v-if="m.IsRight==1"><a-progress :success-percent="m.Count/countNum[index]*100" :strokeWidth="15" :showInfo="false"/></span>
                                <span class="detailOptProgress" v-if="m.IsRight==0"><a-progress :percent="m.Count/countNum[index]*100" :strokeWidth="15" :showInfo="false" status="exception"/></span>&nbsp;&nbsp; -->
                                <span class="progress" v-if="m.IsRight==1"><div :style="{width:m.Count/countNum[index]*100+'%',height:'100%',borderRadius:'100px',backgroundColor:'#1890ff'}"></div> </span>
                                <span class="progress" v-if="m.IsRight==0"><div :style="{width:m.Count/countNum[index]*100+'%',height:'100%',borderRadius:'100px',backgroundColor:'#ff682c'}"></div> </span>
                                <span class="detailOpt" v-html="m.Count"></span>人
                            </div>
                        </div>
                    </div>
                    <br>
                    <br>
                </div>
            </div>
        </div>
        <br>
        <br>
    </div>
</template>
<script>
import echarts from 'echarts'
export default {
    data(){
        return{
            load:false,
            allDatas:'',
            isShowAnswer:false,
            userid:'',
            allAnswerDetail:[],
            countNum:[],
        }
    },
    props:{
        paids:String,
        titles:String,
    },
    watch:{
        paids:function(newval,oldval){
            this.paids=newval
            this.showAccuracy()
            console.log(this.paids);
        },
        titles:function(newval,oldval){
            this.titles=newval
            console.log(this.titles);
        },
    },
    created(){
        this.userid = localStorage.getItem('UserId')
        this.showAccuracy()
    },
    mounted(){
        this.init()
    },
    methods:{
        questionDetail(qid,index,e){
            e.target.innerText=="展开详情↓"?e.target.innerText="收起详情":e.target.innerText="展开详情↓"
            console.log(e.target.style.display);
            // var styles=document.getElementsByClassName('answerDetail')[index].style
            var styles=this.$refs.answerDetail[index].style
            const text=document.getElementsByClassName('packInfo')[index]
            console.log(this.allAnswerDetail);
            if(styles.display=="none"||styles.display==""){
                styles.display="inline-block"
                this.$http.post('/report/TeacherReport/GetPaperItemErrorInfoBySchool', {
                    paperId:this.paids,
                    userId:this.userid,
                    ItemId:qid
                }).then(res=>{
                    var Data=res.Data
                    this.allAnswerDetail[index]=Data
                    console.log(this.allAnswerDetail);
                    const allCount=this.allAnswerDetail[index].map(item=>{
                        return item.Count
                    })
                    var len=Data.length
                    var countNums=0
                    for (var value of allCount) {
                        countNums+=value
                    }
                    this.countNum[index]=countNums
                    console.log(this.countNum);
                    this.$forceUpdate()
                })
            }else{
                styles.display="none"
            }
        },
        showAccuracy(){
            this.load=true
            this.$http.post('/report/TeacherReport/GetPaperOneByOneAnalyze', {
                PaperId:this.paids,
                userId:this.userid
            }).then(res=>{
                this.load=false
                var Data=res.Data.Items
                console.log(Data);
                this.allDatas=Data

                this.allAnswerDetail=new Array(Data.length).fill('')
                this.countNum=new Array(Data.length).fill('')

                const questionCount=Data.map(item=>{
                    return `第${item.Num}题`
                })
                const schoolAccuracy=Data.map(item=>{
                    return item.SchoolPer
                })
                const allAccuracy=Data.map(item=>{
                    return item.AllPer
                })
                this.allDatas.forEach((item, index) => {
                    const reg = /(#&\d+@)/g
                    const inputele = ' '
                    const stem = item.OriItemTitle.replace(reg, inputele)
                    item.OriItemTitle = stem
                })
                console.log(questionCount,schoolAccuracy,allAccuracy);
                const myChart1 = echarts.init(this.$refs.mycharts1)
                 myChart1.setOption({
                    xAxis:{
                        data: questionCount
                    },
                    series: [
                    {
                        data: schoolAccuracy
                    },
                    {
                        data: allAccuracy
                    }
                ]
                 })
            })
        },
        init(){
            const myChart1 = echarts.init(this.$refs.mycharts1)
            myChart1.setOption({
                title: {
                    left: 'center'
                },
                color: ['#FF8B7F', '#71B6A1'],
                tooltip: {
                    trigger: 'axis',
                },
                legend: {
                    data: [ '我校正确率','全区正确率'],
                    top:30,
                    itemGap: 150,
                },
                xAxis: {
                    type: 'category',
                    splitLine: {show: false},
                },
                 yAxis: {
                    type: 'value',
                    max:100,
                    min:0,
                    name: '(百分比：%)',
                },
                grid: {
                    left: '3%',
                    right: '4%',
                    bottom: '3%',
                    containLabel: true
                },
               
                series: [
                    {
                        name: '我校正确率',
                        type: 'line',
                    },
                    {
                        name: '全区正确率',
                        type: 'line',
                    }
                ]
            })
            window.addEventListener('resize',function(){
                myChart1.resize()
            })
        },
    }
}
</script>
<style lang="less" scoped>
 
#title{
    width: 100%;
    background-color: white;
    text-align: center;
    font-size: 20px;
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
.bottom{
    width: 100%;
    border-radius: 5px;
    background-color: white;
    padding: 20px 20px;
    overflow: hidden;
}
.chapterTitle{
    font-size: 20px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #4D5753;
}
.showProcress{
    display: inline-block;
    padding: 20px 20px;
    background-color: #F4F4F4;
    width: 100%;
}
.process{
    position: relative;
    width: 120px;
    height: 110px;
    display: inline-block;
    // margin: 10px 0px;
}
.proce{
    width: 100%;
    height: 70%;
    text-align: center;
}
.number{
    width: 30%;
    height: 30%;
    text-align: center;
    margin: 0 auto;
}
.bottomborder{
    border-bottom: 1px dashed;
}
.showQuestion{
    width: 100%;
    height:  900px;
    overflow-y: scroll;
    background-color: white;
}
.type{
    width: 100%;
}
.question{
    padding: 10px 30px;
    width: 100%;
}
.answer{
    width: 65%;
    padding: 10px 30px;
    background-color: #ece9ea;
}
.answerDetail{
    width: 100%;
    padding: 10px 0px;
    background-color: #ece9ea;
    display:none;
}
.anserTitle{
    font-size: 16px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #4D5753;
}
.options{
    display: inline-block;
}
.progressSpan{
    display: inline-block;
    width: 250px;
    height: 15px;
    position: relative;
    /deep/.ant-progress-line{
        width:100%;
    }
    
}
.packInfo{
    cursor: pointer;
    color: #108ee9;
}
.isShowAnswer{
    display: none;
}
.detailOpt{
    
    display: inline-block;
}
.detailOpt:nth-of-type(1){
    white-space: nowrap;
}
.detailOptProgress{
    display: inline-block;
    width: 250px;
    height: 15px;
}
.Accuracy{
    color:#f6878a;
}
.AccuracyPrecent{
    font-size: 14px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
}
/deep/.ant-progress-text{
    font-size: 16px;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #fd7b47;
}
.ansers{
    max-width: 400px;
    min-width: 120px;
    text-align: right;
}
/deep/.ant-progress-bg{
    background-color: #FF682C;
}
/deep/ .ant-progress-success-bg{
    background-color:#1890FF;
}
.allAnswer{
    margin: 5px 0px;
}
.allAnswer span{
    margin: 3px 0px;
}
.anserTitle{
    /deep/.ant-progress-inner{
        background-color: #FF682C;
    }
}
.progress{
    display: inline-block;
    width: 200px;
    height: 12px;
    border-radius: 100px;
    // border-bottom-right-radius: 100px;
    // border-top-right-radius: 100px;
    background-color: #d1cecf;
    vertical-align: middle;
}
</style>