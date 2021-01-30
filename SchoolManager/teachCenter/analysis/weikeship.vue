<template>
    <div>
        <div v-if="microVideoShow" class="noMicro">
            <p><img src="@/assets/lack/暂无微课视频.png" alt=""></p>
            <p>暂无微课视频</p>
        </div>
        <div v-if="!microVideoShow" id="microVideo">
            <div v-for="(m,index) in microVideo" :key="index"  class="video" >
                <div class="showIcon" :style="{ backgroundImage:'url(' + m.CoverUrl + ')'}" @click="videoDetail">
                    <a-icon type="play-circle" style="fontSize: 42px;color:#fff;margin-top:25%;" />
                </div>
                <div class="message">
                    <div class="detailMessage">{{m.ChapterName}}</div>
                    <div class="detailMessage" style="font-szie:15px;font-weight:500;">{{m.Name}}</div>
                    <div class="detailMessage">
                        <span class="messageSpan">时长：{{m.Duration}}</span><span class="messageSpan">课程评分：</span>
                        <br/>
                        <span class="messageSpan" style="margin-top:0px;"><img style="vertical-align: middle;" src="@/assets/teacher/时长.png" alt="">&nbsp;&nbsp;{{m.ClickNum}}</span><span class="messageSpan"><a-rate :value="2" disabled /></span>
                    </div>
                    <div class="detailMessage">发布时间：{{m.CreateTime}}</div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    data(){
        return{
            microVideoShow:'',
            microVideo:'',
        }
    },
    props:{
        paids:String,
    },
    created(){
        this.getVideo()
    },
    methods:{
        videoDetail(){
            this.$router.push({ path: '/Paper/Exam_MicroLesson/VideoDetails', query: { videoId: this.paids } })
        },
        getVideo(){
            this.$http.post('/Paper/Exam_MicroLesson/GetDataByWhere', {
                PageIndex:1,
                PageRows:8,
                chapter:this.paids,
                subjectId:2
            }).then(res => {
                var Data=res.Data
                console.log(Data.length);
                this.microVideo=Data
                if(Data.length==0){
                    this.microVideoShow=1
                }else{
                    this.microVideoShow=0
                }
                
                console.log(Data);
            })
        },
        
    }
}
</script>
<style lang="less" scoped>
#microVideo{
    position: relative;
    width: 100%;
    height: 1000px;
    overflow: scroll;
    text-align: center;
}
.video{
    padding-top:0px;
    width: 600px;
    height: 200px;
    position: relative;
    display: inline-block;
    margin: 10px 10px;
    text-align:left;
    overflow: hidden;
}
.showIcon{
    padding: 0px;
    width: 50%;
    height: 100%;
    display: inline-block;
    position: relative;
    border-radius: 10px;
    background-repeat:no-repeat;
    background-size:100% 100%;
    cursor: pointer;
    text-align: center;
}
.message{
    padding: 0px;
    padding-left: 10px;
    left: 50%;
    width: 50%;
    height: 100%;
    display: inline-block;
    position: absolute;
}
.noMicro {
    text-align: center;
  }
.detailMessage:nth-of-type(2){
    font-size:20px;
    font-weight:500px;
    margin-top: 10px;
}
.detailMessage:nth-of-type(3){
    margin-top: 10px;
}
.messageSpan{
    display: inline-block;
    width:140px;
    height: 30px;
    // border: 1px solid;
}
.detailMessage:last-child{
    font-size: 12px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #B5B5B5;
  }
</style>