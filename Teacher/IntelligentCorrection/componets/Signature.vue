<template>
  <div class="mask-layer">
    <div class="canvas-info">
      <canvas id="signature" width="700" height="350"></canvas>
    </div>

    <div class="button-info">
      <div class="btn gray" @click="overwrite()">取消</div>
      <div class="btn" @click="confirm()">完成</div>
    </div>
  </div>
</template>

<script>
import { fabric } from 'fabric'
export default {
  components: {
    fabric,
  },

  data() {
    return {
      canvas: null,
    }
  },
  mounted() {
    // let canvas = this.$refs.canvasF
    // // canvas.height = this.$refs.canvasHW.offsetHeight - 100;
    // // canvas.width = this.$refs.canvasHW.offsetWidth - 10;
    // this.canvasTxt = canvas.getContext('2d')
    // this.canvasTxt.strokeStyle = this.color
    // this.canvasTxt.lineWidth = this.linewidth
    this.initCanvas()
  },
  methods: {
    /* 
    //电脑设备事件
    mouseDown(ev) {
      ev = ev || event
      //   ev.preventDefault();
      let obj = {
        x: ev.offsetX,
        y: ev.offsetY,
      }
      console.log(obj, 'offset坐标')
      this.startX = obj.x
      this.startY = obj.y
      this.canvasTxt.beginPath() //开始作画
      this.points.push(obj) //记录点
      this.isDown = true
    },
    //电脑设备事件
    mouseMove(ev) {
      ev = ev || event
      //   ev.preventDefault();
      if (this.isDown) {
        let obj = {
          x: ev.offsetX,
          y: ev.offsetY,
        }
        // console.log(obj,'offset坐标')
        this.moveY = obj.y
        this.moveX = obj.x
        this.canvasTxt.moveTo(this.startX, this.startY) //移动画笔
        this.canvasTxt.lineTo(obj.x, obj.y) //创建线条
        this.canvasTxt.stroke() //画线
        this.startY = obj.y
        this.startX = obj.x
        this.points.push(obj) //记录点
      }
    },
    //电脑设备事件
    mouseUp(ev) {
      ev = ev || event
      //   ev.preventDefault();
      if (1) {
        let obj = {
          x: ev.offsetX,
          y: ev.offsetY,
        }
        this.canvasTxt.closePath() //收笔
        this.points.push(obj) //记录点
        this.points.push({ x: -1, y: -1 })
        this.isDown = false
      }
    },*/
    //重写
    overwrite() {
      /*       this.canvasTxt.clearRect(0, 0, this.$refs.canvasF.width, this.$refs.canvasF.height)
      this.points = []
      this.isDraw = false //签名标记 */
      this.canvas.clear()
    },
    confirm() {
      //func: 是父组件指定的传数据绑定的函数，this.image:子组件给父组件传递的数据
      console.log(3433333333333)
      if (this.canvas.isEmpty()) {
        return
      }
      let img = this.canvas.toDataURL({ format: 'png' })
     
      this.$emit('getSignature', img)
    },
    initCanvas() {
      this.canvas = new fabric.Canvas('signature')
      this.canvas.isDrawingMode = true
      this.canvas.freeDrawingBrush.color = '#E34F51' // 设置自由绘颜色
      this.canvas.freeDrawingBrush.width = 2
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
  flex-direction: column;
  align-items: center;
  background: rgba(255, 255, 255, 0.6);
  .canvas-info {
    width: 700px;
    height: 350px;
    background: #fff;
    border: black 1px dashed;
    position: relative;
  }

  .button-info {
    margin: 20px auto 0;
    display: flex;
    justify-content: center;
    align-items: center;
    .btn {
      background: #0ab7ef;
      margin: 0 15px;
      color: #fff;
      text-align: center;
      padding: 0 2em;
      border-radius: 1em;
      font-size: 16px;
      line-height: 2;
      cursor: pointer;
      &.gray {
        background: #b5b5b5;
      }
    }
  }
}
</style>
