<template>
  <div class="wrapper">
    <div class="head">
      <h3>
        <i class="icon_tit"></i>
        <span>本学期：</span>
        <span>作业批改列表</span> <span> 共{{ allInfoData.CustomExerciseCount }}份</span>
        <span> 已完成批改{{ allInfoData.StudentCorrectCount }}份 </span>
        <span> 平均正确率：{{ allInfoData.AVGDoPaperCorrect }}% </span>
      </h3>

      <div class="search-info first">
        <a-button
          :class="{ active: levelActive == index }"
          v-for="(item, index) in levelList"
          :key="index"
          class="button"
          @click="changeList(index)"
        >
          {{ item.ChapterName }}</a-button
        >
      </div>
      <div class="totals">
        本单元： <span>共{{ infoData.CustomExerciseCount }}份</span>
        <span>已完成批改{{ infoData.StudentCorrectCount }}份</span>
        <span> 平均正确率：{{ infoData.AVGDoPaperCorrect }}% </span>
      </div>
    </div>
            <a-row :gutter="{ xl: 16, xxl: 16 }" type="flex" justify="start" align="middle" class="list" v-if="list">
          <a-col :xl="6" :xxl="4"   v-for="(item, index) in list" :key="index">
  <div class="item" >
        <div class="topic-info">
          <div class="photo-info" v-show="item.ExerciseState != 4">
            <img src="@/assets/schoolmamager/lowreview/default.png" alt />
          </div>

          <div class="photo-info" v-show="item.ExerciseState == 4" @click="blow(item.ExerciseId)">
            <img :src="item.Photo[0]" alt />
            <div class="blow"></div>
          </div>
          <div class="student-info">
            <div class="name-info">
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

              <div
                class="btn"
                :class="{
                  error: item.ExerciseState == '1' || item.ExerciseState == '2',
                  gray: item.ExerciseState == '3',
                }"
                @click="tologs(item.ExerciseState, item.ExerciseId, item.StudentId)"
              >
                {{ item.ExerciseStateName }}
              </div>
            </div>
          </div>
        </div>
        <div class="evaluate-info" @click="toDetail(item.ExerciseId)">
          {{ item.ExerciseName }}
        </div>
        <div class="desc-info">
          <div class="accuracy">
            <p>答题正确率</p>
            <div class="numbs" v-show="item.AVGDoPaperCorrect != '0'">{{ item.AVGDoPaperCorrect }}%</div>
            <div class="numbs" v-show="item.AVGDoPaperCorrect == '0'">--</div>
          </div>
          <div class="time">
            <div class="numbs" v-show="item.AVGDoPaperCorrect">{{ item.AVGDoPaperCorrect }}%</div>
            <div class="numbs" v-show="!item.AVGDoPaperCorrect">--</div>
            <p>
              批改时间 <span v-show="!item.CorrectTime">--</span>
              <span v-show="item.CorrectTime">
                {{ item.CorrectTime }}
              </span>
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

          <div class="photo-info" v-show="item.ExerciseState == 4" @click="blow(item.ExerciseId)">
            <img :src="item.Photo[0]" alt />
            <div class="blow"></div>
          </div>
          <div class="student-info">
            <div class="name-info">
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

              <div
                class="btn"
                :class="{
                  error: item.ExerciseState == '1' || item.ExerciseState == '2',
                  gray: item.ExerciseState == '3',
                }"
                @click="tologs(item.ExerciseState, item.ExerciseId, item.StudentId)"
              >
                {{ item.ExerciseStateName }}
              </div>
            </div>
          </div>
        </div>
        <div class="evaluate-info" @click="toDetail(item.ExerciseId)">
          {{ item.ExerciseName }}
        </div>
        <div class="desc-info">
          <div class="accuracy">
            <p>答题正确率</p>
            <div class="numbs" v-show="item.AVGDoPaperCorrect != '0'">{{ item.AVGDoPaperCorrect }}%</div>
            <div class="numbs" v-show="item.AVGDoPaperCorrect == '0'">--</div>
          </div>
          <div class="time">
            <div class="numbs" v-show="item.AVGDoPaperCorrect">{{ item.AVGDoPaperCorrect }}%</div>
            <div class="numbs" v-show="!item.AVGDoPaperCorrect">--</div>
            <p>
              批改时间 <span v-show="!item.CorrectTime">--</span>
              <span v-show="item.CorrectTime">
                {{ item.CorrectTime }}
              </span>
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
      :defaultCurrent="pageIndex"
      :total="totalNum"
      :defaultPageSize="pageSize"
    />

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
            <div class="title">{{ item.ExerciseName }}</div>
            <div class="img-box">
              <img v-for="(itm, index) in item.Photo" :key="index" :src="itm" alt />
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
      studentId: '',
      chapterId: '',
      totalNum: 0,
      pageIndex: 1,
      //当前页数
      pageSize: 12,
      list: [],
      //学生id
      StudentId: '',
      ClassId: '',
      UserId: '',
      levelList: [],
      //当前单元
      levelActive: 0,
      chapterName: '',
      isLoading: false,
      allInfoData: {},
      infoData: {},
      imgActive: 0,
      imgArr: [],
    }
  },
  created() {
   
  },
  activated() {
    this.init()
    this.getLevelList()
  },
  methods: {
    init(){
      //初始化参数
    this.ClassId = this.$route.query.ClassId
    this.UserId = this.$route.query.UserId
    this.StudentId = this.$route.query.StudentId
    this.pageIndex=1
    this.levelActive=0
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
    //点击查试题名称
    toDetail(ExerciseId) {
      const cc = {
        UserId: this.UserId,
        ClassId: this.ClassId,
        StudentId: this.StudentId,
        ExerciseId: ExerciseId,
      }
      this.$router.push({
        path: `/SchoolManager/LawReview/PracticeDetails`,
        query: cc,
      })
    },
    async changePage(num) {
      this.pageIndex = num
      this.isLoading = true
      await this.getList()
      this.isLoading = false
    },
    async changeList(index) {
      this.levelActive = index
      this.chapterId = this.levelList[index].ChapterId
      this.chapterName = this.levelList[index].ChapterName
      this.pageIndex = 1
      this.isLoading = true
      //列表
      await this.getList()
      //老师的批改信息
      await this.getInfoDatas()
      this.isLoading = false
    },
    blow(ExerciseId) {
      this.isblow = true
      let index = this.imgArr.findIndex((val) => {
        return (val.ExerciseId = ExerciseId)
      })

      this.imgActive = index
    },
    /*     emClose() {
      this.isblow = false
    }, */
    async getLevelDatas() {
      var obj = {
        // pageIndex: this.pageIndex,
        // pageSize: this.pageSize,
        // userid: this.$route.query.UserId,
        ClassId: this.ClassId,
        // chapterId: this.chapterId,
      }

      //获取单元列表
      const res = await this.$uwonhttp.post('/Chapter/TeacherChapter/GetChapterByClass', obj)
      this.levelList = res.data.Data
      this.chapterId = res.data.Data[0].ChapterId
      this.chapterName = res.data.Data[0].ChapterName
    },
    async getList() {
      var obj = {
        pageIndex: this.pageIndex,
        pageSize: this.pageSize,
        studentId: this.StudentId,
        chapterId: this.chapterId,
      }
      const res = await this.$uwonhttp.post('/Customer/CustomExercise/GetStudentCustomExerciseDetailList', obj)
      this.list = res.data.Data.Items

      this.totalNum = res.data.Data.TotalItems
      if (this.list) {
        this.list.forEach((val) => {
          if (val.Photo) {
            val.Photo = val.Photo.split(',')
          }
        })
        this.imgArr = this.list.filter((val) => {
          return val.ExerciseState == '4'
        })
        console.log(this.imgArr, 'aaaaaaaaaaaaaaa')
      }
    },
    async getInfoDatas() {
      var obj = {
        studentId: this.StudentId,
        chapterId: this.chapterId,
      }

      //获取学生练习单元批改数据

      const res = await this.$uwonhttp.post('/Customer/CustomExercise/GetStudentCustomExercise', obj)
      if (this.chapterId) {
        this.infoData = res.data.Data
        if (this.infoData === null) {
          this.infoData = {
            CustomExerciseCount: 0,
            StudentCorrectCount: 0,
            AVGDoPaperCorrect: 0,
          }
        }
      } else {
        this.allInfoData = res.data.Data
        if (this.allInfoData === null) {
          this.allInfoData = {
            CustomExerciseCount: 0,
            StudentCorrectCount: 0,
            AVGDoPaperCorrect: 0,
          }
        }
      }
    },

    async getLevelList() {
      this.isLoading = true
      await this.getInfoDatas()
      await this.getLevelDatas()
      await this.getList()
      await this.getInfoDatas()
      this.isLoading = false
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
      justify-content: flex-start;
      &.first {
        margin: 15px auto;
      }
      // &.next {
      //   margin: 0 auto 15px;
      // }
      // > button {
      //   margin-left: 16px;
      // }
      // .label {
      //   font-size: 14px;
      //   color: #4d5753;
      //   width: 84px;
      //   text-align: right;
      //   // justify-self: flex-end;
      // }
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
            width: 100%;
            max-height: 222px;
            overflow: hidden;
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
            height: 35%;
            width: 100%;
            // flex:1;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;

            /*          .name {
              font-size: 15px;
            border-bottom: #68BB97 1px solid;
            // line-height: 2;
            padding-bottom: 3px;
              color: #68BB97;
              &.gray{
                border-bottom: transparent 1px solid;
              color: #525252;
              }
            } */
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
            flex-direction: column;
            // margin-right:-10px;
            .tools {
              width: 50%;
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

            .btn {
              // width: 90%;
              display: inline-block;
              margin-top: 16px;
              background: #68bb97;
              padding: 0 9px;
              border-radius: 11.4px;
              font-size: 12px;
              color: #fff;
              cursor: pointer;
              text-align: center;
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
        }
      }
      .evaluate-info {
        // display: flex;
        // flex-direction: row;
        // justify-content: space-between;
        cursor: pointer;
        line-height: 1.7;
        border-bottom: 1px solid #eee;

        color: #565656;
        align-items: center;

        padding: 10px 0 4px;
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
          justify-content: space-around;
          align-items: flex-end;
          font-size: 14px;

          color: #6b6a6a;
          line-height: 1.8;
          height: 100%;
          padding-top: 10px;
          .eval {
            font-size: 14px;

            color: #68bb97;
          }
          p {
            text-align: right;
            font-size: 13px;
            span {
              font-size: 12px;
            }
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
