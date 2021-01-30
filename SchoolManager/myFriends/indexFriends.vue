<template>
  <div class="friend-data">
    <div class="friend" id="friend">
      <!-- 星标好友 -->
      <div class="star-beacon" v-if="!loading">
        <p class="title">星标好友</p>
        <p class="star-infor" v-for="item in starFriendsList" :key="item.FriendsId">
          <img :src="item.Photo" alt="">
          <span>
            <p>{{ item.RealName }}</p>
            <p>{{ item.SchoolName }}</p>
            <p>{{ item.Grade }}</p>
          </span>
          <span>共享练习{{ item.PracticeCount }}份</span>
          <span class="cur color" style="font-size: 28px" @click="handleStarFriends(item.FriendsId,item.Interstellar)">☆</span>
        </p>
      </div>
      <!-- 好友列表 -->
      <div class="friend-list" v-if="!loading">
        <p class="title">好友列表</p>
        <div class="search-friend">
          <a-input placeholder="输入老师姓名" v-model="searchTeacher" @keyup.enter="search"></a-input>
          <span @click="search">搜索</span>
          <span @click="reset">重置</span>
          <span @click="addFriend">添加好友</span>
        </div>
        <div class="teacher-infor">
          <img v-if="this.starFriendsList.length !== 0" src="@/assets/teacher/school.png" alt=""><span class="school">{{ teacherName }}</span>

          <div class="school-infor">
            <ul class="clearfix">
              <li v-for="(item,index) in teacherList" :key="index">
                <p class="cur">{{ item.GradeName }}</p>
                <p v-for="(i,j) in item.FriendList" :key="j" @mouseenter="enterMouse(i.FriendsId)" @mouseleave="leave(i.FriendsId)"><img src="@/assets/teacher/老师头像.png" alt="">&nbsp;&nbsp;&nbsp;&nbsp; <span>{{ i.RealName }}</span><span>共享练习{{ i.PracticeCount }}份</span> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                  <span @click="handleStarFriends(i.FriendsId,i.Interstellar)" class="star-num" :class="{'color': i.Interstellar === '1'}" >☆</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                  <span class="cur" v-show="deleteFri === i.FriendsId" @click="deleteFriend(i.FriendsId)" >删除好友</span>
                </p>
              </li>
            </ul>
          </div>
          <a-pagination
            class="paging"
            v-model="current"
            :total="total"
            show-quick-jumper
            :page-size="1"
            hideOnSinglePage
            @change="onChange"/>
        </div>
      </div>
      <div v-if="loading" style="text-align:center;margin-top: 20px;margin:200px auto;">
          <img src="@/assets/lack/暂无关注好友.png" alt="">
          <p style="color:#ccc">暂无关注好友</p>
          <div class="addFriend" @click="addFriend">添加好友</div>
        </div>
    </div>
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
    console.log(this.userId)
    // 好友列表s
    this.getTeacherList()
    // 获取星标好友
    this.getStarBeaconFriend()
  },
  updated () {
    this.heightPage = document.getElementById('friend').offsetHeight
  },
  watch: {
    $route () {
      if (this.$route.fullPath === '/Teacher/My_Friends/FriendsManage') {
        // 好友列表s
        this.getTeacherList()
        // 获取星标好友
        this.getStarBeaconFriend()
      }
    }
  },
  data () {
    return {
      heightPage: 0,
      userId: '',
      teacherList: [],
      teacherName: '',
      current: 1,
      // 搜索老师名称
      searchTeacher: '',
      value: 0,
      // 删除
      deleteFri: '',
      // 总数
      total: 0,
      // 星标好友列表
      starFriendsList: [],
      loading: true

    }
  },
  methods: {
    // 好友列表s
    getTeacherList () {
      debugger
      this.$http.post('/Friend/FriendRelationShip/GetFriendList', { userId: this.userId, pageNo: this.current }).then(res => {
        console.log('好友列表--', res)
        if (res.Data.Items.length !== 0) {
          this.teacherList = res.Data.Items[0].GradeList
          this.teacherName = res.Data.Items[0].SchoolName
          this.total = res.Data.TotalItems
          this.loading = false
          console.log('页数', this.total)
        } else {
          this.teacherList = ''
          this.teacherName = ''
        }
      })
    },
    // 获取星标好友
    getStarBeaconFriend () {
      this.$http.post('/Friend/FriendRelationShip/GetFriendRelationshipList', { userId: this.userId }).then(res => {
        console.log('星标好友--', res)
        this.starFriendsList = res.Data
      })
    },
    // 添加星标好友
    addStarFriend (id) {
      this.$http.post('/Friend/FriendRelationShip/AddFriendRelationship', { userId: this.userId, friendsId: id, type: 1 }).then(res => {
        console.log('添加星标好友--', res)
        if (res.Success) {
          this.$message.success('添加星标成功')
          this.getTeacherList()
          this.getStarBeaconFriend()
        }
      })
    },
    // 取消星标好友
    cancelStar (id) {
      this.$http.post('/Friend/FriendRelationShip/AddFriendRelationship', { userId: this.userId, friendsId: id, type: 0 }).then(res => {
        console.log('取消星标好友--', res)
        if (res.Success) {
          this.$message.success('取消星标成功')
          this.getTeacherList()
          this.getStarBeaconFriend()
        }
      })
    },
    // 星标处理
    handleStarFriends (id, star) {
      console.log('星标处理---', star)
      if (star === '0') {
        this.addStarFriend(id)
      } else {
        this.cancelStar(id)
      }
    },
    // 移入
    enterMouse (id) {
      this.deleteFri = id
    },
    leave () {
      this.deleteFri = ''
    },
    // 删除好友
    deleteFriend (id) {
      debugger
      console.log('删除ID---', id)
      this.$http.post('/Friend/FriendRelationShip/DeleteFriend', { userId: this.userId, friendsId: id }).then(res => {
        console.log('删除好友--', res)
        if (res.Success) {
          this.getTeacherList()
          this.getStarBeaconFriend()
        }
      })
    },
    // 搜索
    search () {
      this.$http.post('/Friend/FriendRelationShip/GetFriendList', { userId: this.userId, teacherName: this.searchTeacher }).then(res => {
        console.log(res)
        if (res.Data.Items.length === 0) {
          this.$message.error('没有该老师')
          this.searchTeacher = ''
        } else {
          this.teacherList = res.Data.Items[0].GradeList
          this.teacherName = res.Data.Items[0].SchoolName
        }
      })
    },
    // 重置
    reset () {
      this.searchTeacher = ''
      this.getTeacherList()
    },
    // 添加好友
    addFriend () {
      this.$router.push({ path: '/SchoolManager/myFriends/appendFriend' })
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
  .color {
     color: #FFDA26;
  }
  // 题目
  .title {
    color:#4D5753;
    font-size: 18px
  }
  .ant-layout-content {
    /deep/.content {
    height: 100%;
    }
    /deep/.page-header-index-wide {
      height: 100%;
      }
  }
  .friend-data {
    height: 100%;
  }
  .friend {
    display: flex;
    height: 100%;
    flex-wrap: wrap;

    // 星标好友
    .star-beacon {
      width: 30%;
      height: 700px;
      margin-right: 5px;
      padding: 20px;
      background-color: #fff;
      // overflow:auto;

      .star-infor {
        margin-top: 27px;
        margin-bottom: 63px;
        img {
        width: 60px;
        border-radius: 30px;
      }
      span {
        margin-right: 50px;
      }
        span:nth-child(2) {
          display: inline-block;
          margin-left: 15px;
          vertical-align: middle;
        }
        span:nth-child(4) {
          margin-right: 0;
        }
      }
    }
    // 好友列表
    .friend-list {
      width: 69%;
      padding: 20px;
      background-color: #fff;
    }
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
    }
    span {
      display: inline-block;
      width: 124px;
      height: 55px;
      line-height: 55px;
      margin-right: 56px;
      text-align: center;
      color: #fff;
      border-radius: 40px;
      cursor: pointer;
    }
    span:nth-child(2) {
      background-color: #68BB97;
    }
    span:nth-child(3) {
      background-color: #A2C240;
    }
    span:nth-child(4) {
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
        border: 1px solid #000000;
      }
      // 加好友
      .add-friends {
        display: inline-block;
        padding: 4px 15px;
        border-radius: 5px;
        color: #cccccc;
        border: 1px solid #cccccc;
      }
      .under-review {
        display: inline-block;
        padding: 4px 15px;
        border-radius: 5px;
        color: #FF682C;
        border: 1px solid #FF682C;
      }
    }
  }
  // 星标显示
  /deep/.ant-rate-disabled .ant-rate-star {
    cursor: pointer;
  }
  .star-num {
    font-size: 18px;
    cursor: pointer;
  }
   .footer {
    font-size: 12px;
    margin: 0 auto;
    margin-top: 80px;
    color: #B5B5B5;
  }
  .paging {
    margin-top: 40px;
      text-align: center;
     /deep/.ant-pagination-item:focus, .ant-pagination-item:hover {
      border-color: #68BB97;
      border-color: #68BB97;
    }
    /deep/.ant-pagination-item-active a {
      color: #000;
      background-color: #68BB97;
    }
  }
  .addFriend{
    margin: 10px auto;
    width: 100px;
    height: 30px;
    border: 1px solid #68BB97;
    border-radius: 100px;
    line-height: 30px;
    text-align: center;
    color: #68BB97;
    cursor: pointer;
  }
</style>
