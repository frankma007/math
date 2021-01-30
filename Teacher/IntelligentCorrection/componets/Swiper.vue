<template>
  <div class="module-list-wrap">
    <div v-swiper:mySwiper="swiperOption">
      <div class="swiper-wrapper">
        <!-- right 正确 -->
        <div
          class="swiper-slide swiper-no-swiping"
          v-for="(item, index) in studentsList"
          :key="item.StudentId"
          @click="revierewTest(index, item.StudentId)"
          ref="slide"
          :class="{ active: isActive === index }"
        >
          <div class="img-container">
            <img :src="item.Photo" />
            <p>{{ item.StudentNo }} {{ item.StudentName }}</p>
          </div>
          <div class="bottom" v-show="item.compelete === false"></div>
          <div class="right-mark" v-show="item.compelete === true">
            <img src="@/assets/teacher/IntelligentCorrection/icon-right.png" alt />
          </div>
        </div>
      </div>
    </div>
    <div class="left-arrow">
      <div class="arrow-img" @click="prevPhoto">
        <i></i>
      </div>
    </div>
    <div class="right-arrow">
      <div class="arrow-img" @click="nextPhoto">
        <i></i>
      </div>
    </div>
  </div>
</template>

<script>
import { Swiper, SwiperSlide, directive } from 'vue-awesome-swiper'
import 'swiper/swiper-bundle.css'
export default {
  props: {
    studentsList: {
      type: Array,
    },
    /*     compeleteList: {
      type: Array,
    }, */
    // customExerciseId: {
    //   type: String,
    // }, //试卷id
  },
  created() {
    console.log(this.studentsList, 'swiper头像列表更新')
  },
  mounted() {
    // console.log(this.mySwiper)
    //初始化将第0个StudentId传入 获取canvas
    // console.log(this.customExerciseId)
    console.log(this.studentsList, 'aaaa')

    // if(this.studentsList!=null){

    // }
  },

  data() {
    return {
      // studentsList:null,

      isActive: 0,
      studentid: '', //学生编号
      swiperOption: {
        noSwiping: true,
        observer: true,
        observeParents: true,
        slidesPerView: 7,
        spaceBetween: 15,
        //  loop : true,
        direction: 'horizontal',
        // navigation: { prevEl: '.left-arrow', nextEl: '.right-arrow' },
      },
    }
  },
  methods: {
    /*     changeDate(){
      debugger
        this.$nextTick(() => {
          console.log('我进来了');
          this.studentsList[this.isActive].compelete=true
                
        })
    }, */
    prevPhoto() {
      console.log('左')
      console.log(this.isActive, '索引')

      if (this.isActive === 0) {
        return
      }
      // this.mySwiper.slidePrev()
      // this.isActive= this.isActive-1;
      //  this.revierewTest()
      //   this.mySwiper.snapIndex--;
      this.isActive--
      this.mySwiper.slideTo(this.isActive)
      this.revierewTest(this.isActive, this.studentsList[this.isActive])
    },
    nextPhoto() {
      // this.isActive++
      console.log(this.isActive, '索引')
      console.log('右')
      if (this.isActive === this.studentsList.length - 1) {
        return
      }

      // this.mySwiper.slideNext()
      // var index = $(this).index();
      this.isActive++
      this.mySwiper.slideTo(this.isActive)
      this.revierewTest(this.isActive, this.studentsList[this.isActive])
    },
    //查阅试卷
    revierewTest(index, StudentId) {
      // index 即是索引

      this.mySwiper.realIndex = index //?
      this.mySwiper.activeIndex = index //?
      this.mySwiper.slideTo(index)

      var list = this.$refs.slide
      // this.mySwiper.slides[index].className+=' swiper-slide-active'
      // var list = this.mySwiper.slides
      list.forEach((val, i) => {
        if (i === index) {
          val.className = 'swiper-slide  active'
        } else {
          val.className = 'swiper-slide'
        }
      })
      this.isActive = index
      this.getStudentTodoCorrectExercise(this.studentsList[index].StudentId)
    },
    // /Customer/CustomExercise/GetStudentTodoCorrectExercise
    getStudentTodoCorrectExercise(studentid) {
      /*      var obj = {
        studentid,
        customExerciseId: this.customExerciseId,
      }

      this.$uwonhttp.post('/Customer/CustomExercise/GetStudentTodoCorrectExercise', obj).then((res) => {
        // this.studentsList=res.data.Data
        // console.log(this.studentsList);
        //  this.studentsList[0].StudentId
        console.log(res.data.Data.PaperImgUrl.split(','), '突突突')
        const {PaperImgUrl,CorrectErrorCnt, CorrectRightCnt, CorrectRightRate, CorrectTotalCnt, Epilogue,StudentDoPaperInfoId,Group } = res.data.Data
        var data ={PaperImgUrl,CorrectErrorCnt, CorrectRightCnt, CorrectRightRate, CorrectTotalCnt, Epilogue,StudentDoPaperInfoId,Group}

        
      }) */
      this.$emit('eventGetCanvasInfo', studentid, this.isActive)
    },
  },

  components: {
    Swiper,
    SwiperSlide,
  },

  watch: {
    studentsList: {
      handler(val) {
        this.$nextTick(() => {
          this.studentsList = val
          //如果监听到studentsList的变动 则向父组件传值
          // console.log(2222222222222222222222);
          debugger
          this.getStudentTodoCorrectExercise(this.studentsList[this.isActive].StudentId)
        })
      },
      deep: true,
    },
    /*     compeleteList: {
      handler(val) {
        debugger
        this.$nextTick(() => {
          if (this.compeleteList != val) {
            this.compeleteList = val
          }
        })
      },
      deep: true,
      immediate: true,
    }, */
  },
}
</script>
<style lang="less" scoped>
.module-list-wrap {
  padding: 24px 80px 0;
  width: 100%;
  position: relative;
  .swiper-slide {
    cursor: pointer;

    width: 100px;
    // width: 80px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-around;
    box-sizing: border-box;

    .img-container {
      width: 100px;
      background: #e8e8e8;
      border-radius: 5px;
      padding: 8px 0 0;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      img {
        width: 40px;
        border-radius: 40px;
        padding-top: 8px;
        filter: grayscale(100%);
        filter: gray;
      }
      p {
        color: #b5b5b5;
        font-size: 12px;
        line-height: 2;
      }
    }
    .bottom {
      width: 76px;
      height: 4px;
      border-radius: 2px;
      background: transparent;
      visibility: hidden;
      margin: 10px auto 10px;
      overflow: hidden;
    }
    .right-mark {
      // display: none;
      width: 24px;
      height: 24px;
      background: transparent;
      visibility: visible;
      text-align: center;
      padding-bottom: 0;
      img {
        width: 100%;
      }
    }

    &:hover,
    // &.swiper-slide-active,
    &.active,
    &.right {
      .img-container {
        background: #ecf5f3;
        img {
          filter: none;
        }
        p {
          color: #373737;
        }
      }
    }
    &:hover,
    // &.swiper-slide-active,
    &.active {
      .bottom {
        background: #68bb97;
        visibility: visible;
      }
    }
    &.right {
      .bottom {
        display: none;
      }
      .right-mark {
        display: block;
      }
    }
  }
  .left-arrow,
  .right-arrow {
    position: absolute;
    display: flex;
    height: 142px;
    align-items: center;
    justify-content: center;
    top: 0;
    .arrow-img {
      &:hover {
        background: #68bb97;
      }
      width: 40px;
      height: 40px;
      cursor: pointer;
      border-radius: 50%;
      background: #dadada;
      display: flex;
      align-items: center;
      justify-content: center;
      i {
        background: url('../../../../assets/teacher/IntelligentCorrection/icon-arrow.png') no-repeat center center;
        width: 16px;
        height: 16px;
      }
    }
  }
  .left-arrow {
    left: 18px;
    .arrow-img {
      transform: rotate(180deg);
    }
  }
  .right-arrow {
    right: 18px;
  }
}
</style>
