<template>
  <div>
    <div class="search-friend">
      <a-input placeholder="输入老师姓名" v-model="searchTeacher"></a-input>
      <span @click="search">搜索</span>
      <span @click="reset">重置</span>
    </div>
    <!-- 老师的信息 -->

    <div class="teacher-infor">
      <img src="@/assets/teacher/school.png" alt=""><span class="school">{{ teacherName }}</span>
      <div class="school-infor">
        <ul class="clearfix">
          <li v-for="(item,index) in teacherList" :key="index">
            <p class="cur">{{ item.GradeName }}</p>
            <!-- 0 互为好友  1 加好友 2 审核中 -->
            <p v-for="(i,j) in item.FriendList" :key="i.UserId"><img src="@/assets/teacher/老师头像.png" alt="">&nbsp;&nbsp;&nbsp;&nbsp; <span>{{ i.RealName }}</span><span>共享练习{{ i.PracticeCount }}份</span> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
              <span @click="AddFriend(i.UserId)" v-show="i.Deleted === 0" class="mutual-friend cur">加好友</span>
              <span v-show="i.Deleted === 1" class="add-friends cur">互为好友</span>
              <span v-show="i.Deleted === 2" class="under-review">申请中</span>
            </p>
          </li>
        </ul>
      </div>
      <a-pagination
        v-model="current"
        :total="50"
        show-quick-jumper
        :page-size="2"
        style="text-align: center;"
        @change="onChange"/>

    </div>
    <p class="footer">Copyright©上海有我科技有限公司，All Rights Reserved</p>
  </div>
</template>

<script>
import '@/utils/utils.less'
import Footer from '@/components/XuHuiFooter/FooterLogo'
export default {
  components: {
    Footer
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    console.log(this.userId)
    this.getTeacherList()
  },
  data () {
    return {
      userId: '',
      teacherList: [],
      teacherName: '',
      current: 1,
      // 搜索老师名称
      searchTeacher: ''

    }
  },
  methods: {
    getTeacherList () {
      this.$http.post('/Friend/FriendRelationShip/GetTeacherList', { userId: this.userId, pageNo: this.current }).then(res => {
        console.log(res)
        this.teacherList = res.Data.Items[0].GradeList
        this.teacherName = res.Data.Items[0].SchoolName
      })
    },
    // 添加好友
    AddFriend (id) {
      console.log(id)
      this.$http.post('/Friend/FriendRelationShip/AddFriend', { userId: this.userId, friendsId: id }).then(res => {
        console.log('添加好友--', res)
      })
      this.getTeacherList()
    },
    // 搜索
    search () {
      this.$http.post('/Friend/FriendRelationShip/GetTeacherList', { userId: this.userId, teacherName: this.searchTeacher }).then(res => {
        console.log(res)
        this.teacherList = res.Data.Items[0].GradeList
        this.teacherName = res.Data.Items[0].SchoolName
      })
    },
    // 重置
    reset () {
      this.searchTeacher = ''
      this.getTeacherList()
    },
    onChange (current) {
      this.current = current
      console.log(this.current)
      this.getTeacherList()
    }
  }
}
</script>

<style lang="less" scoped>
  .cur {
    cursor: pointer;
  }
  // 搜索
  .search-friend {
    .ant-input {
      width: 449px;
      height: 55px;
      margin-left: 10px;
      margin-right: 30px;
      text-indent: 15px;
      border-radius: 35px;
      border: 0;
    }
    span {
      display: inline-block;
      width: 124px;
      height: 55px;
      line-height: 55px;
      margin-right: 56px;
      text-align: center;
      border-radius: 25px;
      cursor: pointer;
    }
    span:nth-child(2) {
      background-color: #68BB97;
    }
    span:nth-child(3) {
      background-color: #A2C240;
    }
  }
  .teacher-infor {
    margin: 20px 0 0 20px;
    padding: 20px 50px;
    background-color: #fff;
    border-radius: 5px;
    .school {
      cursor: pointer;
    }
    .school-infor {
      margin-top: 30px;
      li {
        margin-right: 100px;
        margin-bottom: 50px;
        display:inline-block;
        vertical-align:top;
      }
      // 互为好友
      .mutual-friend {
        display: inline-block;
        padding: 4px 15px;
        border-radius: 5px;
        color: #68BB97;
        border: 1px solid #68BB97;
      }
      // 加好友
      .add-friends {
        display: inline-block;
        padding: 4px 15px;
        border-radius: 5px;
        color: #cccccc;
        border: 1px solid #cccccc;
      }
      // 申请中
      .under-review {
        display: inline-block;
        padding: 4px 15px;
        border-radius: 5px;
        color: #FF682C;
        border: 1px solid #FF682C;
      }
    }
  }
  .footer {
    font-size: 12px;
    margin-top: 20px;
    text-align: center;
    color: #B5B5B5;
  }

</style>
