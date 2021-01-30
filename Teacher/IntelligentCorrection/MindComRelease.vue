<template>
  <div>
    <div class="mind-wait-info">
      <p class="mind-title"><img style="margin-right:6px;" src="@/assets/teacher/矩形 (3).png" alt="">智能批改练习详情</p>
      <p class="mind-sub">{{ aloneDetails.PaperName }}</p>
      <p class="mind-comment">{{ aloneDetails.Content }}</p>
      <div v-if="aloneDetails.AudioUrl !== null" class="play-adiuo" ref="playAdiuo" @click="playAdiuoed" :class="{ 'active': activePlay === '1' }">
        <audio ref="mp3" @ended="overAudio" :src="aloneDetails.AudioUrl">
          <!-- <source src="" type="audio/ogg"> -->
          <!-- <source src="horse.mp3" type="audio/mpeg"> -->
          <!-- <source src=""> -->
          您的浏览器不支持 audio 元素。
        </audio>
        <span style="color: #fff">{{ playTime }}</span>
      </div>
      <p class="mind-pic">
        <img
          v-for="(item,index) in picImgs"
          :key="index"
          class="pic-m-r"
          :src="item"
          alt=""
          v-image-preview="item">
        <!-- <img src="@/assets/teacher/file.png" alt=""> -->
      </p>
      <p>
        <span class="m-r">发布时间：{{ aloneDetails.PublishTimeStr }}</span>
        <span>截止时间：{{ aloneDetails.EndTimeStr }}</span>
      </p>
    </div>
    <div class="answer-situation">
      <p class="mind-title"><img style="margin-right:6px;" src="@/assets/teacher/矩形 (3).png" alt="">作答情况</p>
      <p>
        <!-- color-def -->
        <span class="class-btn" :class="{ 'color-def': defaultClassId === item.ClassId }" @click="handleClass(item.ClassId)" v-for="item in classInfo" :key="item.ClassId">{{ item.ClassName }}</span>
        <!-- <span class="class-btn color-def">二（1）班</span> -->
      </p>
      <div class="mit-info">
        <p class="mit-tit">已提交的待批改练习已智能批改完毕，请检查</p>
        <p>
          <span class="m-r-tit"><span class="f-s-c">{{ classData.FirstTodoCheckCnt }}</span>人<span class="m-l">首次待批改</span></span>
          <span class="m-r-tit"><span class="f-s-c">{{ classData.CorrectTodoCheckCnt }}</span>人<span class="m-l">订正待批改</span></span>
          <span @click="toCorrection(classData.ClassId)" class="cur">去批改<img src="@/assets/teacher/right-jt.png" alt=""></span>
        </p>
      </div>
      <div class="com-info">
        <span class="com-remind c-m-r">
          <span>
            <span class="f-s-c">{{ classData.CompletedCnt }}</span><span class="up-m-r">人</span>
            <span class="cur" @click="toComplete(classData.ClassId)">已完成 <img src="@/assets/teacher/right-jt.png" alt=""></span>
          </span>
        </span>
        <span class="com-remind">
          <span class="f-s-c">{{ classData.UnCompletedCnt }}</span><span class="up-m-r">人未做</span>
          <span @click="onekeyRemind(classData.ClassId)" class="onekey cur">一键提醒 </span><img src="@/assets/teacher/right-jt.png" alt="">
        </span>
      </div>
    </div>
  </div>

</template>

<script>
import Recorder from 'js-audio-recorder'
const recorder = new Recorder()
export default {
  created () {
    this.singleId = this.$route.query.id
    this.getMindComInfo()
  },
  mounted () {
    // setTimeout(() => {
    //   this.playSound()
    // }, 800)
  },
  watch: {
    $route (to, from) {
      // console.log('单题详情加载', to, from)
      this.singleId = this.$route.query.id
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/IntelligentCorrection/MindComRelease?' + ChangId) {
        this.getMindComInfo()
        // setTimeout(() => {
        //   this.playSound()
        // }, 800)
      }
    }
  },
  data () {
    return {
      // 试卷id
      singleId: '',
      // 默认语音
      activePlay: '',
      playTime: 1,
      // 单题详情
      aloneDetails: {},
      // 单题图片
      picImgs: [],
      // 班级信息
      classInfo: [],
      // 默认班级
      defaultClassId: '',
      // 班级的信息
      classData: {}
    }
  },
  methods: {
    getMindComInfo () {
      // console.log(this.singleId)
      this.$uwonhttp.post('/Customer/CustomExercise/GetCustomExerciseDetail', { customExerciseId: this.singleId }).then(res => {
        debugger
        console.log('单题详情', res)
        this.aloneDetails = res.data.Data
        console.log(res.data.Data.CustomExerciseDetailClass[0].ClassId)
        this.picImgs = res.data.Data.PaperImgUrls.split(',')
        this.defaultClassId = res.data.Data.CustomExerciseDetailClass[0].ClassId
        this.classInfo = res.data.Data.CustomExerciseDetailClass
        this.classData = res.data.Data.CustomExerciseDetailClass[0]
      }).then(() => {
        setTimeout(() => {
          this.playSound()
        }, 800)
      })
    },
    // 选择班级
    handleClass (classId) {
      this.defaultClassId = classId
      const arr = this.classInfo.filter(value => {
        return value.ClassId === classId
      })
      this.classData = arr[0]
      console.log(arr[0])
    },
    playSound () {
      console.log('11111')
      // debugger
      // console.log('语音', this.aloneDetails.AudioUrl.join(','))
      if (this.aloneDetails.AudioUrl !== null) {
        debugger
        const audio = this.$refs.mp3
        this.playTime = parseInt(audio.duration)
        console.log(parseInt(audio.duration))
      }
    },
    // 播放
    playAdiuoed () {
      console.log('播放')
      // debugger
      this.activePlay = '1'
      const audio = this.$refs.mp3
      audio.play()
      console.log(parseInt(audio.duration))
    },
    overAudio () {
      this.activePlay = ''
    },
    // 一键提醒
    onekeyRemind (classId) {
      this.$router.push({ path: '/Teacher/IntelligentCorrection/OnekeyRemind',
        query: {
          paperId: this.singleId,
          classId
        } })
    },
    // 完成
    toComplete (classId) {
      this.$router.push({ path: '/Teacher/IntelligentCorrection/CompletePractive',
        query: {
          classId,
          customId: this.singleId
        }
      })
    },
    // 去批改
    toCorrection (classId) {
      console.log('班级id', classId)
      if (this.classData.CorrectTodoCheckCnt === 0) {
        return
      }
      this.$router.push({ path: '/Teacher/IntelligentCorrection/IntelligentCorrection',
        query: {
          customId: this.singleId,
          classId,
          paperName: this.aloneDetails.PaperName
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .pic-m-r {
    width: 128px;
    height: 120px;
    margin-right: 19px;
  }
  .m-l {
    margin-left: 5px;
  }
  .m-r {
    margin-right: 58px;
  }
  .c-m-r {
    margin-right: 30px;
  }
  .f-s-c {
    font-size: 29px;
    color: #0AB7EF;
  }
  .up-m-r {
    margin-right: 15px;
  }
   // 语音
   .sound-show {
      width: 201px;
      height: 39px;
      line-height: 39px;
      margin-top: 15px;
      text-align: center;
      background: url('../../../assets/correcting/录音show.png');
      cursor: pointer;
    }
    @keyframes soundShow {
    0% {
      background: url('../../../assets/correcting/播放1.png');
    }
    25% {
      background: url('../../../assets/correcting/播放2.png');
    }
    50% {
      background: url('../../../assets/correcting/播放3.png');
    }
    100% {
      background: url('../../../assets/correcting/播放3.png');
    }
  }
  .active {
      animation: soundShow 1.6s linear infinite;
    }
   // 音频
  .play-adiuo {
      width: 201px;
      height: 39px;
      line-height: 39px;
      margin-top: 15px;
      margin-bottom: 15px;
      text-align: center;
      background: url('../../../assets/correcting/录音show.png') no-repeat center bottom;
      cursor: pointer;
  }
  // 一键提醒
  .onekey {
    padding: 2px 8px;
    color: #fff;
    background-color: #0AB7EF;
    border-radius: 15px;
  }
  // 最下面的信息
  .com-info {
    margin-top: 16px;
  }
  .com-remind {
    display: inline-block;
    width: 49%;
    height: 134px;
    line-height: 134px;
    text-align: center;
    background-color: #F1F5F6;
    border-radius: 5px;
  }
  .mind-wait-info {
    padding: 24px 24px 16px 24px;
    background-color: #fff;
    border-radius: 5px;
  }
  .mind-title {
    margin-bottom: 26px;
    font-size: 18px;
  }
  .mind-sub {
    margin-bottom: 16px;
    font-size: 16px;
  }
  .mind-comment {
    margin-bottom: 21px;
  }
  .mind-pic {
    margin-bottom: 23px;
  }
  // 作答情况
  .answer-situation {
    margin-top: 24px;
    padding: 24px 24px 16px 24px;
    background-color: #fff;
    border-radius: 5px;
    .class-btn {
      width: 100px;
      height: 40px;
      line-height: 40px;
      margin-top: 20px;
      margin-right: 35px;
      padding: 10px 15px;
      text-align: center;
      border-radius: 20px;
      color: #B5B5B5;
      // background-color: #F4F4F4;
      border: 1px solid #B5B5B5;
      cursor: pointer;
    }
    .color-def {
      border: none;
      color: #fff;
      background-color: #68BB97;
    }
    .mit-info {
      margin-top: 24px;
      padding: 24px 0 34px 24px;
      font-size: 12px;
      background-color: #F1F5F6;
      border-radius: 5px;
      .mit-tit {
        margin-bottom: 47px;
      }
      .m-r-tit {
        margin-right: 64px;
      }
    }
  }
</style>
