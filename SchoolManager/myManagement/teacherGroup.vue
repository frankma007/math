<template>
  <div>
      <div class="top">
           <a-select default-value="0" style="width: 120px;height:100%" @change="handleChange">
               <a-select-option value="0">
                    所有年级
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
            &nbsp;&nbsp;
            <a-input v-model="teacherName" placeholder="搜索老师姓名" style="border-radius:50px;width:120px;height:100%;">
            </a-input>
            &nbsp;&nbsp;
            <span class="button" @click="search">搜索</span>
            &nbsp;&nbsp;
            <span class="button" @click="reset">重置</span>
      </div>
      <br>
      <div class="middle">
          排序：<span class="order" :class="{'color1':orderColor==0}" @click="changeState(0)">出卷数</span> 
                <span class="order" :class="{'color1':orderColor==1}" @click="changeState(1)">出题数</span>
                <span class="order" :class="{'color1':orderColor==2}" @click="changeState(2)">视频数</span>
                <span class="order" :class="{'color1':orderColor==3}" @click="changeState(3)">布置作业数</span>
      </div>
      <br>
      <div class="bottom">
          <div class="message" v-for="(m,index) in allTeacher" :key="index">
              <span class="photo">
                  <img class="img" :src="m.Photo">
              </span>
              <span class="teacherMessage">
                  <span class="name">{{m.trueName}}</span><br>
                  <span class="allClass">{{m.ClassNames}}</span>
              </span>
              <hr>
              <div class="leadClass">
                  <span class="LeadDetail">出卷数     <br><span class="number">{{m.FinishCount}}</span></span>
                  <span class="LeadDetail">出题数     <br><span class="number">{{m.ItemCount}}</span></span>
                  <span class="LeadDetail">视频数     <br><span class="number">{{m.VideoCount}}</span></span>
                  <span class="LeadDetail">布置作业数 <br><span class="number">{{m.FinishCount}}</span></span>
              </div>
          </div>
          <p style="text-align:center">
            <a-pagination :defaultCurrent="1" :total="pageNum" :defaultPageSize="15" @change="handleChangePage" :hideOnSinglePage="true"/>
            </p>
          <br>
      </div>
  </div>
</template>

<script>
export default {
    data(){
        return{
            allTeacher:'',
            //总页数
            pageNum:50,
            //当前年级
            gradeNum:0,
             //排序状态
            orderState:1,
            //教师名字
            teacherName:"",
            //当前页数
            currentPage:1,
            userid:'',
            orderColor:'',
        }
    },
    created(){
        this.userid = localStorage.getItem('UserId')
        this.getAllMessage()
    },
    methods:{
        getAllMessage(){
            this.$uwonhttp.post('/ManagerPersonalsController/ManagerPersonals/GetTeacherManager', { 
                userId: this.userid,
                pageNo:this.currentPage,
                gradeId:this.gradeNum,
                keyWords:this.teacherName,
                order:this.orderState,
            }).then(res => {
                var Data=res.data.Data
                console.log(res.data.Data);

                this.allTeacher=Data.Items
                this.pageNum=Data.TotalItems
            })
        },
        handleChange(val){
            this.gradeNum=val
            console.log(val);
            this.getAllMessage()
        },
        changeState(val){
            console.log(val);
            if(val==0){
                this.orderState=3
                this.orderColor=0
            }else if(val==1){
                this.orderState=5
                this.orderColor=1
            }else if(val==2){
                this.orderState=7
                this.orderColor=2
            }else if(val==3){
                this.orderState=1
                this.orderColor=3
            }
            this.getAllMessage()
        },
        handleChangePage(num){
            this.currentPage=num
            console.log(num);
            this.getAllMessage()
        },
        search(){
            this.getAllMessage()
        },
        reset(){
            this.teacherName=''
        },
    }
}
</script>

<style lang="less" scoped>
.top{
    height: 30px;
    /deep/.ant-select-selection{
        border-radius: 100px;
        height: 100%;
    }
}
.order{
    margin-left: 20px;
    display: inline-block;
    cursor: pointer;
}
.order:nth-of-type(1){
margin-left: 0px;
}
.color1{
    color:#68bb97;
}
.bottom{
    margin-left: -20px;
    position: relative;
    width: 100%;
    height: 650px;
    // text-align: center;
}
.message{
    margin: 10px 20px;
    display: inline-block;
    position: relative;
    text-align: left;
    height: 200px;
    width: 400px;
    min-width: 300px;
    padding: 20px 0px;
    border-radius: 3px;
    background-color: white;
    vertical-align: middle;
}
.photo{
    display: inline-block;
    vertical-align: top;
    text-align: center;
    display: inline-block;
    height: 40%;
    width: 27%;
}
.img{
    width: 50px;
    height: 50px;
}
.teacherMessage{
    display: inline-block;
    width: 73%;
}
.leadClass{
    height: 40%;
    width: 100%;
    text-align: center;
    position: relative;
    line-height: 30px;
}
.LeadDetail{
    height: 20px;
    font-size: 12px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    display: inline-block;
    margin: 0 15px;
    color: #9B9B9B;
    position: relative;
    vertical-align: middle;
} 
.number{
    font-size: 16px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #4D5753;
    display: inline-block;
    vertical-align: middle;
}
.button{
    cursor: pointer;
    display: inline-block;
    width: 120px;
    height:100%;
    line-height: 30px;
    border-radius: 40px;
    color: white;
    background-color: #6bbd9a;
    text-align: center;
    font-size: 13px;
}
.button:nth-of-type(2){
    background-color: #A2C240;
}
.name{
    font-size: 16px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #4D5753;
}
hr{
    border-bottom: none;
    margin: 16px 0px;
    border-top: 0.1px solid #e5e5e5;
}
.bottom{
    /deep/.ant-pagination-item-active{
        background: #68BB97;
        border-color: #68BB97;
        color:black;
    }
    /deep/.ant-pagination-item a{
        color: black;
    }
}
</style>
