<template>
  <div class="wrapper">
    <div class="head">
      <h3>
        <i class="icon_tit"></i>
        <span>{{ infoData.ClassName }}</span> <span> 共{{ infoData.StudentCount }}人</span>
      </h3>

      <div class="search-info first">
        <div>
          <a-button :class="{ active: activeIndex == '' }" class="button" @click="changeList('')"> 全部 </a-button>
          <a-button :class="{ active: activeIndex == 4 }" class="button" @click="changeList('4')"> 已完成 </a-button>
          <a-button :class="{ active: activeIndex == 3 }" class="button" @click="changeList('3')"> 订正中 </a-button>
          <a-button :class="{ active: activeIndex == 2 }" class="button" @click="changeList('2')"> 待批改 </a-button>
          <a-button :class="{ active: activeIndex == 1 }" class="button" @click="changeList('1')"> 待做 </a-button>
        </div>
        <a-button class="button active detail" @click="toDetail"> 查看练习详情 </a-button>
      </div>
    </div>

    <a-row :gutter="{ xl: 16, xxl: 16 }" type="flex" justify="start" align="middle" class="list" v-if="list">
          <a-col :xl="6" :xxl="4"   v-for="(item, index) in list" :key="index">
             <div class="item" >
        <div class="topic-info">
          <div class="photo-info" v-show="item.ExerciseState != 4">
            <img src="@/assets/schoolmamager/lowreview/default.png" alt />
          </div>

          <div class="photo-info" v-show="item.ExerciseState == 4" @click="blow(item.StudentId)">
            <img :src="item.PaperImgUrls[0]" alt />
            <div class="blow"></div>
          </div>
          <div class="student-info">
            <div class="name-info">
              <div
                class="name"
                :class="{ compelete: item.ExerciseState == '4' }"
                @click="toStudentPracticeDetails(item.ExerciseState, item.ExerciseId, item.StudentId)"
              >
                {{ item.StudentName }}
              </div>
              <div
                class="smail"
                :class="{
                  gray: item.ExerciseState == 1 || item.ExerciseState == 2 || item.ExerciseState == 3,
                  lianghao: item.Epilogue == '良好',
                  hege: item.Epilogue == '合格',
                  xunuli: item.Epilogue == '须努力',
                  manfen: item.Epilogue == '满分',
                }"
              ></div>
            </div>

            <div class="tool-info">
              <ul class="tools" v-show="item.CommentState == 1">
                <li class="icon_pen"></li>
                <li class="icon_voice"></li>
              </ul>
              <ul class="tools" v-show="item.CommentState == 2">
                <li class="icon_pen"></li>
              </ul>
              <ul class="tools" v-show="item.CommentState == 3">
                <li class="icon_voice"></li>
              </ul>
            </div>
          </div>
        </div>
        <div class="evaluate-info">
          <div v-show="item.Epilogue">等第评价:{{ item.Epilogue }}</div>
          <div v-show="!item.Epilogue">等第评价:--</div>

          <div
            class="btn"
            :class="{ error: item.ExerciseState == 1 || item.ExerciseState == 2, gray: item.ExerciseState == 3 }"
            @click="tologs(item.ExerciseState, item.ExerciseId, item.StudentId)"
          >
            {{ item.ExerciseStateName }}
          </div>
        </div>
        <div class="desc-info">
          <div class="accuracy">
            <p>答题正确率</p>
            <div class="numbs" v-show="item.AvgAccuracy != '0'">{{ item.AvgAccuracy }}%</div>
            <div class="numbs" v-show="item.AvgAccuracy == '0'">--</div>
          </div>
          <div class="time">
            <p>批改时间</p>
            <p v-show="!item.CorrectTime">--</p>
            <p v-show="item.CorrectTime">
              {{ item.CorrectTime }}
            </p>
          </div>
        </div>
      </div> 
          </a-col>
    </a-row>
<!--     <ul class="list" v-if="list">
      <li class="item" v-for="(item, index) in list" :key="index">
        <div class="topic-info">
          <div class="photo-info" v-show="item.ExerciseState != 4">
            <img src="@/assets/schoolmamager/lowreview/default.png" alt />
          </div>

          <div class="photo-info" v-show="item.ExerciseState == 4" @click="blow(item.StudentId)">
            <img :src="item.PaperImgUrls[0]" alt />
            <div class="blow"></div>
          </div>
          <div class="student-info">
            <div class="name-info">
              <div
                class="name"
                :class="{ compelete: item.ExerciseState == '4' }"
                @click="toStudentPracticeDetails(item.ExerciseState, item.ExerciseId, item.StudentId)"
              >
                {{ item.StudentName }}
              </div>
              <div
                class="smail"
                :class="{
                  gray: item.ExerciseState == 1 || item.ExerciseState == 2 || item.ExerciseState == 3,
                  lianghao: item.Epilogue == '良好',
                  hege: item.Epilogue == '合格',
                  xunuli: item.Epilogue == '须努力',
                  manfen: item.Epilogue == '满分',
                }"
              ></div>
            </div>

            <div class="tool-info">
              <ul class="tools" v-show="item.CommentState == 1">
                <li class="icon_pen"></li>
                <li class="icon_voice"></li>
              </ul>
              <ul class="tools" v-show="item.CommentState == 2">
                <li class="icon_pen"></li>
              </ul>
              <ul class="tools" v-show="item.CommentState == 3">
                <li class="icon_voice"></li>
              </ul>
            </div>
          </div>
        </div>
        <div class="evaluate-info">
          <div v-show="item.Epilogue">等第评价:{{ item.Epilogue }}</div>
          <div v-show="!item.Epilogue">等第评价:--</div>

          <div
            class="btn"
            :class="{ error: item.ExerciseState == 1 || item.ExerciseState == 2, gray: item.ExerciseState == 3 }"
            @click="tologs(item.ExerciseState, item.ExerciseId, item.StudentId)"
          >
            {{ item.ExerciseStateName }}
          </div>
        </div>
        <div class="desc-info">
          <div class="accuracy">
            <p>答题正确率</p>
            <div class="numbs" v-show="item.AvgAccuracy != '0'">{{ item.AvgAccuracy }}%</div>
            <div class="numbs" v-show="item.AvgAccuracy == '0'">--</div>
          </div>
          <div class="time">
            <p>批改时间</p>
            <p v-show="!item.CorrectTime">--</p>
            <p v-show="item.CorrectTime">
              {{ item.CorrectTime }}
            </p>
          </div>
        </div>
      </li>
    </ul> -->
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
      <!-- :defaultCurrent="pageIndex" -->
    <transition
      enter-active-class="animate__animated animate__fadeIn"
      leave-active-class="animate__animated animate__fadeOut"
    >
      <Swiper :imgActive="imgActive" :list="imgArr" v-if="isblow">
        <template v-slot:close>
          <div class="close" @click.stop="isblow = false">
            <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
          </div>
        </template>

        <template v-slot:imgList>
          <div class="swiper-slide" v-for="(item, index) in imgArr" :key="index">
            <div class="title">{{ item.StudentName }}</div>
            <div class="img-box">
              <img v-for="(itm, index) in item.PaperImgUrls" :key="index" :src="itm" alt />
            </div>
          </div>
        </template>
      </Swiper>
    </transition>
    <div class="example" v-if="isLoading">
      <a-spin />
    </div>
  </div>
</template>

<script>
import Swiper from './components/Swiper.vue'
export default {
  data() {
    return {
      isblow: false,

      chapterId: '',
      totalNum: 0,
      pageIndex: 1,
      //当前页数
      pageSize: 12,
      list: [],
      //学生id
      // StudentId: '',
      ClassId: '',
      UserId: '',
      // levelList: [],
      //当前单元
      // levelActive: 0,
      // chapterName: '',
      isLoading: true,
      // allInfoData: null,
      infoData: {},
      ExerciseId: '',
      //状态
      exerciseState: '',
      activeIndex: '',
      imgArr: [],
      imgActive: 0,
    }
  },
  created() {},
  mounted() {},
  activated() {
    this.init()
    this.getLevelList()
  },
  methods: {
    init() {
      this.ClassId = this.$route.query.ClassId
      this.UserId = this.$route.query.UserId
      this.ExerciseId = this.$route.query.ExerciseId
      this.activeIndex = ''
      this.exerciseState = ''
      this.pageIndex =1
    },
    //点击查看练习详情
    toDetail() {
      const cc = {
        UserId: this.UserId,
        ClassId: this.ClassId,
        // StudentId: StudentId,
        ExerciseId: this.ExerciseId,
      }
      this.$router.push({
        path: `/SchoolManager/LawReview/PracticeDetails`,
        query: cc,
      })
    },
    //点击已完成 跳转至练习已完成的修改记录页面
    tologs(ExerciseState, ExerciseId, StudentId) {
      if (ExerciseState != 4) {
        return
      }
      const cc = {
        UserId: this.UserId,
        ClassId: this.ClassId,
        StudentId: StudentId,
        ExerciseId: ExerciseId,
      }
      this.$router.push({
        path: `/SchoolManager/LawReview/CorrectLogsDetails`,
        query: cc,
      })
    },

    //点击姓名跳转到练习详情页面
    toStudentPracticeDetails(ExerciseState, ExerciseId, StudentId) {
      if (ExerciseState != 4) {
        return
      }
      const cc = {
        UserId: this.UserId,
        ClassId: this.ClassId,
        StudentId: StudentId,
        ExerciseId: ExerciseId,
      }
      this.$router.push({
        path: `/SchoolManager/LawReview/StudentPracticeDetails`,
        query: cc,
      })
    },
    async changePage(num) {
      this.pageIndex = num
      this.isLoading = true
      await this.getList()
      this.isLoading = false
    },
    async changeList(state) {
      if (this.activeIndex == state) {
        return
      }
      this.isLoading = true
      this.activeIndex = state
      this.exerciseState = state
      this.pageIndex = 1
      await this.getList()
      this.isLoading = false
    },
    async getLevelList() {
      this.isLoading = true
      await this.getInfoDatas()
      await this.getList()

      this.isLoading = false
    },
    blow(StudentId) {
      this.isblow = true

      let index = this.imgArr.findIndex((val) => {
        return (val.StudentId = StudentId)
      })
      this.imgActive = index
    },
    emClose() {
      this.isblow = false
    },
    async getList() {
      var obj = {
        pageIndex: this.pageIndex,
        pageSize: this.pageSize,

        classId: this.ClassId,

        exerciseState: this.exerciseState,
        exerciseId: this.ExerciseId,
      }
      //获取学生练习列表
      const res = await this.$uwonhttp.post('/Customer/CustomExercise/GetStudentCustomExerciseList', obj)
      this.list = res.data.Data.Items
      if (this.list) {
        this.list.forEach((val) => {
          if (val.PaperImgUrls) {
            val.PaperImgUrls = val.PaperImgUrls.split(',')
          }
        })
        this.imgArr = this.list.filter((val) => {
          return val.ExerciseState == '4'
        })
        console.log(this.imgArr, 'aaaaaaaaaaaaaaa')
      }
      this.totalNum = res.data.Data.TotalItems
    },
    async getInfoDatas() {
      var obj = {
        userId: this.UserId,
        classId: this.ClassId,
        chapterId: '',
      }
      //获取班级练习批改数据
      await this.$uwonhttp.post('/Customer/CustomExercise/GetClassCustomExercise', obj).then((res) => {
        this.infoData = res.data.Data
      })
    },
  },

  components: {
    Swiper,
  },
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
        // &.margL {
        //   margin-right: 64px;
        // }
        // &.green {
        //   font-size: 17px;
        //   color: #68bb97;
        // }
      }
    }
    .search-info {
      // padding-left: 20px;
      font-size: 14px;
      height: 34px;
      line-height: 34px;
      width: 100%;
      display: flex;
      align-items: flex-start;
      justify-content: space-between;
      &.first {
        margin: 15px auto;
        padding: 0 28px 0 0;
      }

      .button {
        margin-right: 16px;
        border: none;
        cursor: pointer;
        display: inline-block;
        // width: 106px;
        // height: 100%;
        border-radius: 2px;
        color: #97b4a7;
        background-color: #dfeae5;
        text-align: center;
        line-height: 34px;
        height: 34px;
        // padding: 0 12px;
        width: 120px;
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
          &.detail {
            margin-right: 0;
          }
        }
      }
    }
  }

  .list {
    // margin-top: 16px;
    margin-bottom: 16px;
    display: flex;
    align-items: center;
    justify-content: flex-start;
    flex-direction: row;
    flex-wrap: wrap;
    .item {
      // width: 16.2%;
      // height: 347px;
      margin: 16px 0 0;
      padding: 15px 10px 10px;
      background: #fff;
      border-radius: 8px;
      display: flex;
      flex-direction: column;
      // margin-right: 0.56%;
      font-size: 14px;
      // &:nth-child(6n) {
      //   margin-right: 0;
      // }
      .topic-info {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        // height: 24px;
        line-height: 24px;
        color: #49534e;

        height: 222px;

        .photo-info {
          display: flex;
          flex-direction: row;
          justify-content: flex-start;
          align-items: center;
          cursor: pointer;
          width: 167px;
          img {
            height: 222px;
            overflow: hidden;
            width: 100%;
          }
          position: relative;
          .blow {
            position: absolute;
            right: 2px;
            top: 2px;
            width: 24px;
            height: 24px;
            background: url('../../../assets/schoolmamager/lowreview/blow.png') no-repeat center center;
            background-size: 100% 100%;
            cursor: pointer;
          }
        }
        .student-info {
          min-width: 0;
          flex: 1;
          display: flex;
          flex-direction: column;
          justify-content: space-between;
          align-items: flex-end;
          height: 100%;

          .name-info {
            color: #525252;
            min-height: 0;
            height: 50%;
            width: 100%;
            // flex:1;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;

            .name {
              overflow: hidden;
            text-overflow: ellipsis;
            white-space:nowrap;
            max-width: 100%;
              &.compelete {
                cursor: pointer;
              }
              font-size: 15px;
              border-bottom: #68bb97 1px solid;
              // line-height: 2;
              padding-bottom: 3px;
              color: #68bb97;
              &.gray {
                border-bottom: transparent 1px solid;
                color: #525252;
              }
            }
            .smail {
              width: 60px;
              height: 78px;
              background: url('../../../assets/schoolmamager/lowreview/smail_yx.png') no-repeat center center;
              background-size: 100% 100%;
              vertical-align: middle;
              &.gray {
                background: url('../../../assets/schoolmamager/lowreview/smail_gray.png') no-repeat center center;
                background-size: 100% 100%;
              }
              &.lianghao {
                background: url('../../../assets/schoolmamager/lowreview/smail_lh.png') no-repeat center center;
                background-size: 100% 100%;
              }
              &.hege {
                background: url('../../../assets/schoolmamager/lowreview/smail_hg.png') no-repeat center center;
                background-size: 100% 100%;
              }
              &.xunuli {
                background: url('../../../assets/schoolmamager/lowreview/smail_xnl.png') no-repeat center center;
                background-size: 100% 100%;
              }
              &.manfen {
                background: url('../../../assets/schoolmamager/lowreview/smail_mf.png') no-repeat center center;
                background-size: 100% 100%;
              }
            }
            .desc {
              font-size: 14px;
            }
          }

          .tool-info {
            display: flex;

            justify-content: center;
            align-items: center;
            width: 100%;
            // margin-right:-10px;
            .tools {
              width: 44px;
              height: 84px;
              display: flex;
              flex-direction: column;
              justify-content: space-around;
              align-items: center;
              background: #dfeae5;
              border-radius: 2px;
              .icon_voice,
              .icon_pen {
                // cursor: pointer;
                width: 20px;
                height: 26px;
                background-size: 100% 100%;
                vertical-align: middle;

                // display: inline-block;
              }

              .icon_pen {
                background: url('../../../assets/schoolmamager/lowreview/icon_pen.png') no-repeat center center;
              }
              .icon_voice {
                background: url('../../../assets/schoolmamager/lowreview/icon_voice.png') no-repeat center center;

                // display: inline-block;
              }
            }
          }
        }
      }
      .evaluate-info {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        line-height: 1.7;
        border-bottom: 1px solid #eee;

        color: #68bb97;
        align-items: center;

        padding: 10px 0 4px;

        .btn {
          background: #68bb97;
          padding: 0 12px;
          border-radius: 11.4px;
          color: #fff;
          cursor: pointer;
          &.gray {
            background: #c7c7c7;
            cursor: default;
          }
          &.error {
            background: #f87175;
            cursor: default;
          }
        }
      }
      .desc-info {
        // flex: 1;
        // min-height: 0;
        height: 70px;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;

        // padding: 0px 0 4px;
        .accuracy {
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: flex-start;

          p {
            padding-top: 8px;
            color: #9f9f9f;
            line-height: 2;
          }
          .numbs {
            font-size: 27px;

            color: #68bb97;
            line-height: 1.3;
          }
        }

        .time {
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: flex-end;
          font-size: 14px;

          color: #6b6a6a;
          line-height: 1.8;
          p {
            text-align: right;
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
