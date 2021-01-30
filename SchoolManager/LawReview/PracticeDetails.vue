<template>
  <div class="wrapper">

    <a-row :gutter="{ xl: 16, xxl: 16 }" type="flex" justify="start" align="middle" class="list" >
          <a-col :xl="12" :xxl="12"    v-if="objs.TrueName">
             <div class="item">
        <div class="head"><i class="icon_tit"></i>老师练习布置详情</div>
        <div class="containers">
          <div class="title-info">
            <span>发布老师：{{ objs.TrueName }}</span> <span> 发布班级：{{ objs.ClassName }}</span>
          </div>

          <h3>练习内容</h3>
          <div class="topic-info">
            <div class="photo-info" @click="blow(objs.PaperImgUrls)">
              <!-- <img src="@/assets/schoolmamager/lowreview/default.png" alt /> -->
              <img v-if="objs.PaperImgUrls[0]" :src="objs.PaperImgUrls[0]" alt />

              <div class="blow"></div>
            </div>
            <div class="student-info">
              <div class="desc-info">
                <div class="title-infos">
                  <h4>单元小节：</h4>
                  <div class="tit-box" :title="objs.ChapterName">{{ objs.ChapterName }}</div>
                  <div class="tit-box" :title="objs.FirstChapterName">{{ objs.FirstChapterName }}</div>
                </div>
                <div class="text">{{ objs.ExerciseName }}</div>
                <div class="gray text">{{ objs.ExerciseContent }}</div>
                <div class="voice-info" v-if="objs.AudioUrl">
                  <div class="voicepacket">
                   
                    <div
                      class="play-adiuo"
                      ref="playAdiuo"
                      @click="playAdiuoed"
                      :class="{ active: activePlay === '1' }"
                    >
                      <audio ref="mp3" @ended="overAudio" :src="objs.AudioUrl">您的浏览器不支持 audio 元素。</audio>
                      <span>{{ playTime[0] }}"</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="bottom">
            <span>发布时间：{{ objs.PublishTime }}</span>
            <span> 截止时间：{{ objs.EndTime }} </span>
          </div>
        </div>
     </div>
           
          </a-col>
      <a-col  :xl="12" :xxl="8" class="item" v-else>
        <a-empty />
      </a-col>
    </a-row>
<!--  -->

    <transition
      enter-active-class="animate__animated animate__fadeIn"
      leave-active-class="animate__animated animate__fadeOut"
    >
      <Blow v-if="isblow">
        <template v-slot:close>
          <div class="close" @click.stop="isblow = false">
            <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
          </div>
        </template>
        <template v-slot:img>
          <img v-for="(item, index) in objs.PaperImgUrls" :key="index" :src="item" alt />
        </template>
      </Blow>
    </transition>

    <div class="example" v-if="isLoading">
      <a-spin />
    </div>
  </div>
</template>

<script>
import Blow from './components/Blow.vue'
export default {
  data() {
    return {
      isblow: false,
      activePlay: '',
      playTime: '',
      // list: [],
      //学生id
      StudentId: '',
      //练习id
      ExerciseId: '',
      ClassId: '',
      UserId: '',
      isLoading: false,
      objs: {},
    }
  },
  created() {},
  activated() {
    this.init()
    this.getDatas()
  },
  methods: {
    init() {
      this.ClassId = this.$route.query.ClassId
      this.UserId = this.$route.query.UserId
      this.ExerciseId = this.$route.query.ExerciseId
      this.StudentId = this.$route.query.StudentId
    },
    playAdiuoed() {
      console.log('播放')
      // debugger
      this.activePlay = '1'
      const audio = this.$refs.mp3
      audio.play()
    },

    blow() {
      this.isblow = true
    },
    emClose() {
      this.isblow = false
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
    async getDatas() {
      var req = {
        userId: this.UserId,
        studentId: this.StudentId,
        exerciseId: this.ExerciseId,
      }
      //获取学生练习列表
      // /Customer/CustomExercise/GetCustomExerciseCorrectDetail
      this.isLoading = true
      const res = await this.$uwonhttp.post('/Customer/CustomExercise/GetCustomExerciseCorrectDetail', req)
      this.isLoading = false
      this.objs = res.data.Data
      if (this.objs.PaperImgUrls) {
        this.objs.PaperImgUrls = this.objs.PaperImgUrls.split(',')
      }
      if (this.objs.AudioUrl) {
        this.objs.AudioUrl = this.objs.AudioUrl.replace('[', '')
        this.objs.AudioUrl = this.objs.AudioUrl.replace(']', '')
        setTimeout(() => {
          this.playTime = Math.round(this.$refs.mp3.duration)
        }, 300)
      }
      console.log(this.objs.AudioUrl)
    },
  },

  components: {
    Blow,
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
  .list {
    margin-top: 16px;
    margin-bottom: 16px;
    // display: flex;
    // align-items: center;
    // justify-content: flex-start;
    // flex-direction: row;
    // flex-wrap: wrap;
    .item {
      // width: 32%;
      // height: 440px;
      padding-bottom: 16px;
      margin: 16px 0 0;
      // padding: 15px 10px 10px;
      background: #f4f4f4;
      border-radius: 4px;
      display: flex;
      flex-direction: column;
      margin-right: 2%;
      font-size: 14px;

      color: #2d2d2d;

      // &:nth-child(3n) {
      //   margin-right: 0;
      // }
      &:first-child {
        background: #fff;
      }
      .head {
        // line-height: 134px;
        width: 100%;
        background-color: #dfeae5;
        padding: 0 16px;
        line-height: 50px;
        border-radius: 4px 4px 0px 0px;
        font-size: 18px;
        color: #2d2d2d;
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
      }
      .containers {
        padding: 9px;
        .title-info {
          line-height: 2;
          border-bottom: 1px solid #eee;

          text-indent: 0.5em;
          padding: 12px 0 4px;
          span {
            margin-right: 40px;
          }
        }
        h3,
        .bottom {
          text-indent: 0.5em;
          line-height: 2.5;
          margin-top: 12px;
        }
        .bottom {
          span {
            margin-right: 40px;
          }
        }

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
            cursor: pointer;
            display: flex;
            flex-direction: row;
            justify-content: flex-start;
            align-items: center;
            width: 167px;
            img {
              width: 100%;
              height: 222px;
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

            .desc-info {
              display: flex;
              width: 100%;
                  padding-left: 20px;
              flex-direction: column;
              justify-content: flex-start;
              align-items: self-start;
              .title-infos {
                display: flex;
                flex-direction: row;
                justify-content: flex-start;
                align-items: center;

                .tit-box {
                  // font-size: 14px;
                  border-radius: 2px;
                  border: 1px solid #dfdfdf;
                  padding: 1px 6px;
                  color: #a6a6a6;
                  margin-right: 16px;
                  width: 128px;
                  overflow: hidden;
                  text-overflow: ellipsis;
                  white-space: nowrap;
                }
              }
              .text {
                line-height: 1.8;
                margin-top: 10px;
              }
              .gray {
                color: #9f9f9f;
              }

              //语音部分
              .voice-info {
                margin: 15px 0;
                text-align: left;
                line-height: 39px;
                height: 39px;
                vertical-align: middle;

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
                  // margin-left: 14px;
                  width: 201px;
                  height: 39px;
                  line-height: 39px;
                  margin-top: 15px;
                  margin-bottom: 15px;
                  text-align: center;
                  background: url('../../../assets/correcting/录音show.png') no-repeat center bottom;
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
                  &.active {
                    animation: soundShow 1.6s linear infinite;
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
</style>
