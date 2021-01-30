<template>
  <div>
    <a-tabs default-active-key="0" @change="callback" >
      <a-tab-pane key="0" tab="待审核" @change="callback">
        <div class="flex">
          <div class="message" v-for="(m,index) in allMessage" :key="index">
            <span class="photo">
              <img class="img" :src="m.Photo" alt="">
            </span>
            <span class="right">
              <span class="name">{{ m.trueName }}</span><br>
              <span class="classCount">{{ m.ClassName }}</span><br>
              <span class="telephone">{{ m.PhoneNum }}</span><br>
              <span class="time">教师注册时间：<br> {{ m.CreateTime }}</span><br>
            </span>
            <hr class="hr">
            <div class="toClick">
              <span class="butt" @click="check(m.UserId,0)">拒绝</span>
              <span class="butt" @click="check(m.UserId,1)">同意</span>
            </div>
          </div>
          <a-divider></a-divider>
          <div>
            <!-- <a-pagination :defaultCurrent="1" :total="totalNum" :defaultPageSize="12" hideOnSinglePage @change="handleChange" /> -->
          </div>
          <br>
          <br>
        </div>
      </a-tab-pane>
      <a-tab-pane key="1" tab="已审核" @change="callback">
        <div class="flex">
          <div class="message" v-for="(m,index) in allMessage" :key="index">
            <span class="photo">
              <img class="img" :src="m.Photo" alt="">
            </span>
            <span class="right">
              <span class="name">{{ m.trueName }}</span><br>
              <span class="classCount">{{ m.ClassName }}</span><br>
              <span class="telephone">{{ m.PhoneNum }}</span><br>
              <span class="time">教师注册时间 ： <br>{{ m.CreateTime }}</span><br>
            </span>
            <hr class="hr">
            <div class="toClick" id="State">
              <span class="color2" v-if="m.State==2">已拒绝</span>
              <span class="color1" v-if="m.State==1">已同意</span>
            </div>
          </div>
          <a-divider></a-divider>
          <div>
          </div>
          <br>
          <br>
        </div>
      </a-tab-pane>
    </a-tabs>
  </div>
</template>

<script>
export default {
  data () {
    return {
      allMessage: '',
      // 总页数
      totalNum: 0,
      isAgree: '',
      checkstate: 0,
      userid: ''
    }
  },
  created () {
    this.userid = localStorage.getItem('UserId')
    this.getAllMessage()
  },
  methods: {
    callback (key) {
      this.checkstate = key
      this.getAllMessage()
      console.log(this.checkstate)
    },
    getAllMessage () {
      this.$uwonhttp.post('/ManagerPersonalsController/ManagerPersonals/GetTeacherReviewList', {
        userId: this.userid,
        type: 0,
        state: this.checkstate
      }).then(res => {
        var Data = res.data.Data
        this.totalNum = Data.length
        this.allMessage = Data
        console.log(Data)
      })
    },
    check (id, isPass) {
      console.log(id, isPass)
      this.$uwonhttp.post('/ManagerPersonalsController/ManagerPersonals/ExamineTeacher', {
        userId: this.userid,
        examineUserId: id,
        reason: '',
        isPass: isPass
      }).then(res => {
        this.$message.success('已审核')
        this.getAllMessage()
      })
    },
    handleChange () {

    }
  }
}
</script>

<style lang="less" scoped>
.flex{
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    width: 100%;
    margin-left: -20px;
}
.message{
     vertical-align: middle;
    margin: 10px 20px;
    position: relative;
    text-align: left;
    display: inline-block;
    width: 300px;
    border-radius: 5px;
    background-color: white;
    padding-top: 20px;
}
.photo{
    display: inline-block;
    vertical-align: top;
    text-align: center;
    display: inline-block;
    height: 60%;
    width: 27%;
}
.img{
    width: 40px;
    height: 40px;
    border-radius: 20px;
}
.right{
    width: 72%;
    display: inline-block;
    margin-top: 0px;
    vertical-align: top;
}
.name{
    font-size: 16px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #4D5753;
}
.butt{
    text-align: center;
    display: inline-block;
    position: relative;
    margin: 0px 10px;
    width: 70px;
    height: 20px;
    line-height: 20px;
    border-radius: 50px;
    background-color:#808080;
    cursor: pointer;
    color: white;
}
.butt:nth-of-type(1){
    background-color: #d9d6d9;

}
.butt:nth-of-type(2){
    background-color: #68bb97;

}

.color1{
    color: #68BB97;
}
.color2{
    color:#FF682C;
}
.toClick{
    width:100%;
    text-align:right;
    margin:10px 0px;
    text-align: center;
    padding-right:10px ;
}
.telephone{
    font-size: 14px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #6D7278;
    margin: 5px 0px;
    display: inline-block;
}
.time{
    display: inline-block;
    font-size: 12px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #B5B5B5;
}
.classCount{
    display: inline-block;
    font-size: 10px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #B5B5B5;
    margin: 5px 0px;
}
#State{
    text-align: right;
}
.hr{
    margin-top: 9px;
    border: 0.1px solid #e5e5e5;
    border-bottom: none;
}
</style>
