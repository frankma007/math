<template>
  <div class="mask-layer">
    <ul class="container">
      <li
        v-for="(item, index) in list"
        :key="index"
        @click="selectVoice(playTime[index], item.AudioUrl)"
      >
        {{ item.Name }}

        <!-- <img src="@/assets/teacher/IntelligentCorrection/packet.png" alt /> -->
        <!-- :class="{ active: activePlay === index }" -->
        <div class="play-adiuo">
          <audio ref="mp3" :src="item.AudioUrl">您的浏览器不支持 audio 元素。</audio>
          <span >{{ playTime[index] }}"</span>
        </div>

        <!-- <i class="ok" v-show="active == index"></i> -->
      </li>
      <!-- <li>良好 <img src="@/assets/teacher/IntelligentCorrection/packet.png" alt /> <i class="ok"></i></li>
      <li>合格 <img src="@/assets/teacher/IntelligentCorrection/packet.png" alt /> <i class="ok"></i></li>
      <li>需努力 <img src="@/assets/teacher/IntelligentCorrection/packet.png" alt /> <i class="ok"></i></li> -->
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      list: [],
      // active: '',
      activePlay: '-1',
      playTime: [],
    }
  },
  beforeMount() {},
  mounted() {
    this.GetCollectionAudios()
  },
  methods: {
    getDuration() {
      if (!Array.isArray(this.$refs.mp3)) {
        return
      }
      this.$refs.mp3.forEach((vals) => {
        //  this.playTime.push(vals.duration)
        // vals.play() // 每一次动态改变aduio的src后，都要load一次
        // vals.addEventListener('ended', ()=> {
           
        // })

         this.playTime.push(Math.round(vals.duration))

       
      })
    },

    // playAdiuoed(index) {
    //   console.log('播放')
    //   // debugger
    //   this.activePlay = index
    //   const audio = this.$refs.mp3[index]
    //   audio.play()
    //   console.log(parseInt(audio.duration))
    // },
    // overAudio() {
    //   this.activePlay = '-1'
    // },
    //返回父组件
    selectVoice(seconds,imgPath) {
      // this.active = index
      // console.log(11111)
      this.$emit('getVoice', imgPath,seconds)
    },
    // /Customer/CustomExercise/GetCollectionAudios
    //获取录音库里的语音
    GetCollectionAudios() {
      console.log(222222233333333333)
      //  localStorage.UserId
      const obj = {
        userid: localStorage.UserId,
      }
      this.$uwonhttp
        .post('/Customer/CustomExercise/GetCollectionAudios', obj)
        .then((res) => {
          console.log(res, '录音文件')
          //  this.studentsList[0].StudentId
          this.list = res.data.Data
        })
        .then(() => {
          setTimeout(() => {
            this.getDuration()
          }, 500)
        })
    },
  },
  computed: {
    obj() {
      const { list, $refs, playTime } = this
      return {
        list,
        $refs,
        playTime,
      }
    },
  },
  watch: {
    obj: {
      handler(val) {
        this.$nextTick(() => {
          this.list = val.list
          this.$refs = val.$refs
         
        })
      },
      deep: true,
    },
  },
}
</script>

<style lang="less" scoped>
.mask-layer {
  width: 100%;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
  display: flex;
  justify-content: center;
  flex-direction: row;
  flex-wrap: nowrap;
  align-items: center;
  background: rgba(0, 0, 0, 0.8);
  z-index: 99999;

  .container {
    font-size: 16px;
    color: #171717;
    width: 589px;
    background: #fff;
    border-radius: 5px;
    padding: 15px;
    li {
      display: flex;
      margin: 0 auto;
      width: 92%;
      justify-content: start;
      align-items: center;
      flex-direction: row;
      border-bottom: #ececec 1px solid;
      padding-top: 4px;
      padding-bottom: 3px;
      text-align: left;
      font-size: 16px;
      line-height: 2;
      text-indent: 1em;
      position: relative;
      

      // 音频
      .play-adiuo {
        margin-left: 14px;
        width: 201px;
        height: 39px;
        line-height: 39px;
        margin-top: 15px;
        margin-bottom: 15px;
        text-align: center;
        background: url('../../../../assets/correcting/录音show.png') no-repeat center bottom;
        background-size: 100% 100%;
        cursor: pointer;
        span{
          color: #fff;
        }
        // 语音
        .sound-show {
          width: 201px;
          height: 39px;
          line-height: 39px;
          margin-top: 15px;
          text-align: center;
          background: url('../../../../assets/correcting/录音show.png');
          cursor: pointer;
        }
        @keyframes soundShow {
          0% {
            background: url('../../../../assets/correcting/播放1.png');
          }
          25% {
            background: url('../../../../assets/correcting/播放2.png');
          }
          50% {
            background: url('../../../../assets/correcting/播放3.png');
          }
          100% {
            background: url('../../../../assets/correcting/播放3.png');
          }
        }
        &.active {
          animation: soundShow 1.6s linear infinite;
        }
      }

      i.ok {
        position: absolute;
        right: 0;
        bottom: 0;
        background: url('../../../../assets/teacher/IntelligentCorrection/icon-right.png') left top no-repeat;
        background-size: cover;
        width: 24px;
        height: 24px;
      }

      img {
        vertical-align: middle;
        height: 20px;
      }
      &:last-child {
        border-bottom: transparent 1px solid;
      }
    }
  }
}
</style>
