<template>
  <div>
    <p class="title">历史编辑记录</p>
    <div class="history-info" v-for="item in historyList" :key="item.Id">
      <span>
        <span class="f-s">{{ item.Title }}</span>

        <span class="f-s-14 m-r">题目数量：{{ item.ItemCount }}道</span>
        <span class="f-s-14 f-s">{{ item.OptCount }}道单选题</span>
        <span class="f-s-14 f-s">{{ item.OptsCount }}道多选题</span>
        <span class="f-s-14 f-s">{{ item.JudgeCount }}道判断题</span>
        <span class="f-s-14 f-s">{{ item.PackCount }}道填空题</span>
        <span class="f-s-14 f-s">{{ item.ApplyCount }}道应用题</span>
      </span>
      <span class="fg">
        <span class="col">创建时间：{{ item.CreateTime }}</span>
        <span @click="continueEdit(item.Id)" class="m-r-30 m-l-60 handle-btn col-con cur">继续编辑</span>
        <span @click="delRecoed(item.Id)" class="m-r handle-btn col-del cur">删除记录</span>
      </span>
    </div>
    <!-- <div class="history-info">
      <span>
        <span class="f-s">高频错题(1)</span>
        <span class="f-s-14 m-r">题目数量：7道</span>
        <span class="f-s-14 f-s">2道选择题</span>
        <span class="f-s-14 f-s">5道填空题</span>
        <span class="f-s-14 f-s">0道应用题</span>
      </span>
      <span class="fg">
        <span class="col">创建时间：2020/12/2 15:00</span>
        <span class="m-r-30 m-l-60 handle-btn col-con">继续编辑</span>
        <span class="m-r handle-btn col-del">删除记录</span>
      </span>
    </div> -->
    <!-- 分页 -->
    <div class="paging">
      <a-pagination :default-current="pageNo" :total="this.pageTotal" @change="onPaging" hideOnSinglePage />
    </div>
  </div>
</template>

<script>
export default {
  created () {
    this.userId = localStorage.getItem('UserId')
    this.getHistoryList()
  },
  data () {
    return {
      pageNo: 0,
      pageTotal: 10,
      historyList: []
    }
  },
  methods: {
    getHistoryList () {
      this.$http.post('/HighFrequency/HighFrequency/GetHighPapers', { userId: this.userId, pageNo: this.pageNo }).then(res => {
        console.log('历史编辑', res)
        this.historyList = res.Data.Items
        this.pageTotal = res.Data.TotalItems
      })
    },
    onPaging (pageNum) {
      this.pageNo = pageNum
      this.getHistoryList()
    },
    delRecoed (id) {
      this.$http.post('/HighFrequency/HighFrequency/RemoveHighPaper', { userId: this.userId, paperId: id }).then(res => {
        console.log('删除记录', res)
        this.getHistoryList()
      })
    },
    // 继续编辑
    continueEdit (paperId) {
      this.$router.push({ path: '/Teacher/My_Task/HighWrong/EditPush',
        query: {
          paperId
        } })
    }
  }
}
</script>

<style lang="less" scoped>
  .f-s {
    margin-right: 24px;
    font-size: 20px;
  }
  .f-s-14 {
    font-size: 14px;
  }
  .m-l-60 {
    margin-left: 60px;
  }
  .m-r {
    margin-right: 80px;
  }
  .m-r-30 {
    margin-right: 30px;
  }
  .col {
    color: #959A98;
  }
  .col-con {
    background-color: #68BB97;
  }
  .col-del {
    background-color: #D6D6D6;
  }
  .handle-btn {
    width: 75px;
    height: 33px;
    padding: 9px 13px;
    color: #fff;
    border-radius: 5px;
  }
  .title {
    margin-bottom: 18px;
    font-size: 18px;
    color: #4D5753;
  }
  .history-info {
    height: 60px;
    line-height: 60px;
    margin-bottom: 16px;
    padding-left: 16px;
    background-color: #fff;
    border-radius: 5px;
  }
</style>
