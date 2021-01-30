<template>
  <div>
    <div class="mask-layer">
      <div class="container">
        <!-- <div class="close" @click.once="close()">
          <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
        </div> -->
        <slot name="close">关闭弹窗插槽</slot>
        <div class="title">
          <h6>知识点：两位数加一位数</h6>
        
        </div>
         <slot name="title"> 标题区 </slot>

        <slot name="list"> 列表区 </slot>


      </div>
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
     
    }
  },
  mounted() {},
  methods: {},
 
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
  background: rgba(0, 0, 0, 0.6);
  z-index: 9999;

  .container {
    font-size: 14px;
    width: 840px;
    height: 550px;

    background: #f2f2f2;

    padding: 24px;
    position: relative;
    border-radius: 8px;
    .close {
      width: 21px;
      position: absolute;
      right: -45px;
      top: 0;
      cursor: pointer;
      img {
        width: 100%;
      }
    }

    .title {
      h6 {
        font-size: 18px;
        color: #4d5753;
        text-align: center;
        line-height: 2;
      }
      .desc {
        color: #727272;
        font-size: 12px;
        line-height: 28px;
        .big {
          font-size: 14px;
          margin-right: 12px;
        }
        .green {
          color: #68bb97;
        }
        .error {
          color: #f87175;
        }
      }
    }

    .list {
      width: 100%;
      height: 400px;
      min-height: 400px;
      overflow-y: auto;
      margin-bottom: 24px;
      .item {
        background: #ffffff;
        border-radius: 8px;
        padding: 16px 16px 2px;
        margin-top: 24px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        position: relative;
        .state {
          position: absolute;
          right: 1em;
          top: 0.5em;

          font-size: 12px;

          color: #ff682c;
        }
        .right-mod {
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          .text {
            font-size: 14px;

            color: #4d5753;
            line-height: 14px;
          }
          .btn {
            margin-top: 40px;
            width: 90px;
            height: 36px;
            background: #a2c240;
            border-radius: 4px;
            font-size: 12px;

            color: #ffffff;
            text-align: center;
            line-height: 36px;
            cursor: pointer;
          }
        }
        .content-mod {
          display: flex;
          flex-direction: column;
          align-items: flex-start;
          justify-content: center;
          .top-info {
            line-height: 32px;
            font-size: 18px;
            color: #49534e;

            .btn {
              color: #fff;
              background: #d6d6d6;
              width: 64px;
              text-align: center;
              line-height: 2;
              font-size: 12px;
              border-radius: 12px;
              display: inline-block;
            }
          }
          .bottom-info {
            font-size: 12px;

            color: #d6d6d6;
            line-height: 2;
          }
          .content {
            display: flex;
            align-items: center;
            justify-content: flex-start;
            .info {
              width: 200px;

              font-size: 14px;

              color: #737976;
              line-height: 34px;
              span {
                margin: 0 5px 0 12px;
              }
              img {
                width: 16px;
              }
            }
            .rate {
              width: 132px;
              margin: 0 24px;
            }
            .Progress {
              flex: 1;
              display: flex;
              justify-content: center;
              align-items: center;
              /*               .ant-progress-circle-wrap,
/deep/ .ant-progress-line-wrap {
  margin-right: 8px;
  margin-bottom: 0px;
} */
              .items {
                text-align: center;
                margin-right: 16px;
                display: flex;
                flex-direction: column;
                align-items: center;
                justify-content: center;
                .gray {
                  color: #d6d6d5;
                }
                p {
                  margin-top: 8px;
                }
                /deep/ .ant-progress {
                  box-sizing: border-box;
                  margin: 0;
                  padding: 0;
                  color: #479aeb;
                  font-size: 0.072917rem;
                  font-variant: tabular-nums;
                  line-height: 1.5;
                  list-style: none;
                  font-feature-settings: 'tnum';
                  display: inline-block;
                  margin-right: 5px;

                  &.ant-progress-status-normal {
                    .ant-progress-text {
                      font-size: 12px;
                      color: #479aeb;
                    }
                  }
                  .ant-progress-text {
                    font-size: 12px;
                    // color:#479AEB;
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

::-webkit-scrollbar {
  /*滚动条整体样式*/

  width: 10px; /*高宽分别对应横竖滚动条的尺寸*/

  height: 1px;
}

::-webkit-scrollbar-thumb {
  /*滚动条里面小方块*/

  border-radius: 10px;

  -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);

  background: #535353;
}

::-webkit-scrollbar-track {
  /*滚动条里面轨道*/
  -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  border-radius: 10px;

  background: #ededed;
}
</style>
