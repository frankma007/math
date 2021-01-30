<template>
  <div>
    <p class="my-back"><img @click="returnGo" class="cur" src="@/assets/teacher/左边箭头.png" alt=""><span class="my-back-name">我的反馈:</span></p>
    <div v-if="this.myBack.length === 0" style="text-align:center">
      <img src="@/assets/lack/暂无搜索记录.png" alt="">
      <p style="color:#ccc;">暂无我的反馈</p>
    </div>
    <div class="back-list" v-for="(item, index) in myBack" :key="index">
      <p class="back-info">{{ item.FaultContent }}</p>
      <p>{{ item.CreateTime }}</p>
    </div>
    <!-- 分页 -->
    <div class="paging">
      <a-pagination :default-current="pageNo" :total="totalNum" @change="onPaging" hideOnSinglePage/>
    </div>
  </div>
</template>

<script>
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    this.getMyBack()
  },
  watch: {
    $route () {
      if (this.$route.fullPath === '/Teacher/HelpFeedback/MyBack') {
        this.getMyBack()
      }
    }
  },
  data () {
    return {
      userId: '',
      myBack: [],
      pageNo: 1,
      totalNum: 1
    }
  },
  methods: {
    getMyBack () {
      this.$http.post('/Customer/Suggestion/GetDataList', { PageIndex: this.pageNo, PageRows: 10, condition: 'CreatorId', keyword: this.userId }).then(res => {
        console.log('我的反馈---', res)
        this.myBack = res.Data
        this.totalNum = res.Total
      })
    },
    onPaging (pageNumber) {
      this.pageNo = pageNumber
      this.getMyBack()
    },
    returnGo () {
      this.$router.go(-1)
    }
  }
}
</script>

<style lang="less" scoped>
  .my-back {
    padding-bottom: 15px;
    font-size: 16px;
    color: #737976;
    border-bottom: 1px solid #ccc;
  }
  .my-back-name {
    margin-left: 15px;
  }
  .back-list {
    padding-bottom: 10px;
    font-size: 14px;
    color: #737976;
    border-bottom: 1px solid #ccc;
  }
  .back-info {
    margin-top: 15px;
  }
</style>
