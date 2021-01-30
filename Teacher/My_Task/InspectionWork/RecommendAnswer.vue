<template>
  <div>
    <p class="title">推荐参考答案</p>
    <div >
      <p v-html="testQuestion.Title"></p>
      <p class="chapter-name">
        <img src="@/assets/finemicro/link.png" alt="">
        <span>说一说</span>
      </p>
    </div>
  </div>
</template>

<script>
export default {
  created () {
    this.getStem()
    this.getChapterName()
  },
  data () {
    return {
      paperId: '',
      // 章节名称
      paperChapterName: '',
      testQuestion: []
    }
  },
  methods: {
    // 获取单个题目的题干
    getStem () {
      this.$http.post('/Paper/Exam_Item/GetTheData', {
        id: '1309391907280916480'
      }).then(res => {
        console.log('题干信息', res)
        this.testQuestion = res.Data
      })
    },
    // 获取章节名称
    getChapterName () {
      this.$http.post('/Paper/Exam_Paper/GetTheData', {
        id: this.paperId
      }).then(res => {
        console.log('章节名称---', res)
        this.paperChapterName = res.Data.ChapterName
        console.log('试卷的基本信息---', this.paperChapterName)
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .title {
    font-size: 16px;
  }
  .chapter-name {
    img {
      width: 17px;
      margin-right: 15px;
    }
    span {
      padding: 0 15px;
      background-color: #ddd;
      border-radius: 5px;
    }
  }
</style>
