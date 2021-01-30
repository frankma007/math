<template>
  <div class="wrapper">
    <div class="head">
      <h3>
        <i class="icon_tit"></i>
        <span>三(1)班</span> <span> 共{{ totalNum }}人</span>
      </h3>
      <div class="search-info">
        <span> 班级列表： </span>
        <a-button @click="toPractice()" class="button"> 班级练习列表 </a-button>
        <a-button class="button active" disabled> 班级学生列表 </a-button>
      </div>
    </div>
        <a-row :gutter="{ xl: 16, xxl: 16 }" type="flex" justify="start" align="middle" class="list" v-if="list">
          <a-col :xl="6" :xxl="4"   v-for="(item, index) in list" :key="index">
                  <div class="item"  @click.stop="toPracticeDetails(item.StudentId)">
        <div class="students-info">
          <div class="photo">
            <img v-show="!item.Photo" src="@/assets/schoolmamager/lowreview/photo_students.png" alt />
            <img v-show="!!item.Photo" :src="item.Photo" alt />
          </div>

          <h3>{{ item.TrueName }}</h3>
        </div>
        <div class="review-info">
          <div class="infos-item">
            <div class="nums">{{ item.AVGDoPaperCorrect }}%</div>
            <p>平均正确率</p>
          </div>
          <div class="infos-item">
            <div class="nums">{{ item.TeacherStudentDoPaperCount }}</div>
            <p>被批改练习数</p>
          </div>
        </div>
      </div>
          </a-col>
        </a-row>
<!--     <ul class="list" v-if="list">

      <li class="item" v-for="(item, index) in list" :key="index" @click.stop="toPracticeDetails(item.StudentId)">
        <div class="students-info">
          <div class="photo">
            <img v-show="!item.Photo" src="@/assets/schoolmamager/lowreview/photo_students.png" alt />
            <img v-show="!!item.Photo" :src="item.Photo" alt />
          </div>

          <h3>{{ item.TrueName }}</h3>
        </div>
        <div class="review-info">
          <div class="infos-item">
            <div class="nums">{{ item.AVGDoPaperCorrect }}%</div>
            <p>平均正确率</p>
          </div>
          <div class="infos-item">
            <div class="nums">{{ item.TeacherStudentDoPaperCount }}</div>
            <p>被批改练习数</p>
          </div>
        </div>
      </li>
    </ul> -->
    <div class="empty-mod" v-else>
      <a-empty />
    </div>

    <a-pagination
      v-if="list"
      :hideOnSinglePage="true"
      :show-total="(total) => `共 ${total}人`"
      class="text-center"
      v-model="pageIndex"
      :total="totalNum"
      @change="changePage"
      :defaultPageSize="pageSize"
    />
    <div class="example" v-if="isLoading">
      <a-spin />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      list: [],
      //当前总数数
      totalNum: 0,
      //当前页
      pageIndex: 1,
      //当前页数
      pageSize: 14,
      UserId: '', //教师id
      ClassId: '', //班级id
      // StudentId:''
      isLoading: false,
    }
  },
  created() {
    // this.ClassId = this.$route.query.ClassId
    // this.UserId = this.$route.query.UserId
  },
  mounted() {},
  activated() {
    this.init()
    this.getList()
  },
  methods: {
    init() {
      this.ClassId = this.$route.query.ClassId
      this.UserId = this.$route.query.UserId
      this.pageIndex = 1
    },
    toPractice() {
      const cc = {
        UserId: this.UserId,
        ClassId: this.ClassId,
      }
      this.$router.push({
        path: '/SchoolManager/LawReview/ClassPractice',
        query: cc,
      })
    },

    toPracticeDetails(StudentId) {
      const cc = {
        UserId: this.UserId,
        ClassId: this.ClassId,
        StudentId: StudentId,
      }
      this.$router.push({
        path: '/SchoolManager/LawReview/StudentPracticeDetails',
        query: cc,
      })
    },
    changePage(num) {
      this.pageIndex = num
      this.getList()
    },
    async getList() {
      var obj = {
        pageIndex: this.pageIndex,
        pageSize: this.pageSize,
        // userId:this.$route.query.UserId,
        classId: this.ClassId,
        // classId: '1349561470114861056',
      }
      // /Customer/CustomExercise/GetClassStudentList
      //获取班级学生列表
      this.isLoading = true
      const res = await this.$uwonhttp.post('/Customer/CustomExercise/GetClassStudentList', obj)
      this.isLoading = false
      this.list = res.data.Data.Items
      this.totalNum = res.data.Data.TotalItems
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
      }
    }
    .search-info {
      margin: 15px auto 20px;
      padding-left: 20px;
      font-size: 14px;
      height: 34px;
      line-height: 34px;
      width: 100%;
      display: flex;
      align-items: flex-start;
      justify-content: flex-start;
      > button {
        margin-left: 16px;
      }
      .title {
        font-size: 14px;
        color: #4d5753;
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
        padding: 0 12px;
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
  }

  .list {
    margin-top: 12px;
    // display: flex;
    // align-items: center;
    // justify-content: flex-start;
    // flex-direction: row;
    // flex-wrap: wrap;
    .item {
      // width: 13%;
      height: 180px;
      margin: 16px 0;
      padding: 16px 0;
      background: #fff;
      border-radius: 8px;
      display: flex;
      flex-direction: column;
      justify-content: space-around;
      // margin-right: 1.5%;
      cursor: pointer;
      // &:nth-child(7n) {
      //   margin-right: 0;
      // }
      .students-info {
        display: flex;
        flex-direction: row;
        // justify-content: flex-start;
        align-items: center;
   
          justify-content: center;
        height: 84px;
        .photo {
          display: flex;
          justify-content: center;
          align-items: center;
          width: 50%;
         

          img {
            width: 50px;
             border-radius: 50%;
          }
        }
        h3 {
          flex: 1;
          font-size: 18px;
          color: #49534e;
          text-align: center;
        }
      }
      .review-info {
        display: flex;
        flex-direction: row;
        align-items: flex-start;
        justify-content: space-around;
        .infos-item {
          width:50%;
          display: inline-block;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          p {
            font-size: 14px;

            color: #9ba6a9;
          }
          .nums {
            font-size: 24px;
            color: #479aeb;
          }
        }
      }
    }
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
