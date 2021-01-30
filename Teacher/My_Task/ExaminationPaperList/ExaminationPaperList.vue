<template>
  <div>
    <div class="total" >
      <div class="total-div">
        <span class="total-span">
          <a-cascader
            :field-names="{ label: 'ChapterName', value: 'ChapterId', children: 'Firsts' }"
            style="width: 350px;height:60px;"
            :options="options"
            change-on-select
            @change="onChangeKnowledge"
            placeholder="全部练习"/>
        </span>
        <a-select
          label-in-value
          :default-value="{ key: '' }"
          style="height: 60px;"
          @change="handleChangeTime"
        >
          <a-select-option value="">
            所有日期
          </a-select-option>
          <a-select-option value="1">
            近一周
          </a-select-option>
          <a-select-option value="2">
            近一个月
          </a-select-option>
        </a-select>
        <!-- <span @click="waitExamine" class="wait-examine cur">待审核</span> -->
        <span class="fg search-data">
          <span class="search-view">
            <a-input v-model="key" placeholder="输入练习名称"/>
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
    <a-empty
      v-if="taskExamination"
      style="margin-top: 15px"
    >
      <span slot="description">暂无全区作业审查</span>
    </a-empty>
    <div id="total">
      <div v-for="item in paperList" :key="item.PaperId" class="dataPaper" >
        <p class="dataPaper-title">{{ item.PaperTitle }}<span class="dataPaper-check fg" :class="{'notCheck':item.IsCheck==0}">{{ item.IsCheck==0?'未审核':'已审核' }}</span></p>
        <span class="dataPaper-chapterName m-b">所属单元：{{ item.ChapterName }}</span>
        <p class="dataPaper-chapterName">{{ item.GradeName }}年级</p>
        <span class="dataPaper-slogan fg" v-if="item.IsCheck===0">打开专科专练教师端app进行审查</span>
        <div class="dataPaper-info fg cur" v-if="item.IsCheck===1" @click="lookInfo(item.PaperId)">查看</div>
        <p class="dataPaper-time">推送时间：{{ item.PublishTime }}</p>
      </div>
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
    this.getGradeKnowledge()
    this.GetTeacherPaperList()
  },
  updated () {
    this.heightPage = document.getElementById('total').offsetHeight
  },
  data () {
    return {
      heightPage: 0,
      options: [],
      day: '',
      key: '',
      type: '',
      userId: '',
      chapterId: '',
      paperList: [],
      // 是否存在作业审查
      taskExamination: false
    }
  },
  methods: {
    // 切换待审核
    waitExamine () {

    },
    getGradeKnowledge (id) {
      this.$http.post('/Paper/Exam_SubjectChapter/GetChapterTreeByUserId', { userId: this.userId }).then(res => {
        console.log('年级对应知识点---', res)
        this.options = res.Data
        console.log(this.options)
      })
    },
    GetTeacherPaperList () {
      this.$uwonhttp.post('/Paper/TeacherPaper/PaperCenterTwo', {
        Check: 2,
        Range: 0,
        Key: this.key,
        GradeId: 0,
        TeacherId: this.userId,
        PageNo: 1,
        Day: this.day,
        ChapterId: this.chapterId
      }).then(res => {
        console.log('审查试卷列表', res)
        this.paperList = res.data.Data.Items
        if (this.paperList.length === 0) {
          this.taskExamination = true
        } else {
          this.taskExamination = false
        }
      })
    },
    handleChangeTime (value) {
      // this.day = value.key
    },
    search () {
      this.GetTeacherPaperList()
    },
    Reset () {
      this.type = ''
      this.day = ''
      this.key = ''
      console.log('重置')
    },
    // 知识点选择(我的题库)
    onChangeKnowledge (value) {
      // if (value.length > 1) {
      //   this.chapterId = value[1]
      // }
      // if (value.length === 0) {
      //   this.chapterId = ''
      // }
    },
    lookInfo (Id) {
      this.$router.push({ path: '/Teacher/My_Task/ExaminationPaperList/WholePracticePreview',
        query: {
          id: Id
        }
      })
    }
  }
}
</script>
<style lang="less" scoped>
  .m-b {
    margin-bottom: 17px;
  }
  .search-data {
    width: 600px;
  }
  // 待审核按钮
  .wait-examine {
    margin-left: 50px;
    padding: 10px 20px;
    background-color: rgb(104,187,152);
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
