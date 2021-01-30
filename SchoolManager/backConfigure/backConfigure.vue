<template>
  <div class="web">
      <p class="title">
          预警值自定义设置
      </p>
      <br>
      <div class="warn">
          <div id="red">红色预警</div>
          <div class="type">正确率：</div>
          <div>一年级~三年级：低于<span v-html="RedLowAccuracy"></span>%  <span v-if="showEdit" class="inp"><a-input v-model="RedLowAccuracy"></a-input></span>(不含)</div>
          <br>
          <div>四年级~五年级：低于<span v-html="RedHighAccuracy"></span>%  <span v-if="showEdit" class="inp"><a-input v-model="RedHighAccuracy"></a-input></span>(不含)</div>
          <br>
          <div class="type">完成率：</div>
          <div>一年级~三年级：低于<span v-html="RedLowFinish"></span>%  <span v-if="showEdit" class="inp"><a-input v-model="RedLowFinish"></a-input></span>(不含)</div>
          <br>
          <div>四年级~五年级：低于<span v-html="RedHighFinish"></span>%  <span v-if="showEdit" class="inp"><a-input v-model="RedHighFinish"></a-input></span>(不含)</div>
      </div>
      <div class="warn">
          <div id="yellow">黄色预警</div>
          <div class="type">正确率：</div>
          <div>一年级~三年级：低于<span v-html="YellowFirstLowAccuracy"></span>% <span v-if="showEdit" class="inp"><a-input v-model="YellowFirstLowAccuracy"></a-input></span>~ <span v-html="YellowSecondLowAccuracy"></span>% <span v-if="showEdit" class="inp"><a-input v-model="YellowSecondLowAccuracy"></a-input></span>(不含)</div>
          <br>
          <div>四年级~五年级：低于<span  v-html="YellowFirstHighAccuracy"></span>% <span v-if="showEdit" class="inp"><a-input v-model="YellowFirstHighAccuracy"></a-input></span>~ <span v-html="YellowSecondHighAccuracy"></span>% <span v-if="showEdit" class="inp"><a-input v-model="YellowSecondHighAccuracy"></a-input></span>(不含)</div>
          <br>
          <div class="type">完成率：</div>
          <div>一年级~三年级：低于<span v-html="YellowLowFinish"></span>% <span v-if="showEdit" class="inp"><a-input v-model="YellowLowFinish"></a-input></span>(不含)</div>
          <br>
          <div>四年级~五年级：低于<span v-html="YellowHighFinish"></span>% <span v-if="showEdit" class="inp"><a-input v-model="YellowHighFinish"></a-input></span>(不含)</div>
      </div>
      <div class="warn">
          <div id="green">绿色预警</div>
          <div class="type">正确率：</div>
          <div>一年级~三年级：低于<span v-html="GreenLowAccuracy"></span>% <span v-if="showEdit" class="inp"><a-input v-model="GreenLowAccuracy"></a-input></span>(不含)</div>
          <br>
          <div>四年级~五年级：低于<span v-html="GreenHighAccuracy"></span>% <span v-if="showEdit" class="inp"><a-input v-model="GreenHighAccuracy"></a-input></span>(不含)</div>
          <br>
          <div class="type">完成率：</div>
          <div>一年级~三年级：低于<span v-html="GreenLowFinish"></span>% <span v-if="showEdit" class="inp"><a-input v-model="GreenLowFinish"></a-input></span>(不含)</div>
          <br>
          <div>四年级~五年级：低于<span v-html="GreenHighFinish"></span>% <span v-if="showEdit" class="inp"><a-input v-model="GreenHighFinish"></a-input></span>(不含)</div>
        <br>
      </div>
      <br>
       <a-divider></a-divider>
       <div class="button">
            <span class="buttonSpan" style="width:150px" @click="edit">
                自定义编辑
            </span> 
                &nbsp;&nbsp;
            <span class="buttonSpan" style="width:150px" @click="save">
                保存
            </span>
       </div>
  </div>
</template>

<script>
export default {
    data(){
        return{
            showEdit:'',
            //红色低年级准确率
            RedLowAccuracy:90,
            //红色高年级准确率
            RedHighAccuracy:70,
            //红色低年级完成率
            RedLowFinish:'',
            //红色高年级完成率
            RedHighFinish:'',
            //黄色低年级最低正确率
            YellowFirstLowAccuracy:'',
            //黄色低年级最高正确率
            YellowSecondLowAccuracy:'',
            //黄色高年级最低正确率
            YellowFirstHighAccuracy:'',
            //黄色高年级最低正确率
            YellowSecondHighAccuracy:'',
            //黄色高年级完成率
            YellowLowFinish:'',
            //黄色高年级完成率
            YellowHighFinish:'',
            //绿色低年级正确率
            GreenLowAccuracy:'',
            //绿色高年级正确率
            GreenHighAccuracy:'',
            //绿色低年级完成率
            GreenLowFinish:'',
            //绿色高年级完成率
            GreenHighFinish:'',
            userid:'',
        }
    },
    created(){
        this.userid = localStorage.getItem('UserId')
        this.getinitial()
        
    },
    methods:{
        getinitial(){
            this.$http.post('/HomePageDataView/TeacherManage/GetEarlyWarningSchoolMapping',{userId:this.userid}).then(res=>{
                var Data=res.Data
                console.log(res.Data);
                this.RedLowAccuracy=Data.RedLowAccuracy
                this.RedHighFinish=Data.RedHighFinish
                this.RedLowAccuracy=Data.RedLowAccuracy
                this.RedLowFinish=Data.RedLowFinish

                this.YellowFirstHighAccuracy=Data.YellowFirstHighAccuracy
                this.YellowFirstLowAccuracy=Data.YellowFirstLowAccuracy
                this.YellowHighFinish=Data.YellowHighFinish
                this.YellowLowFinish=Data.YellowLowFinish
                this.YellowSecondHighAccuracy=Data.YellowSecondHighAccuracy
                this.YellowSecondLowAccuracy=Data.YellowSecondLowAccuracy

                this.GreenHighAccuracy=Data.GreenHighAccuracy
                this.GreenHighFinish=Data.GreenHighFinish
                this.GreenLowAccuracy=Data.GreenLowAccuracy
                this.GreenLowFinish=Data.GreenLowFinish
            })
        },
        edit(){
            this.showEdit="1"
        },
        save(){
            this.$http.post('/HomePageDataView/TeacherManage/UpdateEarlyWarningSchoolMapping',{
                Id:this.userid,
                RedLowAccuracy:this.RedLowAccuracy,
                RedHighFinish:this.RedHighFinish,
                RedLowAccuracy:this.RedLowAccuracy,
                RedLowFinish:this.RedLowFinish,
                YellowFirstHighAccuracy:this.YellowFirstHighAccuracy,
                YellowFirstLowAccuracy:this.YellowFirstLowAccuracy,
                YellowHighFinish:this.YellowHighFinish,
                YellowLowFinish:this.YellowLowFinish,
                YellowSecondHighAccuracy:this.YellowSecondHighAccuracy,
                YellowSecondLowAccuracy:this.YellowSecondLowAccuracy,
                GreenHighAccuracy:this.GreenHighAccuracy,
                GreenHighFinish:this.GreenHighFinish,
                GreenLowAccuracy:this.GreenLowAccuracy,
                GreenLowFinish:this.GreenLowFinish,
                }).then(res=>{
                    if(res.Success){
                        console.log(res);
                        this.$message.success('已保存完成')
                    }else{
                        console.log(res);
                        this.$message.error('设置失败')
                    }
                })
            
            this.showEdit=0
        },
    },
}
</script>

<style lang="less" scoped>
.web{
    padding: 10px 20px;
    background-color: white;
    height: 1000px;
    border-radius: 8px;
}
.title{
    font-size: 20px;
    font-weight: 600;
    color: #49534E;
}
.warn{
    width: 33%;
    display: inline-block;
    position: relative;
    vertical-align: top;
}
#red{
    color: #F87175;
}
#yellow{
    color:#F7B500;
}
#green{
    color: #68BB97;
}
.inp{
    display: inline-block;
    width: 48px;
    height: 5px;
}
.type{
    color: #000000;
    font-weight: 500;
    font-size: 20px;
}
.button{
    margin-top: 70px;
}
.buttonSpan{
    width: 124px;
    height: 40px;
    border-radius: 40px;
    cursor: pointer;
    text-align: center;
    line-height: 40px;
    color: white;
    display: inline-block;
}
.buttonSpan:nth-of-type(1){
    background-color: #839398;
}
.buttonSpan:nth-of-type(2){
    background-color:#68BB97;
}
/deep/.inp{
    .ant-input{
        text-align: left;
        height: 20px;
    }
}
</style>