<template>
  <div class="wrapper">
    <div class="head">
      <h2>教师列表</h2>
      <div class="search-info">
        <a-select default-value="" class="select" @change="a">
          <a-select-option value=""> 所有年级 </a-select-option>
          <a-select-option value="1"> 一年级 </a-select-option>
          <a-select-option value="2"> 二年级 </a-select-option>
          <a-select-option value="3"> 三年级 </a-select-option>
          <a-select-option value="4"> 四年级 </a-select-option>
          <a-select-option value="5"> 五年级 </a-select-option>
        </a-select>
        <a-input v-model="teacherName" placeholder="搜索老师姓名" class="text-field"> </a-input>

        <a-button class="button" @click="getTeacher()"> 搜索 </a-button>
        <a-button class="button" @click="reset()"> 重置 </a-button>
      </div>
      <div class="totals">共{{ totalNum }}人</div>
    </div>

      <a-row :gutter="{ xl: 16, xxl: 16 }" type="flex" justify="start" align="middle" class="list" v-if="teacherList">
          <a-col :xl="8" :xxl="5"   v-for="(item, index) in teacherList" :key="index">
      <div class="item">

    
        <div class="teacher-info">
          <div class="photo">
            <img v-show="!item.Photo" src="@/assets/schoolmamager/lowreview/photo.png" alt />
            <img v-show="!!item.Photo" :src="item.Photo" alt />
          </div>
          <div class="text-info">
            <h3>{{ item.TrueName }}</h3>
            <div class="infos">
              <div class="infos-item">
                <p>练习发布总份数</p>
                <div class="nums">{{ item.CustomExCount }}</div>
              </div>
              <div class="infos-item">
                <p>已完成批改总数</p>
                <div class="nums">{{ item.EcetcCount }}</div>
              </div>
            </div>
          </div>
        </div>
        <div class="review-info">
          <h3>查看批改</h3>
          <div class="btn-box">
            <div
              class="btn"
              v-for="itm in item.ClassInfo"
              :key="itm.ClassId"
              :title="itm.className"
              @click.stop="toPractice(item.UserId, itm.ClassId)"
            >
              {{ itm.className }}
            </div>           
          </div>
        </div>
       </div>
          </a-col>
      </a-row>
<!--     <ul class="list" v-if="teacherList">
      <li class="item" v-for="(item, index) in teacherList" :key="index">
        <div class="teacher-info">
          <div class="photo">
            <img v-show="!item.Photo" src="@/assets/schoolmamager/lowreview/photo.png" alt />
            <img v-show="!!item.Photo" :src="item.Photo" alt />
          </div>
          <div class="text-info">
            <h3>{{ item.TrueName }}</h3>
            <div class="infos">
              <div class="infos-item">
                <p>练习发布总份数</p>
                <div class="nums">{{ item.CustomExCount }}</div>
              </div>
              <div class="infos-item">
                <p>已完成批改总数</p>
                <div class="nums">{{ item.EcetcCount }}</div>
              </div>
            </div>
          </div>
        </div>
        <div class="review-info">
          <h3>查看批改</h3>
          <div class="btn-box">
            <div
              class="btn"
              v-for="itm in item.ClassInfo"
              :key="itm.ClassId"
              :title="itm.className"
              @click.stop="toPractice(item.UserId, itm.ClassId)"
            >
              {{ itm.className }}
            </div>           
          </div>
        </div>
      </li>
    </ul> -->
    <div class="empty-mod" v-else>
      <a-empty />
    </div>

    <a-pagination
      :hideOnSinglePage="true"
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
// import {mapState} from 'vuex'
export default {
  data() {
    return {
      // 老师名称
      teacherName: '',
      // 班级
      gradeId: '',
      teacherList: [],
      //当前总数数
      totalNum: 0,
      //当前页
      pageIndex: 1,
      //当前页数
      pageSize: 15,
      isLoading: false,
    }
  },
  methods: {
    toPractice(UserId, ClassId) {
      const cc = {
        UserId: UserId,
        ClassId: ClassId,
      }
      this.$router.push({
        path: `/SchoolManager/LawReview/ClassPractice`,
        query: cc,
      })
    },
    changePage(num) {
      this.pageIndex = num
      this.getTeacher()
    },
    a(e) {
      console.log(e)
      this.gradeId = e
    },
    reset() {
      this.teacherName = ''
      this.gradeId = ''
    },
    async getTeacher() {
      this.isLoading = true
      var obj = {
        pageIndex: this.pageIndex,
        pageSize: this.pageSize,
        userId: localStorage.UserId,
        gradeId: this.gradeId,
        // gradeId: 1,
        keyWords: this.teacherName,
      }
      const res = await this.$uwonhttp.post('/Customer/CustomExercise/GetTeacherTeamList', obj)
      this.isLoading = false
      this.teacherList = res.data.Data.Items
      this.totalNum = res.data.Data.TotalItems
    },
  },
  components: {
   
    // ...mapGetters(['LawReview'])
    /*      ...mapState({
      'teacherList':(state)=>{
        console.log(state,"djflkasjflkjdsa");
        debugger
        return state.LawReview.teacherList

      }
    }) */
    //  ...mapGetters([
    //       ])
    // ...mapState,

  },
  mounted() {
    // this.haha()
    this.getTeacher()
    console.log(this.LawReview);
    console.log(this.app);
    debugger
    // debugger
    /*  console.log(this.teacherList,"33333333") */
  },
  watch: {
    // teacherList
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
    // height: 134px;
    display: flex;
    align-items: flex-start;
    justify-content: space-around;
    flex-direction: column;
    padding: 10px 10px 10px 16px;
    //  overflow: hidden;
    border-radius: 8px;
    h2 {
      font-size: 18px;
      font-weight: bold;
      color: #4a4a4a;
      line-height: 2;
    }
    .search-info {
      margin: 15px auto 20px;
      font-size: 14px;
      height: 40px;
      line-height: 40px;
      width: 100%;
      display: flex;
      align-items: flex-start;
      justify-content: flex-start;
      > div,
      > input,
      > button {
        margin-right: 16px;
      }

      .select {
        width: 184px;
        height: 100%;
        border-radius: 20px;
        /deep/.ant-select-selection {
          border-radius: 20px;
          height: 40px;
          line-height: 40px;
          /deep/.ant-select-selection__rendered {
            line-height: 40px;
          }
        }
      }
      .text-field {
        width: 205px;
        height: 100%;
        border-radius: 20px;
      }
      .button {
        cursor: pointer;
        display: inline-block;
        width: 124px;
        height: 100%;

        border-radius: 20px;
        color: white;
        background-color: #6bbd9a;
        text-align: center;
        &:hover,
        &:focus {
          border-color: #6bbd9a;
        }
      }
      .button:nth-of-type(2) {
        background-color: #a2c240;
        &:hover,
        &:focus {
          border-color: #a2c240;
        }
      }
    }
    .totals {
      color: #737976;
      line-height: 2;
    }
  }
  .list {
    margin-top: 12px;
/*     display: flex;
    align-items: center;
    justify-content: flex-start;
    flex-direction: row;
    flex-wrap: wrap; */
    .item {

      // width: 18%;
      height: 220px;
      margin: 12px 0;
      padding: 10px 0;
      background: #fff;
      border-radius: 5px;
      display: flex;
      flex-direction: column;
      // margin-right: 2.5%;
/*       &:nth-child(5n) {
        margin-right: 0;
      } */
      .teacher-info {
        display: flex;
        flex-direction: row;

        border-bottom: 1px solid #f6f6f6;

        height: 100px;
        .photo {
          display: flex;
          justify-content: center;
          align-items: center;
          width: 82px;

          img {
            width: 50px;
            border-radius: 50%;
          }
        }
        .text-info {
          // width: 217px;
          flex: 1;
          display: flex;
          flex-direction: column;
          align-items: flex-start;
          justify-content: center;

          // justify-items: ;
          h3 {
            font-size: 16px;
            color: #4d5753;
          }
          .infos {
            width: 100%;
            display: flex;
            flex-direction: row;
            align-items: flex-start;
            justify-content: center;

            .infos-item {
              display: inline-block;
              // padding-right: 15px;
              width:50%;
              p {
                font-size: 12px;
                color: rgba(155, 155, 155, 0.85);
              }
              .nums {
                font-size: 16px;
                color: #4d5753;
              }
            }
          }
        }
      }
      .review-info {
        // height: 82px;
        padding: 8px;
        // height: 82px;
        display: flex;
        align-items: flex-start;
        justify-content: space-around;
        flex-direction: column;
        width: 100%;
        h3 {
          font-size: 14px;
          margin-top: 2px;
          color: #4d5753;
          line-height: 1.8;
        }
        .btn-box {
          width: 100%;
          display: flex;
          align-items: center;
          justify-content: flex-start;
          flex-wrap: wrap;

          .btn {
            cursor: pointer;
            width: 24%;
            margin-top: 5px;
            margin-right: 0.75%;
            font-size: 12px;
            overflow: hidden;

            text-overflow: ellipsis;

            white-space: nowrap; //文本不换行，这样超出一行的部分被截取，显示...
            color: #97b4a7;
            // width: 62px;
            // height: 18px;
            line-height: 18px;
            background: #dfeae5;
            border-radius: 2px;
            text-align: center;
            padding: 2px 12px;
            &:nth-child(4n) {
              margin-right: 0;
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

@media (min-width: 1600px){
.ant-col-xxl-5 {

    width: 20%;
}
}


</style>
