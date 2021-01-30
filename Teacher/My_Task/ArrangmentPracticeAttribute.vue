<template>
  <div>
    <div v-if="isLoading">
      <a-spin tip="Loading...">
        <div class="spin-content">
          <!-- 我的描述文案是自定义的。。。 -->
        </div>
      </a-spin>
    </div>

    <div v-if="isLoadingInfor">
      <p class="attribute">填写练习属性信息</p>
      <a-form :form="form" :label-col="{ span: 5 }" :wrapper-col="{ span: 8 }" @submit="handleSubmit">
        <a-row :gutter="12">
          <a-col :span="10">
            <a-form-item label="所属年级">
              <!-- initialValue: defalutGrade -->
              <a-select
                v-decorator="[
                  'grade',
                  { rules: [{ required: true, message: '必选' }],initialValue: defalutGrade},
                ]"
                placeholder="年级选择"
                @change="handleSelectChangeGrade"
              >
                <a-select-option
                  v-for="item in gradeInfor"
                  :key="item.Id"
                  :value="item.Id + '' + item.Gradename"
                >{{ item.Gradename }}</a-select-option>
              </a-select>
            </a-form-item>
            <a-form-item class="level" label="练习难度">
              <a-rate
                @change="starChange"
                style="height: 32px"
                v-decorator="['Level', { rules: [{ required: true, message: '必选' }] }]"
              ></a-rate>
            </a-form-item>
          </a-col>
          <a-col :span="10">
            <a-form-item label="知识点">
              <a-tree-select
                allowClear
                showSearch
                v-decorator="['knowledge', { rules: [{ required: true, message: '必选' }]}]"
                :tree-data="options"
                treeNodeFilterProp="title"
                @change="onChange"
                placeholder="知识点选择"
                initialValue=""
              >
              </a-tree-select>
            </a-form-item>
            <a-form-item label="出卷人">
              <!-- <a-tree-select
              allowClear
              showSearch
              treeNodeFilterProp="title"
              :dropdownStyle="{ maxHeight: '400px', overflow: 'auto' }"
              :treeData="UserTreeData"
              placeholder="选择出卷人"
              treeDefaultExpandAll
              v-decorator="['ImportUserId', { rules: [{ required: true,message: '必选' }], initialValue: userId }]"
              @change="CreateUserChange"
            ></a-tree-select> -->
              <a-input
                v-decorator="[
                  'ImportUserId',
                  { rules: [{ required: true, message: '必选' }]},
                ]"
                defaultlValue="defalutName"
              ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="10">
            <a-form-item label="练习名称">
              <a-input
                v-decorator="[
                  'name',
                  { rules: [{ required: true, message: '必选' }] },
                ]"
              ></a-input>
            </a-form-item>
          <!-- <a-form-item label="做卷开始时间">
             <a-date-picker
              :showTime="{ format: 'HH:mm' }"
              format="YYYY-MM-DD HH:mm"
              @change="TimeChange"
                v-decorator="[
                'time',
                { rules: [{ required: true, message: '必选' }] },
              ]">

            </a-date-picker>
          </a-form-item>-->
          </a-col>
        </a-row>
      </a-form>
      <p class="handle-btn">
        <span @click="handleBack">返回</span>
        <span @click="handleSubmit">确定</span>
      </p>
    </div>
    <footerLogo></footerLogo>
  </div>
</template>

<script>
import footerLogo from '@/components/XuHuiFooter/FooterLogo'
export default {
  components: {
    footerLogo
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    // 获取老师对应班级
    this.getTeacherClassList()
    // 获取出卷人
    // this.getUsers()
    // 获取教师的信息
    this.getTeacherList()
  },
  watch: {
    $route () {
      if (this.$route.fullPath === '/Teacher/My_Task/ArrangmentPracticeAttribute') {
        // 获取老师对应班级
        this.getTeacherClassList()
        // 获取教师的信息
        this.getTeacherList()
      }
    }
  },
  data () {
    return {
      // 老师id
      userId: '',
      // 默认年级
      defalutGrade: '',
      form: this.$form.createForm(this),
      // 知识点
      options: [],
      optionsList: [],
      // 所属年级
      gradeInfor: [],
      // 出卷人
      UserTreeData: [],
      AnswerEndDateTime: '',
      defalutName: '',
      // 确定效果
      isLoading: false,
      isLoadingInfor: true,
      // 知识点名称
      KnowledgeName: '',
      // 年级名称
      gradeName: ''
    }
  },
  methods: {
    // 获取教师的信息
    getTeacherList () {
      this.$uwonhttp.post('/ManagerPersonalsController/ManagerPersonals/GetTeacherInfo', { userId: this.userId }).then(res => {
        console.log('教师的信息---', res)
        this.defalutGrade = res.data.Data.Grade
        this.defalutGradeId = res.data.Data.Grade.slice(0, 1)
        this.gradeName = res.data.Data.Grade.slice(1)
        console.log('年级的名称', this.gradeName)
        this.defalutName = res.data.Data.TrueName
        this.form.setFieldsValue({
          ImportUserId: this.defalutName
        })
        console.log(this.defalutGrade)
        this.getGradeKnowledge(this.defalutGradeId)
      })
    },
    handleSubmit (e) {
      e.preventDefault()
      const that = this
      this.form.validateFields((err, values) => {
        if (!err) {
          console.log('Received values of form: ', values)
          this.isLoading = true
          this.isLoadingInfor = false
          const gradeId = values.grade.slice(0, 1)
          console.log('============', gradeId)
          this.$http
            .post('/Paper/Exam_Item/AddPaper', {
              Title: values.name,
              CreatorId: that.userId,
              GradeId: gradeId,
              DifficultyStar: values.Level,
              ChapterId: values.knowledge,
              PaperDateTime: this.AnswerEndDateTime
            })
            .then((res) => {
              console.log(res)
              if (res.Success) {
                this.$router.push({
                  path: '/Teacher/My_Task/ArrangmentTaskBasics',
                  query: {
                    paperId: res.Data,
                    name: values.name,
                    level: values.Level,
                    grade: gradeId,
                    knowledge: values.knowledge,
                    KnowledgeName: this.KnowledgeName,
                    gradeName: this.gradeName
                  }
                })
                this.isLoading = false
                this.isLoadingInfor = true
              }
            })
        }
      })
    },
    TimeChange (data, dataString) {
      this.AnswerEndDateTime = dataString
      console.log('截止作答时间---------' + this.AnswerEndDateTime)
    },
    // 获取出卷人
    getUsers () {
      this.$http.post('/Base_Manage/Base_User/GetTreeDataListBySchoolId', { schoolId: null }).then((resJson) => {
        console.log('出卷人---', resJson.Data)
        if (resJson.Success) {
          this.UserTreeData = resJson.Data
        }
      })
    },
    // 选择出卷人
    CreateUserChange (value) {
      console.log('选择出卷人---', value)
    },
    handleSelectChangeGrade (value) {
      console.log('年级的id---', value)
      // this.options = ''
      // 获取年级知识点
      const id = value[-1, 0]
      this.gradeName = value.slice(1)
      console.log('年级的名称---', this.gradeName)
      // console.log('年级的id---', id)
      this.getGradeKnowledge(id)
    },
    // 获取年级知识点
    getGradeKnowledge (id) {
      console.log('知识点=======', id)
      this.$http
        .post('/Paper/Exam_SubjectChapter/GetTreeDataListAsyncPaper', { gradeComb: id, subjectId: '2' })
        .then((res) => {
          console.log('年级对应知识点---', res)
          this.options = ''
          this.options = res.Data
          this.optionsList = this.options.map(item => {
            item.disabled = true
            return item
          })
          console.log(this.options)
        })
    },
    // 获取年级
    getTeacherClassList () {
      // this.$http.post('/Base_Manage/Exam_ClassGradeConfig/GetTreeDataList').then((res) => {
      //   console.log('老师对应班级---', res)
      //   this.gradeInfor = res.Data
      //   console.log('老师对应班级---', this.gradeInfor)
      // })
      this.$uwonhttp.get('/Class/TeacherClassManager/GetGradeList').then(res => {
        console.log('老师对应年级---', res)
        this.gradeInfor = res.data.Data
      })
    },
    // 知识点选择
    onChange (value, label, extra) {
      console.log(value)
      console.log(label)
      this.KnowledgeName = label[0]
      this.form.setFieldsValue({
        name: label[0]
      })
    },
    // 难度选择
    starChange (value) {
      console.log(value)
    },
    // 返回
    handleBack () {
      this.$router.go(-1)
    }
  }
}
</script>

<style lang="less" scoped>
.level {
  /deep/.ant-form-item-control {
    height: 52px;
    // text-indent: 5px;
    padding-left: 6px;
    border: 1px solid #ccc;
    border-radius: 5px;
    line-height: 42px;
    background-color: #fff;
  }
}
/deep/.ant-select-selection-selected-value {
    display: block;
    opacity: 1;
    width: 349px;
    line-height: 52px;
    height: 52px;
    font-size: 16px;
}
/deep/.ant-select-selection__rendered {
  width: 349px;
    height: 52px;
}
/deep/.ant-select-selection--single {
  width: 349px;
  height: 52px;
}
.ant-form-item-control .has-success {
  width: 349px;
  height: 52px;
}
/deep/.ant-form-item-control {
  width: 349px;
  height: 52px;
}

.ant-input {
  height: 52px;
}
.attribute {
  padding-left: 50px;
  font-size: 20px;
  font-weight: 700;
  color: #4D5753;
}
/deep/.ant-select-dropdown-menu-item {
  background-color: #68BB97;
}
//
.handle-btn {
  position: absolute;
  right: 100px;
  bottom: 100px;
  span {
    display: inline-block;
    width: 114px;
    height: 60px;
    font-size: 16px;
    line-height: 60px;
    text-align: center;
    border-radius: 5px;
    cursor: pointer;
  }
  span:nth-child(1) {
    margin-right: 48px;
    background-color: #fff;
  }
  span:nth-child(2) {
    color: #fff;
    background-color: #68bb97;
  }
}
</style>
