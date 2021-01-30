<template>
  <div>
    <div class="total" >
      <div class="total-div">
        <span class="total-span">
          <a-cascader
            :options="options"
            :field-names="{ label: 'ChapterName', value: 'ChapterId', children: 'Firsts' }"
            style="width: 300px;"
            @change="getCharpter"
            placeholder="请选择章节"/>
        </span>
        <a-select
          default-value=""
          style="width: 120px;height: 60px;"
          @change="getDataTime"
        >
          <a-select-option value="">
            全部日期
          </a-select-option>
          <a-select-option value="1">
            近一周
          </a-select-option>
          <a-select-option value="2">
            近一个月
          </a-select-option>
          <!-- <a-select-option value="3">
            自定义时间
          </a-select-option> -->
        </a-select>
        <span class="fg search-data">
          <span class="search-view">
            <a-input  v-model="content" placeholder="搜索诊断内容"/>
          </span>
          <span class="search-view-btn" @click="Search">
            搜索
          </span>
          <span class="search-view-cz" @click="reset()">
            重置
          </span>
        </span>
      </div>
    </div>
    <a-spin :spinning="loading">
      <div id="showWarn">
          <div v-for="(item,index) in dataCount" :key="index" class="dataDiv">
            <br>
            <p class="warnTime">{{warnTime[index]}}</p>
            <br>
            <br>
            <div v-for="(items,indexs) in warnList[0]" :key="indexs" class="Conetent">
              <span class="Content-p">{{items.Content}}</span>
              <span class="checkWarn" @click="detailWarn(items.Id)">查看</span>
              <a-divider></a-divider>
            </div>
          </div>
      </div>
    </a-spin>
  </div>
</template>
<script>
export default {
  data(){
    return {
      //请求参数
        options:[],
        userId:'',
        dataCount:'',
        warnTime:[],
        warnList:[],
        chapter:'',
        time:'',
        content:'',

        loading:true,
        
    }
  },
  created(){
    this.userId = localStorage.getItem('UserId')
    this.inithapter()
    //获取所有预警
    this.getAllWarnCond()
  },
  mounted(){
    this.$nextTick(() => {
      this.getAllWarnCond()
    })
  },
  
  methods: { 
    //初始化联机下拉框
    inithapter(){
        this.$http.post('/Paper/Exam_SubjectChapter/GetChapterTreeBy').then(res=>{
            var Data=res.Data
            this.options=Data
            console.log(Data);
        })
    },
    getAllWarnCond(){
      this.loading=true
        this.$http.post('/HomePageDataView/TeacherManage/GetEarlyWarningPushList',
        { 
          chapterIds:this.chapter,
          userId:this.userId,
          time:this.time,
          content:this.content,
        }).then(res => {
          this.loading=false
          let Data=res.Data
          //获取预警日期数量
          var count=Data.Items.length
          this.dataCount=count
          //获取预警日期
          const dataTime=Data.Items.map(item=>{
            return item.CreateTime
          })
          this.warnTime=dataTime
          //获取预警列表
          const warnlist=Data.Items.map(item=>{
            return item.List
          })
          this.warnList=warnlist
          console.log(this.warnList);
        })
    },
    detailWarn(val){
      this.$router.push({path:'/SchoolManager/warnDetail',query: {Id:val}})
      console.log(val);
    },
    getCharpter(val){
      console.log(val);
      if(val!=undefined){
        this.chapter=val[2]
        console.log(this.chapter);
      }
    },
    getDataTime(value){
      this.time=value
      this.getAllWarnCond()
      console.log(value);
    },
    Search(){
      this.getAllWarnCond()
    },
    reset(){
      this.content=''
    },
  },
}
</script>
<style lang="less" scoped>
  .m-b {
    margin-bottom: 17px;
  }
  .search-data {
    width: 600px;
  }
  // 搜索框
.search-view {
  display: inline-block;
  width: 300px;
  text-align: center;
}
input-placeholder {
  text-align: center;
}
.search-view .ant-input{
  width: 100%;
  height: 50px;
  text-indent: 15px;
  border-radius: 35px;
  border: 1px solid #ccc;
}
// 搜索按钮
.search-view-btn,.search-view-cz {
    display: inline-block;
    cursor: pointer;
    width: 100px;
    text-align: center;
    margin-left: 5%;
    height: 40px;
    line-height: 40px;
    background-color: rgb(104,188,150);
    border-radius: 30px;
    color: white;
}
.search-view-cz {
   background-color: rgb(162,194,67);
}
.total {
  background-color: white;
  height: 150px;
  border-radius: 5px;
//   position: absolute;
}
/deep/.ant-cascader-picker{
  height: 50px;
}
.total-div {
  padding-top: 40px;
  padding-left: 50px;
}
.dataPaper {
  margin-top: 50px;
  padding-bottom: 16px;
  background-color: white;
  position: relative;
  border-radius: 10px;
}
.dataPaper-title {
    padding-top: 1%;
    padding-left: 2%;
    font-size: 20px;
    font-weight: 900;
}
.dataPaper-chapterName {
    padding-top: 1%;
    padding-left: 2%;
    font-size: 10px;
    color: #737976;
}
.notCheck {
  background-color: rgb(80,88,83);
}
.dataPaper-info {
  width: 100px;
  height: 40px;
  line-height: 40px;
  margin-top: -3.5%;
  margin-right: 10%;
  text-align: center;
  color: white;
  background-color: rgb(104,187,152);
  border-radius: 5px;
}
.dataPaper-slogan {
  margin-top: -3.5%;
  margin-right: 10%;
}
.dataPaper-time {
  padding-left: 2%;
   padding-top: 1%;
    color: rgb(230,228,229);
}
/deep/.ant-cascader-input.ant-input {
  border-radius: 35px;
  height: 50px;
  text-align: center;
}
.total-div{
    /deep/.ant-select-selection--single {
    height: 50px;
    width: 200px;
    border-radius: 35px;
    padding-left:50px;
    padding-top:10px;
    margin-left:10% ;
  }
}

#showWarn{
    position: relative;
    background-color: white;
    margin-top:20px ;
    width: 100%;
    overflow-y:auto;
    border-radius: 5px;
    padding-left: 20px;
    overflow-y: auto;
}
.dataDiv{
 width: 98%;
}
.warnTime{
  font-size: 20px;
  font-family: PingFangSC-Medium, PingFang SC;
  font-weight: 500;
  color: #49534E;
}
.Conetent{
  width: 100%;
  height: 50px;
  margin: 25px 0px;
  padding: 10px;
}
.Content-p{
  display: inline-block;
  width: 70%;
  font-weight: 500;
  font-size: 16px;
  color: #49534E;
}
.checkWarn{
  width:100px;
  height: 30px; 
  display: inline-block;
  border-radius: 5px;
  background-color: #68bb98;
  margin-left: 13%;
  text-align: center;
  line-height: 30px;
  font-size: 15px;
  cursor: pointer;
}
</style>
