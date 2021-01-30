<template>
  <div>
    <div class="total" >
      <div class="total-div">
        <span class="m-r">
          <a-select
            label-in-value
            style="height: 60px;"
            placeholder="选择年级"
            @change="handleChangeTime"
          >
            <a-select-option value="1">
              一年级
            </a-select-option>
            <a-select-option value="2">
              二年级
            </a-select-option>
            <a-select-option value="3">
              三年级
            </a-select-option>
            <a-select-option value="4">
              四年级
            </a-select-option>
            <a-select-option value="5">
              五年级
            </a-select-option>
          </a-select>
        </span>
        <span v-show="termName">
          <a-select
            style="height: 60px;"
            placeholder="选择学期"
            :defaultValue="termName"
            @change="handleChangeTerm"
          >
            <a-select-option :value="1">
              上学期
            </a-select-option>
            <a-select-option :value="2">
              下学期
            </a-select-option>
          </a-select>
        </span>
        <!-- <span @click="handleGrade(1)" class="grade-select sel-le cur" :class="{ 'sel-grade-col': this.termId === 1}" >上学期</span>
        <span @click="handleGrade(2)" class="grade-select cur" :class="{ 'sel-grade-col': this.termId === 2}">下学期</span> -->
        <span class="fg search-data">
          <span class="search-view">
            <a-input v-model="paperTitle" placeholder="输入练习名称"/>
          </span>
          <span class="search-view-btn" @click="search">
            搜索
          </span>
          <span class="search-view-cz" @click="Reset">
            重置
          </span>
        </span>
      </div>
    </div>
    <div class="trial-lack" v-if="showLack">
      <img src="@/assets/lack/暂无搜索记录.png" alt="">
      <p>暂无搜索内容</p>
    </div>
    <div id="total">
      <div v-for="item in paperList" :key="item.PaperId" class="dataPaper" >
        <p class="dataPaper-title">{{ item.Title }}
          <span class="dataPaper-check fg" :class="{'notCheck':item.IsCheck==0}">{{ item.AuditStatus==2?'已审核':'未审核' }}</span>
        </p>
        <span class="dataPaper-chapterName m-b">所属单元：{{ item.ChapterName }}</span>
        <p class="dataPaper-chapterName">{{ item.GradeName }} 出题人：{{ item.RealName }}</p>
        <!-- <span class="dataPaper-slogan fg" v-if="item.IsCheck===0">打开专科专练教师端app进行审查</span> -->
        <div class="dataPaper-info fg cur" v-if="item.AuditStatus === 1 || item.AuditStatus === 0" @click="lookInfo(item.PaperId,item.Grade)">审查</div>
        <p class="dataPaper-time">推送时间：{{ item.CreateTime }}</p>
      </div>
    </div>
    <!-- 分页 -->
    <div class="paging">
      <a-pagination :default-current="pageNo" :total="this.pageTotal" @change="onPaging" hideOnSinglePage />
    </div>
    <!-- <p class="footer">Copyright©上海有我科技有限公司，All Rights Reserved</p> -->
    <footerLogo :heightPage="heightPage"></footerLogo>
  </div>
</template>

<script>
import footerLogo from '@/components/newFooterLogo/newFooterLogo'
export default {
  components: {
    footerLogo
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    // 试卷列表
    this.GetTeacherPaperList()
    this.getGradeSit()
  },
  updated () {
    this.heightPage = document.getElementById('total').offsetHeight
  },
  watch: {
    $route () {
      this.pageNo = this.nowPageNo
      if (this.$route.fullPath === '/Teacher/My_Task/AssistExamine/AssistExamine') {
        this.GetTeacherPaperList()
      }
    }
  },
  data () {
    return {
      termName: 1,
      // 缺省
      showLack: false,
      heightPage: 0,
      day: '',
      key: '',
      type: '',
      userId: '',
      chapterId: '',
      // 当前页
      pageNo: 1,
      nowPageNo: 1,
      // 年级
      gradeId: '',
      // 搜索试卷
      paperTitle: '',
      // 学期
      termId: 1,
      paperList: [],
      // 分页总数
      pageTotal: 1
    }
  },
  methods: {
    // 获取年级上下学期
    getGradeSit () {
      this.$http.get('/HomePageDataView/TeacherManage/GetTerm').then(res => {
        console.log('上下学期', res)
        this.termName = res.Data.Term
        this.termId = res.Data.Term
        console.log('上下学期', this.termName)
      })
    },
    // 年级选择
    handleChangeTime (value) {
      console.log(value.key)
    },
    // 学期选择
    handleChangeTerm (value) {
      console.log(value.key)
    },
    // /Teacher/My_Task/AssistExamine/AssistExamine
    // 获取试卷列表
    GetTeacherPaperList () {
      this.$http
        .post('/HomePageDataView/TeacherManage/GetMangerPaperList', {
          UserId: this.userId,
          Grade: this.gradeId,
          ChapterId: this.chapterId,
          PaperTitle: this.paperTitle,
          PageNo: this.pageNo,
          PageIndex: 8,
          Term: this.termId
        })
        .then(res => {
          if (res.Data.Items.length === 0) {
            // this.load = false
            this.showLack = true
          } else {
            this.showLack = false
          }
          this.paperList = res.Data.Items
          console.log('试卷信息---', res)
          this.pageTotal = res.Data.TotalItems
          // this.load = false
        })
    },
    onPaging (pageNum) {
      this.pageNo = pageNum
      console.log('分页---', this.pageNo)
      this.nowPageNo = this.pageNo
      this.GetTeacherPaperList()
    },
    search () {
      this.GetTeacherPaperList()
    },
    Reset () {
      this.type = ''
      this.day = ''
      this.key = ''
      this.paperTitle = ''
      console.log('重置')
      this.GetTeacherPaperList()
    },
    lookInfo (Id, gradeId) {
      this.$router.push({ path: '/TeachingFellow/ModifyPaper',
        query: {
          paperId: Id,
          grade: gradeId,
          isFellow: 0
        }
      })
    }
  }
}
</script>
<style lang="less" scoped>
  .m-r {
    margin-right: 50px;
  }
  .m-b {
    margin-bottom: 17px;
  }
  .search-data {
    width: 600px;
  }

  .trial-lack {
    text-align: center;
    color: #ccc;
  }
   // 年级选择
  .grade-select {
    margin-right: 16px;
    padding: 3px 12px;
    color: #fff;
    background-color: #ddd;
    border-radius: 5px;
  }
  // 搜索框
.search-view {
  display: inline-block;
  width: 300px;
  // margin-top: 40px;
  text-align: center;
  //  margin-left:24% ;
}
input-placeholder {
  text-align: center;
}
.search-view .ant-input{
  width: 100%;
  height: 50px;
  // margin-left: 80px;
  text-indent: 15px;
  border-radius: 35px;
  border: 1px solid #ccc;
}
// 搜索按钮
.search-view-btn,.search-view-cz {
    display: inline-block;
    cursor: pointer;
    width: 100px;
    text-align: center;
    margin-left: 5%;
    height: 40px;
    line-height: 40px;
    background-color: rgb(104,188,150);
    border-radius: 30px;
    color: white;
}
.search-view-cz {
   background-color: rgb(162,194,67);
}
.total {
  background-color: white;
  height: 150px;
  border-radius: 5px;
}
.total-div {
  padding-top: 40px;
  padding-left: 50px;
}
.total-span {
  // background-color: red;
}
.dataPaper {
  display: inline-block;
  margin-right: 25px;
  width: 31%;
  margin-top: 50px;
  padding: 15px 20px 15px 15px;
  background-color: white;
  position: relative;
  border-radius: 10px;
}
.dataPaper-title {
    margin-bottom: 10px;
    padding-top: 1%;
    padding-left: 2%;
    font-size: 20px;
    font-weight: 900;
}
.dataPaper-chapterName {
    padding-top: 1%;
    padding-left: 2%;
    font-size: 10px;
    color: #737976;
}
.dataPaper-check {
  // width: 80px;
  // height: 30px;
  // line-height: 30px;
  padding: 3px 15px;
  font-size: 15px;
  text-align: center;
  color: white;
  background-color: rgb(218,216,217);
  border-radius: 20px;
}
.notCheck {
  background-color: rgb(80,88,83);
}
.dataPaper-info {
  // width: 100px;
  // height: 40px;
  // line-height: 40px;
  margin-top: -3.5%;
  padding: 9px 18px;
  // margin-right: 10%;
  text-align: center;
  color: white;
  background-color: rgb(104,187,152);
  border-radius: 5px;
}
.dataPaper-slogan {
  margin-top: -3.5%;
  margin-right: 10%;
}
.dataPaper-time {
  padding-left: 2%;
   padding-top: 1%;
    color: rgb(230,228,229);
}
/deep/.ant-cascader-input.ant-input  {
  border-radius: 35px;
  height: 50px;
  text-align: center;
}
/deep/.ant-select-selection--single {
    height: 50px;
    width: 200px;
    border-radius: 35px;
    padding-left:50px;
    padding-top:10px;
    margin-left:10% ;
}
</style>
