<template>
  <div>
    <div class="mask-layer">
      <div class="container">
        <!-- <div class="close" @click.stop.once="close()">
          <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
        </div> -->
         <slot name="close">关闭弹窗插槽</slot>

        <!-- <div class="title">
            01 王义玲
        </div>
    -->
        <div class="img-container">
         <slot name="img">图片内容插槽</slot>
         
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {}
  },
  methods: {
/*     close() {
      this.$emit('emClose')
    }, */
  },
  components: {},
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
    width: 574px;
    height: 763px;

    background: #fff;
    border-radius: 5px;
    padding: 0;
    position: relative;

    .img-container {
      width: 100%;
      height: 100%;
      min-height: 100%;
      overflow-y: auto;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      img {
        width: 100%;
      }
    }
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
