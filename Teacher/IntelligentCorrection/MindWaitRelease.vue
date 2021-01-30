<template>
  <div >
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
          :preview="item">
      <!-- <img class="pic-m-r" src="@/assets/teacher/file.png" alt=""> -->
      </p>
      <p>
        <span class="m-r">预计发布时间：{{ aloneDetails.PublishTimeStr }}</span>
        <span>截止时间：{{ aloneDetails.EndTimeStr }}</span>
      </p>
    </div>
    <p class="operation">
      <span @click="hanleRevoke" class="handle-btn revoke cur">撤销</span>
      <span @click="handleAdjus" class="handle-btn adjustment cur">调整</span>
    </p>
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
  data () {
    return {
      playTime: 1, // 录音时间
      activePlay: '', // 播放
      singleId: '', // 试卷id
      aloneDetails: [], // 单题详情
      picImgs: [] // 图片
    }
  },
  methods: {
    getMindComInfo () {
      // console.log(this.singleId)
      this.$uwonhttp.post('/Customer/CustomExercise/GetCustomExerciseDetail', { customExerciseId: this.singleId }).then(res => {
        console.log('单题详情', res)
        this.aloneDetails = res.data.Data
        // console.log(res.data.Data.CustomExerciseDetailClass[0].ClassId)
        this.picImgs = res.data.Data.PaperImgUrls.split(',')
      })
    },
    playSound () {
      console.log('11111')
      // debugger
      if (this.aloneDetails.AudioUrl !== null) {
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
    // 撤销
    hanleRevoke () {
      this.$uwonhttp.post('/Customer/CustomExercise/cancel', { id: this.singleId }).then(res => {
        console.log('撤销', res)
        if (res.data.Success) {
          this.$message.success('撤销成功')
          this.$router.push({ path: '/Teacher/IntelligentCorrection/MindPractice' })
        }
      })
    },
    // 调整
    handleAdjus () {
      this.$router.push({ path: '/Teacher/My_Task/CustomPractice',
        query: {
          id: this.singleId,
          isAdjust: '1'
        } })
      // this.$uwonhttp.post('')
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
  .m-r {
    margin-right: 58px;
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
  // 操作按钮
  .handle-btn {
    display: inline-block;
    width: 103px;
    height: 39px;
    line-height: 39px;
    font-size: 16px;
    text-align: center;
    border-radius: 5px;
  }
  .operation {
    margin-top: 32px;
  }
  // 撤销
  .revoke {
    margin-right: 40px;
    color: #B5B5B5;
    border: 1px solid #B5B5B5;
  }
  // 调整
  .adjustment {
    color: #fff;
    background-color: #68BB97;
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
</style>
