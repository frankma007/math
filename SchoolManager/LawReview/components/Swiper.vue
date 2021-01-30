<template>
  <div>
    <div class="mask-layer">
      <div class="container">
        <!-- <div class="close" @click.once="close()">
          <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
        </div> -->
         <slot name="close">关闭弹窗插槽</slot>
        <div class="swiper-container" v-swiper:mySwiper="swiperOption">
          <div class="swiper-wrapper">
             <slot name="imgList">图片内容插槽</slot>
<!--             <div class="swiper-slide">
              <div class="title">01 王义玲</div>
              <div class="img-box">
                <img src="@/assets/schoolmamager/lowreview/default.png" alt />
               
              </div>
            </div>
            <div class="swiper-slide">
              <div class="title">02 王义玲</div>
              <div class="img-box">
                <img src="@/assets/schoolmamager/lowreview/default.png" alt />
                <img src="@/assets/schoolmamager/lowreview/default.png" alt />
              </div>
            </div> -->
            <div class="swiper-pagination"></div>
          </div>
        </div>

        <!-- <div class="title">
            01 王义玲
        </div>
         <img src="@/assets/schoolmamager/lowreview/default.png" alt /> -->

        <div class="arrow-left" @click="prevPhoto">
          <i></i>
        </div>
        <div class="arrow-right" @click="nextPhoto">
          <i></i>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Swiper, SwiperSlide } from 'vue-awesome-swiper'
import 'swiper/swiper-bundle.css'
export default {
  props:{
list:Array,
imgActive:Number,
  },
  data() {
    return {
      swiperOption: {
        slidesPerView: 1,
        spaceBetween: 10,
        direction: 'horizontal',
        navigation: { prevEl: '.arrow-left', nextEl: '.arrow-right' },
      },
    }
  },
  mounted(){
this.mySwiper.slideTo(this.imgActive)
  },
  methods: {
    close() {
      this.$emit('emClose')
    },
    prevPhoto() {
      this.mySwiper.slidePrev()
    },
    nextPhoto() {
      this.mySwiper.slideNext()
    },
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
    height: 832px;
    background: #fff;
    border-radius: 5px;
    padding: 0;
    position: relative;
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

    .arrow-left,
    .arrow-right {
      cursor: pointer;
      position: absolute;
      display: flex;
      width: 71px;
      height: 832px;
      flex: 1;
      min-height: 0;
      align-items: center;
      justify-content: center;
      top: 0;
      i {
        background: url('../../../../assets/schoolmamager/lowreview/icon_arrow.png') no-repeat center center;
        width: 71px;
        height: 70px;
        background-size: 100% 100%;
        display: inline-block;
      }
    }
    .arrow-left {
      left: -111px;
      i {
        transform: rotate(180deg);
      }
    }
    .arrow-right {
      right: -111px;
    }

    /deep/ .swiper-wrapper {
      .swiper-slide {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        .title {
          font-size: 35px;
          color: #68bb97;
          line-height: 2;
          text-align: center;
          height: 70px;
        }
        .img-box {
          width: 100%;
          min-height: 763px;
          height: 763px;
          border-radius: 0 0 5px 5px;
          overflow-y: auto;
          img {
            width: 100%;
            //   height: 763px;

            //    overflow: hidden;
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
