<template>
  <div>
    <div class="nav-bar">
      <!-- <p><span style="width:100px">练习属性：</span><span :class="{ 'color1' : paperTypeblue == '1' }" @click="changePaperType(1)">全区练习</span>
          <span :class="{ 'color1' : paperTypeblue == '2' }" @click="changePaperType(2)">班级练习</span></p> -->
      <p style="font-size:18px"><span style="width:100px">发布状态: </span>
        <!-- <span :class="{ 'color1' : racticeblue == '1' }" @click="changeRactice(0)">全部</span> -->
        <span :class="{ 'color1' : racticeblue == 1 }" @click="changeRactice(1)">已发布</span>
        <span :class="{ 'color1' : racticeblue == 0 }" @click="changeRactice(0)">待发布</span>
      </p>
    </div>
    <div class="example" v-if="loadMind" >
      <a-spin />
    </div>
    <!-- 列表页 已发布 -->
    <div class="clearfix">
      <div class="clearfix" v-for="(item,index) in mindList" :key="index">
        <p class="time-tit">{{ item.DateTimeDayStr }}</p>
        <div v-show="racticeblue === 1" class="mind-list fl" v-for="(ite,inde) in item.CustomExercisePaperList" :key="inde">
          <p class="mind-tit">{{ ite.PaperTitle }}</p>
          <p class="f-z" v-for="(it,ind) in ite.CustomExerciseClassList" :key="ind">
            <span class="mind-btn">{{ it.ClassName }}</span>
            <span class="com-m-r"><span class="com-col">{{ it.DoStudentCnt }}</span>/{{ it.ClassStudentCnt }}完成</span>
            <span>
              <span class="re-col">{{ it.ToDoCheckCnt }}</span>人待批改</span>
          </p>
          <div class="clearfix" v-if="racticeblue === 1">
            <span class="fl">
              <!-- <p>发布时间：2021年1月2日 14:23</p> -->
              <p>截止时间：{{ ite.EndDateTimeStr }}</p>
            </span>
            <span @click="toCorrection(ite.Id)" class="fg Correction cur">
              去批改
            </span>
          </div>
        </div>
        <!-- 列表页 待发布 -->
        <div v-show="racticeblue === 0" class="mind-list fl" v-for="(ite,inde) in item.CustomExercisePaperList" :key="ite.ClassId">
          <p class="mind-tit">{{ ite.PaperTitle }}</p>
          <p class="release-class">
            <span>发布班级</span>&nbsp;&nbsp;

            <span class="mind-btn" v-for="(it,ind) in ite.CustomExerciseClassList" :key="ind">{{ it.ClassName }}</span>
          </p>
          <!-- <span class="com-m-r"><span class="com-col">{{ it.DoStudentCnt }}</span>/{{ it.ClassStudentCnt }}完成</span> -->

          <div class="clearfix" v-if="racticeblue === 0">
            <span class="fl">
              <!-- <p>发布时间：2021年1月2日 14:23</p> -->
              <p>截止时间：{{ ite.EndDateTimeStr }}</p>
            </span>
            <span @click="editRelease(ite.Id)" class="fg Correction cur">
              修改发布
            </span>
          </div>
        </div>

      <!-- <div class="mind-list fl">
        <p class="mind-tit">2021.01.02数学练习</p>
        <p class="f-z">
          <span class="mind-btn">一(1)班</span>
          <span class="com-m-r"><span class="com-col">5</span>/45完成</span>
          <span>
            <span class="re-col">5</span>人待批改</span>
        </p>
        <div class="clearfix">
          <span class="fl">
            <p>发布时间：2021年1月2日 14:23</p>
            <p>截止时间：2021年1月9日 14:23</p>
          </span>
          <span class="fg Correction">
            去批改
          </span>
        </div>
      </div> -->
      </div>

      <!-- 分页 -->
      <div class="paging">
        <a-pagination
          show-quick-jumper
          :default-current="pageNo"
          :total="this.pageTotal"
          :defaultPageSize="8"
          @change="onPaging"
          hideOnSinglePage />
      </div>
      <!-- <p class="footer">Copyright©上海有我科技有限公司，All Rights Reserved</p> -->
      <footerLogo :heightPage="heightPage"></footerLogo>
    </div>
  </div></template>

<script>
import '@/utils/utils.less'
import footerLogo from '@/components/newFooterLogo/newFooterLogo'
export default {
  components: {
    footerLogo
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    this.getMindList()
  },
  mounted () {
  },
  updated () {
    // this.heightPage = document.getElementById('class-task').offsetHeight
  },
  data () {
    return {
      // 加载
      loadMind: true,
      userId: '',
      heightPage: 0,
      // 发布状态
      racticeblue: 1,
      pageNo: 1,
      pageTotal: 1,
      // 智能批改列表
      mindList: []
    }
  },
  methods: {
    // 获取智能批改列表
    getMindList () {
      this.loadMind = true
      this.$uwonhttp.post('/Customer/CustomExercise/GetExercises', { pageindex: 1, pageSize: 10, teacherid: this.userId, state: this.racticeblue }).then(res => {
        console.log('智能批改', res)
        this.pageTotal = res.data.Data.length
        this.mindList = res.data.Data
        this.loadMind = false
      })
    },
    // 发布状态
    changeRactice (value) {
      if (value === 0) {
        this.racticeblue = 0
      } else {
        this.racticeblue = 1
      }
      this.getMindList()
    },
    // 修改发布
    editRelease (id) {
      this.$router.push({ path: '/Teacher/IntelligentCorrection/MindWaitRelease',
        query: {
          id
        } })
    },
    // 去批改
    toCorrection (id) {
      console.log(id)
      this.$router.push({ path: '/Teacher/IntelligentCorrection/MindComRelease',
        query: {
          id
        } })
    },
    onPaging (pageNum) {
      this.pageNo = pageNum
    }
  }
}

</script>

<style lang="less" scoped>
  .example {
    text-align: center;width:100%;
  }
  .release-class {
    padding-bottom: 8px;
    border-bottom: 1px solid #ECECEC;
  }
.color1 {
  padding-bottom: 5px;
  color: #68BB97;
  border-bottom: 2px solid #68BB97;
}
.m-r {
  margin-right: 30px;
}
.f-z {
  margin-bottom: 10px;
  padding-bottom: 10px;
  font-size: 16px;
  border-bottom: 1px solid #ECECEC;
}
.time-tit {
    margin-top: 20px;
    padding: 5px 20px;
    font-size: 18px;
    border-radius: 5px;
    background-color: #fff;
}
.m-b {
  margin-bottom: 20px;
}
.com-m-r {
  margin-right: 16px;
}
   .com-col {
     color: #68BB97;
   }
   .re-col {
     color: #F87175;
   }
  // 导航栏
  .nav-bar {
    // border-bottom: 2px solid #ccc;
    margin-bottom: 20px;
    span {
      margin-left: 11px;
      margin-right: 15px;
      cursor: pointer;
    }
  }
  .class-col {
    color: #B5B5B5;
    // background-color: #F4F4F4;
    border: 1px solid #B5B5B5;
  }

  // 分页
  .paging {
    margin-top: 20px;
    text-align: center;

    /deep/.ant-pagination-item:focus, .ant-pagination-item:hover {
      border-color: #68BB97;
      border-color: #68BB97;
    }
    /deep/.ant-pagination-item-active a {
      color: #000;

    }
  }
  .ant-pagination {
    /deep/.ant-pagination-item-active {
    border-color: #68BB97;
    color: #000;
    background-color: #68BB97;
    }
    /deep/.ant-pagination-item:hover {
      color: #000;
      border-color: #68BB97;
    }
  }
  // 智能列表
  .mind-list {
    width: 348px;
    margin-top: 10px;
    margin-right: 54px;
    height: 165px;
    padding: 15px 16px 16px 16px;
    background-color: #fff;
    border-radius: 5px;
    overflow: auto;
    .mind-tit {
      margin-bottom: 16px;
      font-size: 20px;
    }
    .mind-btn {
      display: inline-block;
      width: 69px;
      height: 23px;
      margin-right: 21px;
      line-height: 23px;
      font-size: 12px;
      text-align: center;
      color: #D8D8D8;
      border: 1px solid #D8D8D8;
      border-radius: 15px;
    }
  }
  .Correction {
    display: inline-block;
    width: 74px;
    height: 25px;
    margin-top: 10px;
    line-height: 25px;
    text-align: center;
    color: #fff;
    background-color: #68BB97;
    border-radius: 15px;
  }
</style>
