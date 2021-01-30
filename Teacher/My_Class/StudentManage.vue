<template>
  <div>
    <div id="stu-manage">
      <!-- 学生管理信息 -->
      <div class="clearfix">
        <div class="fl stu-manage">
          <span>
            <a-select v-if="classId" class="select-flex" @change="handleChangeClass" :default-value="classId">
              <a-select-option v-for="item in classInfo" :key="item.Id" :value="item.Id">
                {{ item.ClassName }}
              </a-select-option>
            </a-select>
          </span>
          <span>共{{ classNum }}位学生</span>
          <span @click="handleLock()" class="cur"><img :src="lockIcon" alt="">&nbsp;&nbsp;&nbsp;{{ lockName }}</span>
          <a-modal
            v-model="Lock"
            title="温馨提醒"
            @ok="handleOkLock"
            width="630px"
            :bodyStyle="{ height: '215px',fontSize: '20px',padding: '40px 0 0 50px'}"
            centered
          >
            <p>确定要锁定班级吗？</p>
            <p>锁定班级后，其他学生将无法注册到您的班级。</p>
          </a-modal>
          <a-modal
            v-model="Unlock"
            title="温馨提醒"
            @ok="handleOkUnLock"
            width="630px"
            :bodyStyle="{ height: '215px',fontSize: '20px',padding: '40px 0 0 50px'}"
            centered
          >
            <p>确定要解锁班级吗？</p>
            <p>解锁班级后，新注册的学生可以直接进入您的班级。</p>
          </a-modal>
        </div>
        <div class="fg stu-manage-operation">
          <span @click="addStudent">添加学生</span>
          <span @click="manageStudent">{{ manageBtn }}</span>
          <span v-if="this.showCancel" @click="cancel">取消</span>
          <a-modal
            v-model="visible"
            title="提醒"
            @ok="handleOk"
            width="630px"
            centered
          >
            <p>确定移除吗？</p>
          </a-modal>
        </div>
      </div>
      <!-- 全部学生信息 -->
      <div class="all-stu-infor">
        <ul class="clearfix">
          <li v-for="(item,index) in allStudentList" :key="item.UserId">
            <img @click="removeStudent(item.UserId,$event)" v-show="checkStudent" class="img" :src="checkIcon" alt="">
            <p><img class="classStu-img" :src="item.Photo" alt=""></p>
            <p class="cur" style="margin-bottom: 5px;" @click="toStudentManage(item.UserId)">{{ item.UserNo }}</p>
            <p class="cur" @click="toStudentManage(item.UserId)">{{ item.TrueName }}</p>
          </li>
        <!-- <li>
          <img class="img" src="@/assets/teacher/未勾选.png" alt="">
          <p><img src="@/assets/teacher/老师头像.png" alt=""></p>
          <p>01</p>
          <p>杨思琦</p>
        </li> -->
        </ul>
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
    // 获取教师对应的班级
    this.getTeacherClassList()
  },
  updated () {
    this.heightPage = document.getElementById('stu-manage').offsetHeight
  },
  data () {
    return {
      heightPage: 0,
      // 教师ID
      userId: '',
      // 教师对应的班级
      classInfo: [],
      // 默认的班级ID
      classId: '',
      // 是否勾选
      checkStudent: false,
      // 勾选图标
      checkIcon: require('@/assets/teacher/未勾选.png'),
      // s所有学生信息
      allStudentList: [],
      // 班级人数
      classNum: '',
      // 管理按钮
      manageBtn: '管理',
      visible: false,
      // 上锁
      Lock: false,
      // 解锁
      Unlock: false,
      // 锁名称
      lockName: '',
      // 锁图标
      lockIcon: require('@/assets/teacher/未上锁.png'),
      arr: [],
      arr1: '',
      showCancel: false
    }
  },
  watch: {
    $route () {
      if (this.$route.fullPath === '/Teacher/My_Class/StudentManage') {
        this.getAllStudents(this.classId)
      }
    }
  },
  methods: {
    // 获取教师对应的班级
    getTeacherClassList () {
      this.$http.post('/ClassManage/Exam_Class/GetListByUserId', { userId: this.userId }).then(res => {
        console.log('教师对应的班级---', res.Data)
        console.log(this.$route.fullPath)
        this.classInfo = res.Data
        this.classId = res.Data[0].Id
        console.log('默认班级--', this.classId)
        // 获取全部学生信息
        this.getAllStudents(res.Data[0].Id)
        // 获取班级是否上锁
        this.getClassLock(res.Data[0].Id)
      })
    },
    // 获取全部学生信息
    getAllStudents (id) {
      this.$uwonhttp.post('/TeacherUserCenter/TeacherPersonal/GetStudentList', { classId: id }).then(res => {
        console.log('全部学生信息--', res)
        this.allStudentList = res.data.Data
        this.classNum = res.data.Data.length
        console.log('全部学生信息--', this.allStudentList)
      })
    },
    // 获取班级是否上锁
    getClassLock (id) {
      this.$uwonhttp.get('/Class/TeacherClassManager/GetClassMsg', {
        params: {
          ClassId: id
        }
      }).then(res => {
        console.log('是否上锁--', res)
        if (res.data.Data.IsLock === 0) {
          this.lockName = '未上锁'
          this.lockIcon = require('@/assets/teacher/未上锁.png')
        } else {
          this.lockName = '已上锁'
          this.lockIcon = require('@/assets/teacher/上锁.png')
        }
      })
    },
    manageStudent () {
      this.checkStudent = true
      if (this.manageBtn === '管理') {
        this.manageBtn = '移出学生'
        this.showCancel = true
      } else {
        this.visible = true
        this.showCancel = false
      }
    },
    cancel () {
      this.checkStudent = false
      this.showCancel = false
      this.manageBtn = '管理'
    },
    // 选择班级
    handleChangeClass (value) {
      console.log('选择班级--', value)
      this.classId = value
      this.arr1 = ''
      this.manageBtn = '管理'
      this.getAllStudents(value)
      this.getClassLock(value)
    },
    // 添加学生
    addStudent () {
      this.$router.push({ path: '/Teacher/My_Class/AddStudent', query: { classId: this.classId } })
    },
    // 学生管理
    toStudentManage (id) {
      console.log('学生id---', id)

      this.$router.push({ path: '/Teacher/My_Class/StudentInfor', query: { id, classId: this.classId } })
    },
    // 批量移出学生
    removeStudent (id, e) {
      console.log('学生id---', e.target.src)
      // const arr2 = []
      if (this.arr.indexOf(id) === -1) {
        this.arr.push(id)
      }

      console.log(this.arr)
      // debugger

      // console.log(this.arr1)
      if (e.target.src === require('@/assets/teacher/未勾选.png')) {
        e.target.src = require('@/assets/teacher/勾选.png')
      } else {
        e.target.src = require('@/assets/teacher/未勾选.png')
        // debugger
        const idx = this.arr.indexOf(id)
        if (idx !== -1) {
          this.arr.splice(idx, 1)
        }
      }

      this.arr1 = this.arr.join(',')
      console.log(this.arr1)
    },
    // 移出学生弹框确认
    handleOk () {
      this.$uwonhttp.post('/Student/User/DeleteStudentsWeb', { StudentId: this.arr1, TeacherId: this.userId }).then(res => {
        console.log('批量移除学生--', res)
        this.visible = false
        if (res.data.Success) {
          this.$message.success('成功移除学生')
          this.getAllStudents(this.classId)
        } else {
          this.$message.error(res.data.Msg)
        }
      })
    },
    // 处理上锁/解锁
    handleLock () {
      if (this.lockName === '未上锁') {
        this.Lock = true
      }
      if (this.lockName === '已上锁') {
        this.Unlock = true
      }
    },
    // 上锁
    handleOkLock () {
      this.$uwonhttp.get('/Class/TeacherClassManager/LockClass', {
        params: {
          UserId: this.userId,
          ClassId: this.classId
        }
      }).then(res => {
        console.log('上锁--', res)
        this.Lock = false
        this.lockName = '已上锁'
        this.lockIcon = require('@/assets/teacher/上锁.png')
      })
    },
    // 解锁
    handleOkUnLock () {
      this.$uwonhttp.get('/Class/TeacherClassManager/LockClass', {
        params: {
          UserId: this.userId,
          ClassId: this.classId
        }
      }).then(res => {
        console.log('解锁--', res)
        this.Unlock = false
        this.lockName = '未上锁'
        this.lockIcon = require('@/assets/teacher/未上锁.png')
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .classStu-img {
    width: 56px;
    height:56px;
    border-radius: 30px;
  }
  .select-flex {
    width: 128px;
    height: 48px;
    border-radius: 35px;
    background-color:#F4F4F4;
  }
  .cur {
    cursor: pointer;
  }
  .text {
    text-align: center;
  }
  /deep/.ant-modal-title {
    color: #fff;
  }
  .ant-modal-close-x {
    color: #fff;
  }
  // 学生管理信息
  .stu-manage {
    span:nth-child(1) {
      margin-right: 24px;
    }
    span:nth-child(2) {
      margin-right: 80px;
    }
  }
   .stu-manage-operation {
      span {
        // display: inline-block;
        width: 110px;
        height: 48px;
        line-height: 48px;
        margin-right: 24px;
        padding: 10px 15px;
        text-align: center;
        border-radius: 24px;
        color: #fff;
        background-color: #68BB97;
        cursor: pointer;
      }
      span:nth-child(2) {
        background-color: #A2C240;
      }
    }
  // 全部学生信息
  .all-stu-infor {
    margin-top: 16px;
    background-color: #fff;
    ul {
      padding: 32px 0 0 30px;
      background-color: #fff;
      border-radius: 5px;
      li {
        position: relative;
        float: left;
        width: 83px;
        height: 130px;
        margin-right: 60px;
        margin-bottom: 15px;
        text-align: center;
        // cursor: pointer;
        .img {
          position: absolute;
          top: -1px;
          left: 54px;
          cursor: pointer;
        }
      }
    }
  }
  // 选择框
  /deep/.ant-select-selection--single {
    width: 128px;
    height: 48px;
    border-radius: 35px;
    line-height: 48px;
  }
  /deep/.ant-select-selection__rendered {
    width: 128px;
    height: 48px;
    margin-left: 0;
    /* border-radius: 35px; */
    line-height: 48px;
  }
  /deep/.ant-select-selection-selected-value {
    width: 128px;
    text-align: center;

  }
   // 模态框
  /deep/.ant-modal-header {
    background-color: #68BB97;
  }
  /deep/.ant-modal-footer button + button {
    background-color: #68BB97;
    border: none;
  }
</style>
