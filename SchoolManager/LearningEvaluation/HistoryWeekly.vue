<template>
  <div class="wrapper">
    <div class="head-info"><img class="img" src="@/assets/weekly/weekly.png" alt="" />历史周报</div>


    <ul v-if="isLoading" class="list">
      <div class="example">
        <a-spin />
      </div>
    </ul>
    <ul class="list" v-if="Lists">
      <li class="item" v-for="(item, index) in Lists" :key="index">
        <div class="tit">
          {{ item.Title }} <span>{{ item.WeeklyTime }} </span>
        </div>
        <div class="right" @click="toLink(item.Id)">查看 <a-icon type="right" /></div>
      </li>
    </ul>
    <ul v-else class="list">
      <div class="example">
        <a-empty />
      </div>
    </ul>

    <a-pagination
      :hideOnSinglePage="true"
      class="text-center"
      v-model="pageIndex"
      :total="TotalItems"
      @change="changePage"
      :defaultPageSize="pageSize"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      pageIndex: 1,
      pageSize: 10,
      Lists: [],
      isLoading: false,
      TotalItems: 0,
    }
  },
  methods: {
    changePage(num) {
      this.pageIndex = num
      this.getData()
    },
    init() {
      this.pageIndex = 1
    },
    async getData() {
      var req = {
        pageNo: this.pageIndex,
        pageIndex: this.pageSize,
      }

      this.isLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/StudentWeekly/GetHistoryStudentWeekly', req)
        this.Lists = res.data.Data.Items
        this.TotalItems = res.data.Data.TotalItems
      } catch (error) {
        console.log(error)
        throw error
      }

      this.isLoading = false
    },
    toLink(id) {
      this.$router.push({
        path: '/SchoolManager/LearningEvaluation/Weekly',
        query: {
          weeklyId: id,
        },
      })
    },
  },
  activated() {
    this.getData()
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
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    // min-height: 300px;
  }
  .head-info {
    margin: 24px auto;
    padding: 0 16px;
    line-height: 48px;
    background: #ffffff;
    border-radius: 8px;

    font-size: 16px;

    color: #4d5753;
    img {
      width: 32px;
      vertical-align: middle;
      margin-right: 4px;
    }
  }

  .list {
    // margin-top: 16px;

    background: #ffffff;
    border-radius: 8px;

    .item {
      border-bottom: 1px solid #d6dcde;
      line-height: 42px;
      padding: 10px 16px 0;

      width: 100%;

      // padding: 24px 0;

      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: space-between;
      font-size: 12px;
      &:last-child {
        border-bottom: transparent 1px solid;
      }
      .tit {
        font-size: 16px;

        color: #505050;
        span {
          font-size: 12px;

          color: #9f9f9f;
          margin-left: 21px;
        }
      }
      .right {
        color: #0ab7ef;
        cursor: pointer;
      }
    }
  }
  .empty-mod {
    width: 100%;
    height: 300px;
    margin-top: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .text-center {
    text-align: center;
    margin-top: 64px;
  }

  /deep/ .ant-pagination-item {
    color: rgba(0, 0, 0, 0.65);

    &.ant-pagination-item-active {
      font-weight: 500;
      background: #68bb97;
      color: #fff;
      border-color: #68bb97;
      a {
        color: #fff;
      }
    }
    &:hover,
    &:focus {
      font-weight: 500;
      background: #68bb97;
      color: #fff;
      border-color: #68bb97;
      a {
        color: #fff;
      }
    }
  }
}
</style>
