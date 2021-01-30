<template>
  <div>
    <div class="mask-layer">
      <div class="container">
        <!-- <div class="close" @click.once="close()">
          <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
        </div> -->
        <slot name="close">关闭弹窗插槽</slot>

        <slot name="title"> 标题区 </slot>

        <slot name="table"> 表格区 </slot>
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
    width: 650px;
    // height: 550px;
    background: #fff;

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
        .yellow {
          color: #ffb718;
        }
      }
    }

    .table-info {
      width: 100%;
      // height: 400px;
      max-height: 400px;
      overflow-y: auto;
      padding: 0;
      margin-top: 14px;
      background: #fff;
      border-radius: 2px;
      text-align: center;
      /deep/.ant-table-thead {
        > tr {
          > th {
            line-height: 2;
            color: #fff;
            font-size: 14px;
            font-weight: bold;
            border-bottom: none;
            text-align: center;
            background: #68bb97;
            padding: 5px;
            height: 48px;
          }
        }
      }
      /deep/ .ant-table-tbody {
        > tr {
          > td {
            text-align: center;
            font-size: 14px;
            padding: 5px;
            background: #fff;
            height: 48px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            color: #595959;
            .yellow {
              color: #ff682c;
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
