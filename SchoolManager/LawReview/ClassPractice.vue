<template>
  <div class="wrapper">
    <div class="head">
      <h3>
        <i class="icon_tit"></i>
        <!-- {{ chapterId }} -->
        <span>{{ infoData.ClassName }}</span> <span> 共{{ infoData.StudentCount }}人</span>
        <span class="margL">出题人：{{ infoData.TrueName }}</span>
        <span> 练习发布总份数</span>
        <span class="green">{{ infoData.AllCustomExerciseCount }}</span>
        <span>已完成批改总数</span>
        <span class="green"> {{ infoData.StudentCorrectCount }} </span>
      </h3>
      <div class="search-info first">
        <span class="label"> 班级列表： </span>
        <a-button class="button active" disabled> 班级练习列表 </a-button>
        <a-button class="button" @click="toPractice()"> 班级学生列表 </a-button>
      </div>
      <div class="search-info next">
        <span class="label"> 排序与筛选： </span>

        <a-button
          :class="{ active: levelActive == index }"
          v-for="(item, index) in levelList"
          :key="index"
          class="button"
          @click="changeList(index)"
        >
          {{ item.ChapterName }}</a-button
        >
        <!--         <a-button class="button"> 第二单元 </a-button>
        <a-button class="button"> 第三单元 </a-button>
        <a-button class="button"> 第四单元 </a-button>
        <a-button class="button"> 第五单元 </a-button> -->
      </div>
      <div class="totals">
        {{ chapterName }} <span>共{{ infoDataSingle.CustomExerciseCount  }}份</span>
      </div>
    </div>
    <ul class="list" v-if="list">
      <li class="item" v-for="(item, index) in list" :key="index">
        <div class="topic-info">
          <div class="title-info">
            <h3>{{ item.ExerciseName }}</h3>
             <a-tooltip
                  placement="right"
                  :title="item.FirstChapterName"
                  :auto-adjust-overflow="false"
                >
                   <div class="tit-box">{{ item.FirstChapterName }}</div>
                </a-tooltip>
         <a-tooltip
                  placement="right"
                  :title="item.ChapterName"
                  :auto-adjust-overflow="false"
                >
                   <div class="tit-box">{{ item.ChapterName }}</div>
                </a-tooltip>

           
          </div>
          <div class="time">作业发布时间：{{ item.PublishTime }}</div>
        </div>
        <div class="content-info">
          <div class="numbs-info">
            <div class="infos-item">
              <div class="nums">{{ item.AvgAccuracy }}%</div>
              <p>平均正确率</p>
            </div>
            <div class="infos-item">
              <div class="nums">
                {{ item.StudentCorrectCount }}/<span class="gray">{{ item.StudentCount }}</span>
              </div>
              <p>批改完成份数</p>
            </div>
            <div class="infos-item">
              <div class="nums">{{ item.WordCommentCount }}</div>
              <p>文字点评记录数</p>
            </div>
            <div class="infos-item">
              <div class="nums">{{ item.VoiceCommentCount }}</div>
              <p>语音点评记录数</p>
            </div>
          </div>
          <a-button class="button" @click="toStudentPractice(item.ExerciseId)"> 查看 </a-button>
        </div>
      </li>

      <!-- <li class="item">
        <div class="topic-info">
          <div class="title-info">
            <h3>2020.11.11数学练习</h3>
            <div class="tit-box">一 十以内的数</div>
            <div class="tit-box">说一说</div>
          </div>
          <div class="time">作业发布时间：2020-11-11 9:25</div>
        </div>
        <div class="content-info">
          <div class="numbs-info">
            <div class="infos-item">
              <div class="nums">89%</div>
              <p>平均正确率</p>
            </div>
            <div class="infos-item">
              <div class="nums">30/<span class="gray">36</span></div>
              <p>批改完成份数</p>
            </div>
            <div class="infos-item">
              <div class="nums">20</div>
              <p>文字点评记录数</p>
            </div>
            <div class="infos-item">
              <div class="nums">15</div>
              <p>语音点评记录数</p>
            </div>
          </div>
          <a-button class="button"> 查看 </a-button>
        </div>
      </li>
      <li class="item">
        <div class="topic-info">
          <div class="title-info">
            <h3>2020.11.11数学练习</h3>
            <div class="tit-box">一 十以内的数</div>
            <div class="tit-box">说一说</div>
          </div>
          <div class="time">作业发布时间：2020-11-11 9:25</div>
        </div>
        <div class="content-info">
          <div class="numbs-info">
            <div class="infos-item">
              <div class="nums">89%</div>
              <p>平均正确率</p>
            </div>
            <div class="infos-item">
              <div class="nums">30/<span class="gray">36</span></div>
              <p>批改完成份数</p>
            </div>
            <div class="infos-item">
              <div class="nums">20</div>
              <p>文字点评记录数</p>
            </div>
            <div class="infos-item">
              <div class="nums">15</div>
              <p>语音点评记录数</p>
            </div>
          </div>
          <a-button class="button"> 查看 </a-button>
        </div>
      </li> -->
    </ul>

    <div class="empty-mod" v-else>
      <a-empty />
    </div>
    <a-pagination
      :hideOnSinglePage="true"
      :show-total="(total) => `共 ${total}张练习`"
      @change="changePage"
      class="text-center"
      v-model="pageIndex"
      :total="totalNum"
      :defaultPageSize="pageSize"
    />

    <div class="example" v-if="isLoading">
      <a-spin />
    </div>
  </div>
</template>

<script>
export default {
  watch: {
    /* totalNum(val) {
        
        this.$nextTick(()=>{

        });
      }, */
  },
  data() {
    return {
      isLoading: true,
      //列表
      list: [],
      //当前总数数
      totalNum: 0,
      //当前页
      pageIndex: 1,
      //当前页数
      pageSize: 10,
      chapterId: '',
      //单元列表
      levelList: [],
      //当前单元
      levelActive: 0,
      chapterName: '',
      //班级id
      ClassId: '',
      //教师id
      UserId: '',
      //信息
      infoData: '',
      infoDataSingle:{},
    }
  },
  created() {
    // this.ClassId = this.$route.query.ClassId
    // this.UserId = this.$route.query.UserId
  },
  mounted() {},
  activated() {
    this.init()
    this.getLevelList()
  },
  methods: {
    init() {
      this.ClassId = this.$route.query.ClassId
      this.UserId = this.$route.query.UserId
      this.levelActive = 0
      this.pageIndex = 1
    },
    async changePage(num) {
      this.pageIndex = num
      this.isLoading = true
      await this.getList()
      this.isLoading = false
    },

    toStudentPractice(ExerciseId) {
      const cc = {
        UserId: this.UserId,
        ClassId: this.ClassId,
        ExerciseId: ExerciseId,
      }
      this.$router.push({
        path: '/SchoolManager/LawReview/StudentsPractise',
        query: cc,
      })
    },
    toPractice() {
      const cc = {
        UserId: this.UserId,
        ClassId: this.ClassId,
      }
      this.$router.push({
        path: '/SchoolManager/LawReview/StudentsList',
        query: cc,
      })
    },
    async changeList(index) {
      this.levelActive = index
      this.chapterId = this.levelList[index].ChapterId
      this.chapterName = this.levelList[index].ChapterName
      //列表
      this.isLoading = true
      await this.getList()
      //老师的批改信息
      await this.getInfoDatas()
      this.isLoading = false
    },

    async getLevelDatas() {
      var obj = {
        // pageIndex: this.pageIndex,
        // pageSize: this.pageSize,
        // userid: this.$route.query.UserId,
        ClassId: this.ClassId,
        // chapterId: this.chapterId,
      }
      //// /Chapter/TeacherChapter/GetChapterByClass
      //获取单元列表
      await this.$uwonhttp.post('/Chapter/TeacherChapter/GetChapterByClass', obj).then((res) => {
        console.log('ok 成功了e范德萨发生')
        console.log(res.data.Data)
        this.levelList = res.data.Data
        this.chapterId = res.data.Data[0].ChapterId
        this.chapterName = res.data.Data[0].ChapterName
        // console.log(this.totalNum,"dddddddddddddddddddddddddd");
      })
    },
    async getList() {
      var obj = {
        pageIndex: this.pageIndex,
        pageSize: this.pageSize,
        // userId: this.$route.query.UserId,
        classId: this.ClassId,
        chapterId: this.chapterId,
      }
      //获取班级练习列表
      await this.$uwonhttp.post('/Customer/CustomExercise/GetClassChapterCustomExerciseList', obj).then((res) => {
        this.list = res.data.Data.Items
        this.totalNum = res.data.Data.TotalItems
      })
    },

    async getInfoDatas() {
      var obj = {
        // pageIndex: this.pageIndex,
        // pageSize: this.pageSize,
        userId: this.UserId,
        classId: this.ClassId,
        chapterId: this.chapterId,
      }
      //获取班级练习批改数据
      // /Customer/CustomExercise/GetClassCustomExercise
      const res = await this.$uwonhttp.post('/Customer/CustomExercise/GetClassCustomExercise', obj)/*  */
      if(this.chapterId==''){
 this.infoData = res.data.Data
      }
      else{
        this.infoDataSingle = res.data.Data
      }
     
    },
    async getLevelList() {
       await this.getInfoDatas()
      await this.getLevelDatas()
      await this.getInfoDatas()
      await this.getList()
     
      this.isLoading = false
    },
  },
  components: {},
  watch: {
    $route: function () {
      this.init()
    },
  },
}
</script>

<style lang="less" scoped>
.wrapper {
  margin: 16px auto;
  .example {
    position: fixed;
    width: 100%;
    height: 100%;
    // background: rgba(255, 255, 255, 0.45);
    display: flex;
    align-items: center;
    justify-content: center;
    left: 0;
    top: 0;
  }
  .head {
    // line-height: 134px;
    width: 100%;
    background-color: #fff;
    display: flex;
    align-items: flex-start;
    justify-content: space-around;
    flex-direction: column;
    padding: 10px 16px;
    border-radius: 8px;
    h3 {
      //   padding-left: 6px;
      line-height: 2;
      font-size: 14px;
      color: #2d2d2d;
      display: flex;
      justify-content: flex-start;
      align-items: center;
      i {
        width: 28px;
        height: 28px;
        overflow: hidden;
        background: url('../../../assets/schoolmamager/lowreview/icon_title.png') no-repeat center center;
        background-size: 100% 100%;
        vertical-align: middle;
        margin-right: 24px;
        display: inline-block;
      }
      span {
        margin-right: 16px;
        &.margL {
          margin-right: 64px;
        }
        &.green {
          font-size: 17px;
          color: #68bb97;
        }
      }
    }
    .search-info {
      padding-left: 20px;
      font-size: 14px;
      height: 34px;
      line-height: 34px;
      width: 100%;
      display: flex;
      align-items: flex-start;
      justify-content: flex-start;
      &.first {
        margin: 22px auto 15px;
      }
      &.next {
        margin: 0 auto 15px;
      }
      > button {
        margin-left: 16px;
      }
      .label {
        font-size: 14px;
        color: #4d5753;
        width: 120px;
        text-align: right;
        // justify-self: flex-end;
      }
      .button {
        border: none;
        // cursor: pointer;
        display: inline-block;
        // width: 106px;
        // height: 100%;
        border-radius: 2px;
        color: #97b4a7;
        background-color: #dfeae5;
        text-align: center;
        line-height: 34px;
        height: 34px;
        padding: 0 12px;
        // width: 120px;
        &:hover,
        &:focus {
          border-color: #dfeae5;
        }
        &.active {
          background-color: #68bb97;
          color: #fff;
          &:hover,
          &:focus {
            border-color: #68bb97;
          }
        }
      }
    }

    .totals {
      line-height: 2;
      font-size: 14px;
      color: #7e9a8e;
      span {
        margin-left: 10px;
      }
    }
  }

  .list {
    margin-top: 16px;
    display: flex;
    align-items: center;
    justify-content: flex-start;
    flex-direction: row;
    flex-wrap: wrap;
    .item {
      width: 49%;
      height: 160px;
      margin: 16px 0;
      padding: 24px 0;
      background: #fff;
      border-radius: 8px;
      display: flex;
      flex-direction: column;
      margin-right: 2%;
      font-size: 14px;
      &:nth-child(2n) {
        margin-right: 0;
      }
      .topic-info {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        height: 24px;
        line-height: 24px;
        color: #49534e;

        .title-info {
          display: flex;
          flex-direction: row;
          justify-content: flex-start;
          align-items: center;
          h3 {
            // margin-left: 24px;
            // margin-right: 40px;
            margin: 0  16px;
            font-size: 17px;
            max-width: 203px;
            overflow: hidden;
         text-overflow: ellipsis;
            white-space: nowrap;
          }

          .tit-box {
            // font-size: 14px;
            border-radius: 2px;
            border: 1px solid #dfdfdf;
            padding: 1px 4px;
            color: #a6a6a6;
            margin-right: 12px;
            max-width: 100px;
            overflow:hidden;
            text-overflow:ellipsis;
            white-space:nowrap; cursor:pointer;
          }
        }
        .time {
          margin-right: 16px;
          font-size: 12px;
          color: #4f4b4b;
        }
      }
      .content-info {
        margin-top: 25px;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        height: 80px;
        .numbs-info {
          flex: 1;
          margin-left: 16px;
          //   overflow: hidden;
          display: flex;
          flex-direction: row;
          justify-content: flex-start;
          align-items: center;
          height: 80px;
          // width: 70%;
          .infos-item {
            display: inline-block;
            display: flex;
            width: 25%;
            flex-direction: column;
            align-items: flex-start;
            justify-content: center;
            p {
              margin-top: 12px;
              font-size: 14px;
              color: #9ba6a9;
            }
            .nums {
              font-size: 18px;
              color: #479aeb;
              .gray {
                color: #969696;
              }
            }
          }
        }

        .button {
          margin-right: 16px;
          border: transparent 1px solid;
          cursor: pointer;
          display: inline-block;
          border-radius: 5px;
          background-color: #68bb97;
          color: #fff;
          text-align: center;
          line-height: 36px;
          // padding: 0 28px;
          height: 36px;
          width: 90px;
          text-align: center;

          &:hover,
          &:focus {
            border-color: #68bb97;
          }
        }
      }
    }
  }
  .empty-mod {
    width: 100%;
    height: 300px;
    margin-top: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .text-center {
    text-align: center;
    margin-top: 64px;
  }

  /deep/ .ant-pagination-item {
    color: rgba(0, 0, 0, 0.65);

    &.ant-pagination-item-active {
      font-weight: 500;
      background: #68bb97;
      color: #fff;
      border-color: #68bb97;
      a {
        color: #fff;
      }
    }
    &:hover,
    &:focus {
      font-weight: 500;
      background: #68bb97;
      color: #fff;
      border-color: #68bb97;
      a {
        color: #fff;
      }
    }
  }
}
</style>
