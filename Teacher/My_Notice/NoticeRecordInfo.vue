<template>
  <div>
    <div class="studentCotice"><div style="font-size: 18px;padding-left: 2%;padding-top: 14px;">学生通知</div></div>
    <div class="dataInfo" id="dataInfo">
      <div class="clearfix">
        <a-select
          style="width: 120px; marginRight: 20px;"
          @change="handleChangeType"
          show-search
          label-in-value
          :default-value="{ key: '0' }">
          <a-select-option value="0">
            所有类型
          </a-select-option>
          <a-select-option value="1">
            通知
          </a-select-option>
          <a-select-option value="2">
            作业
          </a-select-option>
        </a-select>
        <a-select
          label-in-value
          :default-value="{ key: '' }"
          style="width: 120px"
          @change="handleChangeTime"
        >
          <a-select-option value="">
            所有日期
          </a-select-option>
          <a-select-option value="1">
            上周
          </a-select-option>
          <a-select-option value="2">
            上个月
          </a-select-option>
          <a-select-option value="3">
            自定义时间
          </a-select-option>
        </a-select>

        <span class="search-view">
          <a-input @keyup.enter="search" v-model="content" placeholder="输入名称"/>
        </span>
        <span class="search-view-btn" @click="search" >
          搜索
        </span>
        <span class="search-view-cz" @click="Reset">
          重置
        </span>
        <div v-if="historicalNotice" class="no-history-notice">
          <p><img src="@/assets/lack/暂无历史通知.png" alt=""></p>
          <p>暂无历史通知</p>
        </div>

        <div v-show="!historicalNotice" v-for="(item,indexfirst) in NoticeData" :key="indexfirst" class="history-infor" >
          <!-- <hr v-if="indexfirst!=0"> -->
          <p style="font-weight:900;font-size:20px;">{{ item.PublishDate }}</p>
          <div v-for="(child,index) in item.Contents" :key="index">
            <p class="line" style="margin-bottom: 15px" v-if="index!=0"/>
            <p style="font-weight:700;font-size:15px;color: #000">{{ child.Title }}</p>
            <div v-for="(childcontent,childindex) in child.ItemContent" :key="childindex">
              <p v-if="childindex<(childcontent.Id===NoticeId?child.ItemContent.length:2)">{{ childcontent.Content }}</p>
            </div>
            <!-- <span class="lookInfo fg" v-if="child.ItemContent.length>2&&child." @click="lookInfo(child.ItemContent.length,child.ItemContent[0].Id)">查看详情</span> -->
            <span class="lookInfo fg cur" v-if="child.ItemContent.length>2" @click="lookInfo(child.ItemContent.length,child.ItemContent[0].Id)">查看详情</span>
          </div>
        </div>
      </div>
    </div>
    <!-- 分页 -->
    <div class="paging">
      <a-pagination show-quick-jumper :default-current="1" :total="totalNum" @change="onPaging" hideOnSinglePage />
    </div>
    <!-- <p class="footer">Copyright©上海有我科技有限公司，All Rights Reserved</p> -->
    <footerLogo :heightPage="heightPage"></footerLogo>
  </div>
</template>
<script>
import Swiper from 'swiper'
import footerLogo from '@/components/newFooterLogo/newFooterLogo'
export default {
  components: {
    Swiper,
    footerLogo
  },
  created () {
    this.classId = this.$route.query.classId
    this.NoticeRecord()
  },
  updated () {
    this.heightPage = document.getElementById('dataInfo').offsetHeight
    // console.log('历史通知的高度---', this.heightPage)
  },
  wacth: {
    $route () {
      console.log('wacth...............')
      const ChangeId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/My_Notice/NoticeRecordInfo?' + ChangeId) {
        this.NoticeRecord()
      }
    }
  },
  mounted () {

  },
  data () {
    return {
      heightPage: 0,
      type: 0,
      day: '',
      content: '',
      pageNo: 1,
      classId: '',
      NoticeData: {},
      indexNumber: 2,
      NoticeId: '',
      indexNumbers: -1,
      // 暂无历史通知
      historicalNotice: false,
      // 通知总数
      totalNum: 0
    }
  },
  methods: {
    NoticeRecord () {
      this.$http
        .post('/Notice/NoticeRecord/GetNoticeRecordList',
          {
            Type: this.type,
            Day: this.day,
            Content: this.content,
            PageNo: this.pageNo,
            Today: 1,
            classId: this.classId
          }).then(res => {
          console.log('历史通知---', res)
          if (res.Data.length === 0) {
            this.historicalNotice = true
            this.totalNum = 0
          } else {
            this.historicalNotice = false
            this.NoticeData = res.Data
            this.totalNum = res.Data[0].TotalCount
            console.log(this.NoticeData)
          }
        })
    },
    lookInfo (index, id) {
      this.indexNumber = -1
      this.indexNumbers = index
      this.NoticeId = id
    },
    search () {
      console.log('this.type', this.type)
      console.log('搜索')
      this.NoticeRecord()
    },
    Reset () {
      this.type = '0'
      this.day = ''
      this.content = ''
      console.log('重置')
    },
    handleChangeType (value) {
      console.log(value.key)
      this.type = value.key
    },
    handleChangeTime (value) {
      this.day = value.key
    },
    onPaging (pagenum) {
      this.pageNo = pagenum
      this.NoticeRecord()
    }
  }
}
</script>
<style lang="less" scoped>
  .line {
    border-bottom: 1px solid rgba(232, 232, 232);
  }
 .swiper-container{
    --swiper-theme-color: #ff6600;/* 设置Swiper风格 */
    --swiper-navigation-color: #00ff33;/* 单独设置按钮颜色 */
    --swiper-navigation-size: 30px;/* 设置按钮大小 */
  }
  .studentCotice{
    height: 60px;
    // border:1px solid rgb(121,121,121);
    background-color: #fff;
    border-radius: 5px;
  }
  .dataInfo {
    // border:1px solid rgb(121,121,121);
    // border-width:0 1px 1px 1px;
  }
  // 暂无历史通知
  .no-history-notice {
    display: inline-block;
    position: absolute;
    top: 50%;
    left: 50%;
    -webkit-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
    text-align: center;
    color: rgba(165, 166, 167);
  }
  // 搜索框
.search-view {
  display: inline-block;
  margin-top: 40px;
  margin-left: 35%;
}
.search-view .ant-input{
  width: 150%;
  margin-left: -10%;
  text-indent: 15px;
  border: 0;
  height: 50px;
  border-radius: 20px;
}
// 搜索按钮
.search-view-btn,.search-view-cz {
  display: inline-block;
    cursor: pointer;
    width: 100px;
    text-align: center;
    height: 50px;
    background-color: #68BB97;
    border-radius: 40px;
    line-height: 50px;
    margin-left: 10px;
}
.search-view-btn{
  margin-left: 10%;
}
.search-view-cz {
   background-color: #A2C240;
}
.lookInfo {
  margin-top:-3%;
  margin-right:5%;
}
.history-infor {
  padding: 10px 20px;
  border-radius: 5px;
  margin-top: 20px;
  margin-bottom: 15px;
  background-color: #fff;
}
</style>
