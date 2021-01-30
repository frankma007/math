<template>
  <div>
    <div class="mask-layer">
      <div class="container">
        <div class="close" @click.once="close()">
          <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
        </div>
        <h4>点评“{{ StudentName }}”的练习</h4>
        <div class="statistics">
          <span class="big">统计</span>
          <span>共智能批改{{ exerciseData.CorrectTotalCnt }}题</span>
          <span
            >正确<i class="ok">{{ exerciseData.CorrectRightCnt }}</i
            >题</span
          >
          <span
            >错误<i class="error">{{ exerciseData.CorrectErrorCnt }}</i
            >题</span
          >
          <span>正确率：{{ exerciseData.CorrectRightRate }}%</span>
        </div>
        <div class="grade-info">
          <div class="grade-navigate">
            <span class="big">等第评价</span>
            <a-radio-group name="radioGroup"  v-model="exerciseData.Epilogue">
              <a-radio value="优秀" >优秀</a-radio>
              <a-radio value="良好">良好</a-radio>
              <a-radio value="合格">合格</a-radio>
              <a-radio value="须努力">须努力</a-radio>
            </a-radio-group>
          </div>
          <div class="area-info">
            <textarea class="area" rows="5" v-model="title"></textarea>
            <span class="strNum">{{ title.length }}/{{ titleMaxLength }}</span>
          </div>

          <div class="voice-info" v-show="voicePath === ''">
            <div class="voice-icon">
              <img @click="openVoice()" src="@/assets/teacher/IntelligentCorrection/voiceTube.png" alt />
            </div>
          </div>
          <div class="voice-info" v-show="voicePath !== ''">
            <div class="voicepacket">
              <!-- <img src="@/assets/teacher/IntelligentCorrection/packet.png" alt /> -->
              <div class="play-adiuo" ref="playAdiuo" @click="playAdiuoed" :class="{ active: activePlay === '1' }">
                <audio ref="mp3" @ended="overAudio" :src="voicePath">您的浏览器不支持 audio 元素。</audio>
                <span>{{ playTime }}"</span>
              </div>
              <span @click="removeAudio()">删除</span>
            </div>
          </div>

          <div class="grade-navigate">
            <span class="normal">是否打回订正</span>
            <a-radio-group name="radioGroup" :default-value="Number(NeedCorrect)" v-model="NeedCorrect">
              <a-radio :value="1">否</a-radio>
              <a-radio :value="2">是</a-radio>
            </a-radio-group>
          </div>
          <div class="btn-info" @click.stop.once="confirm()">提交评价</div>
        </div>
      </div>
    </div>
    <VoiceLibrary @getVoice="getVoice" v-if="isLibaray === true"></VoiceLibrary>
  </div>
</template>

<script>
import VoiceLibrary from './VoiceLibrary.vue'
export default {
  components: { VoiceLibrary },
  props: {
    exerciseData: {
      type: Object,
    },
    StudentName: {
      type: String,
    },
  },
  data() {
    return {
      NeedCorrect: 1,
      // Epilogue2: 1,
      activePlay: '',
      title: '',
      titleMaxLength: 100,
      // isVoice: false, //显示弹出
      isLibaray: false, //声音库弹窗展示
      voicePath: '',
      playTime: '',
    }
  },
/*   beforeMount() {
    console.log(this.exerciseData.Epilogue, 'sdfasdfasd')
    switch (this.exerciseData.Epilogue) {
      case '优秀':
        this.Epilogue2 = 1
        break
      case '良好':
        this.Epilogue2 = 2
        break
      case '合格':
        this.Epilogue2 = 3
        break
      case '需努力':
        this.Epilogue2 = 4
        break
      default:
        break
    }
    console.log(this.Epilogue2);

    // this.Epilogue2=this.Epilogue2.toString()
  }, */
  methods: {
    close(){
      this.$emit('evluateClose');
    },
    playAdiuoed() {
      console.log('播放')
      // debugger
      this.activePlay = '1'
      const audio = this.$refs.mp3
      audio.play()
    },
    overAudio() {
      this.activePlay = ''
    },
    removeAudio() {
      this.playTime = ''
      this.voicePath = ''
    },
    //从音频库里获取音频文件
    getVoice(voice, playTime) {
      // this.isLibaray=true //打开语音库
      console.log(voice, '音频文件')
      this.isLibaray = false //关闭音频库弹窗
      //将话筒图标隐藏 并展示 音频文件
      this.voicePath = voice
      this.playTime = playTime
      // this.isVoice=false
    },
    openVoice() {
      this.isLibaray = true //打开语音库
    },
    confirm() {
         const obj = {
        StudentDoPaperInfoId: this.exerciseData.StudentDoPaperInfoId,
        TeacherId: localStorage.UserId,
        Epilogue: this.exerciseData.Epilogue,
        Comment: this.title,
        AudioUrl: this.voicePath,
        NeedCorrect: this.NeedCorrect,
      }
      //提交评价信息
      this.$emit('emconfirmEvaluate',obj)
    },
    // SubmitComment() {

    //   // /Customer/CustomExercise/SubmitComment
    //   // this.$uwonhttp.post('/Customer/CustomExercise/SubmitComment', obj).then((res) => {})
    //    this.$emit('emCommitEvaluate', obj)
    // },
  },
  watch: {
    title() {
      if (this.title.length > this.titleMaxLength) {
        this.title = this.title.toString().slice(0, this.titleMaxLength)
      }
    },
    exerciseData: {},
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
  z-index: 9999;

  .container {
    font-size: 14px;
    width: 589px;
    background: #fff;
    border-radius: 5px;
    padding: 15px 0;
    position: relative;
    .close{
      width: 16px;
      position:absolute;
      right: -20px;
      top: -8px;
      cursor: pointer;
      img{
        width: 100%;
      }
    }

    // color: #3F3F3F;
    h4 {
      text-align: center;
      font-size: 16px;
      color: #383838;
      line-height: 2;
      padding-bottom: 2px;
      border-bottom: #ececec 1px solid;
      width: 100%;
    }
    .statistics {
      margin: 20px auto;
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: center;
      // text-align: center;
      height: 32px;

      color: #383838;
      .big {
        // font-size: 16px;
        // font-weight: bold;
        font: 16px bold;
      }
      span {
        .ok {
          color: #68bb97;
          font-style: normal;
        }
        .error {
          color: #f87175;
          font-style: normal;
        }
        margin: 0 0.5em;
        font-size: 14px;
      }
    }
    .grade-info {
      width: 80%;
      margin: 0 auto;

      .grade-navigate {
        margin: 15px auto;
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: start;
        text-align: left;
        height: 32px;
        color: #383838;
        .big {
          // font-size: 16px;
          // font-weight: bold;
          font: 16px bold;
          margin-right: 1em;
        }
        .normal {
          margin-right: 1em;
        }
        .ant-radio-group {
          font-size: 14px;
        }
      }
      .area-info {
        position: relative;

        .area {
          background: #f4f4f4;
          padding: 0.5em;
          font-size: 14px;
          line-height: 2;
          resize: none;
          width: 100%;
          border: none;
          border-radius: 4px;
        }
        .strNum {
          position: absolute;
          right: 7px;
          bottom: 7px;
        }
      }
      //语音部分
      .voice-info {
        margin: 15px 0;
        text-align: left;
        line-height: 39px;
        height: 39px;
        vertical-align: middle;
        .voice-icon {
          cursor: pointer;
          width: 20px;
          img {
            width: 100%;
          }
        }
        .voicepacket {
          height: 39px;
          line-height: 39px;
          display: flex;
          flex-direction: row;
          justify-content: start;
          align-items: center;
          cursor: pointer;
          img {
            height: 100%;
          }
          span {
            margin-left: 15px;
            color: #c1c1c1;
          }
        }

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
          span {
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
      }
      .btn-info {
        background: #68bb97;
        color: #fff;
        line-height: 3;
        margin: 0.5em auto;
        padding: 0 4em;
        text-align: center;
        display: inline-block;
        border-radius: 1.5em;
        cursor: pointer;
      }
    }
  }
}
</style>
