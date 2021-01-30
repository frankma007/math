<template>
    <div>
        <div class="title" @click="allReaded()">全部标记为已读</div>
        <br>
        <br>
        <div class="message" v-for="(m,index) in message" :key="index">
            <div class="detail"><a-badge status="error" v-if="m.IsRead==0"/>{{m.Content}}</div>
            <div class="time">2019-09-13</div>
            <div v-if="m.Type===1" class="toCheck" @click="toMeetingDetail(m.KeyId,m.Id)">查看详情</div>
            <div v-if="m.Type===2" class="toCheck" @click="toAllMeeting(m.Id)">参加会议</div>
            <div v-if="m.Type===3" class="toCheck" @click="toCheck(m.Id)">前往审核</div>
            <div v-if="m.Type===4" class="toCheck" @click="toMeetingDetail(m.KeyId,m.Id)">点击提交</div>
            <br>
            <br>
            <hr class="hr">
        </div>
    </div>
</template>

<script>
export default {
    data(){
        return{
            message:'',
            page:1,
            userid:'',
        }
    },
    created(){
        this.userid = localStorage.getItem('UserId')
        this.getMessage()
    },
    methods:{
        allReaded(keyid){
            // alert("暂不支持")
            this.$uwonhttp.post('/Msg/ManagerMsg/UpdateIsRead', {  userId: this.userid,InformationId:keyid })
            this.getMessage() 
        },
        getMessage(){
            this.$uwonhttp.post('/Msg/ManagerMsg/GetMyMsgList', {  userId: this.userid,pageNo:this.page}).then(res => {
                var Data=res.data.Data.Items
                this.message=Data
                console.log(this.message);
            })  
        },
        toAllMeeting(keyid){
            this.allReaded(keyid)
            this.$router.push({ path: '/SchoolManager/teacherMeeting/allMeeting'})
        },
        toCheck(keyid){
            this.allReaded(keyid)
            this.$router.push({ path: '/SchoolManager/myManagement/messageCheck'})
        },
        toMeetingDetail(mid,keyid){
            this.allReaded(keyid)
            this.$router.push({path:"/SchoolManager/teacherMeeting/meetDetail",query:{meetId:mid}})
        },
    },
}
</script>

<style lang="less" scoped>
.title{
    position: relative;
    width: 150px;
    cursor: pointer;
    font-size: 15px;
    font-weight:600;
    height: 30px;
    color: #B5B5B5;
    margin-left: 10%;
    float: right;
    top: -10px;
    // border: 1px solid;
}
.message{
    background-color: white;
    padding-left: 10px;
    padding: 10px 10px;
}
.detail{
    color:#49534E;
    margin-bottom: 10px;
    font-size: 17px;
}
.hr{
    width: 95%;
}
.time{
    margin-left:13px;
    display: inline-block;
    color: #B5B5B5;
    vertical-align: middle;
    font-size: 12px;
}
.toCheck{
    background-color: #68bb97;
    color: white;
    width: 100px;
    height: 40px;
    line-height: 40px;
    text-align: center;
    border-radius: 5px;
    display: inline-block;
    margin-left: 78%;
    vertical-align: middle;
    cursor: pointer;
}
</style>