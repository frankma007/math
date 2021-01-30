<template>
  <div>
    <!-- 学生提问详情 -->
    <div class="clearfix student-info-dea">
      <a-avatar size="large" icon="user" class="fl"/>
      <div class="student-data">
        <p>王小清</p>
        <p class="line">一（1）班</p>
        <p><span>昨天</span><span>15:26</span></p>
      </div>
      <p class="answer"><span>这道题我看了解析，我还是不明白怎么弄，例如xxxxxxxxxxxxxxxxxxxxxxxxxx…</span></p>
      <p>1, 小朋友排队买票，小军从前边数排第3，后边有5个人，这一队共有几人？（    ）</p>
      <p><span>A  7</span><span>B  8</span><span>C  9</span></p>
      <p class="chapter-icon"><img src="@/assets/finemicro/link.png" alt="">一.20以内加减法</p>
      <p>正确答案：<span style="color:#4D5753">C</span></p>
      <p>ta的答案：<span style="color:#FF682C">A</span></p>
      <p style="color:#68BB97">正确答案：<span>平均正确率95%</span></p>
    </div>
    <!-- 教师回答详情 -->
    <div class="teacher-details">
      <a-avatar size="large" icon="user" class="fl"/>
      <div class="student-data">
        <p>吴江花</p>
        <p class="line">高一小学</p>
      </div>
      <p class="input-content"><input type="text"></p>
      <img src="@/assets/coach/图片.png" alt=""><img src="@/assets/coach/录音.png" alt=""><img src="@/assets/coach/视频.png" alt="">
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {

    }
  }
}
</script>

<style lang="less" scoped>
  .student-info-dea {
    padding: 10px 30px;
    background-color: #fff;
    border-radius: 5px;
    span {
        margin-right: 10px;
    }
    .line {
      padding-bottom: 5px;
      border-bottom: 1px dotted #ccc;
    }
     .chapter-icon {
      font-size: 12px;
      img {
        width: 20px;
        margin-right: 8px;
      }
    }
  }
  .student-data {
    position: relative;
    p:nth-child(3) {
      position: absolute;
      top: 18px;
      right: 0;
    }
  }
  .answer {
    margin-bottom: 100px;
  }
  .teacher-details {
    margin-top: 20px;
    padding: 10px 30px;
    background-color: #fff;
  }
  .input-content {
    input {
      width: 800px;
      height: 43px;
      border: none;
      border: 1px solid #D6D6D6;
      border-radius: 5px;
    }
  }
</style>
