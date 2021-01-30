<template>
  <a-modal
    title="编辑"
    width="40%"
    :maskClosable="false"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleSubmit"
    @cancel="handleCancel"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item label="用户名" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['UserName', { rules: [{ required: true, message: '请输入用户名' }] }]" />
        </a-form-item>
        <a-form-item label="密码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input
            type="password"
            autocomplete="false"
            placeholder="请输入密码"
            v-decorator="['newPwd', { rules: [{ required: false, message: '请输入密码' }] }]"
          ></a-input>
        </a-form-item>
        <a-form-item label="姓名" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['RealName', { rules: [{ required: true, message: '请输入用户名' }] }]" />
        </a-form-item>
        <a-form-item label="手机号码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['PhoneNum', { rules: [{ required: true, message: '请输入手机号码' }] }]" />
        </a-form-item>
        <a-form-item label="区" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['AreaId', { rules: [{ required: true, message: '请输入区' }] }]" />
        </a-form-item>
        <a-form-item label="性别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-radio-group v-decorator="['Sex', { rules: [{ required: true }], initialValue: 0 }]">
            <a-radio :value="0">女</a-radio>
            <a-radio :value="1">男</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item label="出生日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-date-picker
            style="width: 300px"
            format="YYYY-MM-DD"
            v-decorator="['Birthday', { rules: [{ required: false }] }]"
          />
        </a-form-item>
        <a-form-item label="所属部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-tree-select
            allowClear
            style="width: 300px"
            :dropdownStyle="{ maxHeight: '400px', overflow: 'auto' }"
            :treeData="DepartmentIdTreeData"
            placeholder="请选择部门"
            treeDefaultExpandAll
            v-decorator="['DepartmentId', { rules: [{ required: true, message: '请选择所属部门' }] }]"
          ></a-tree-select>
        </a-form-item>

        <a-form-item label="所属学校" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-tree-select
            allowClear
            showSearch
            treeNodeFilterProp="title"
            style="width: 300px"
            :dropdownStyle="{ maxHeight: '400px', overflow: 'auto' }"
            :treeData="SchoolTreeData"
            placeholder="请选所属学校"
            treeDefaultExpandAll
            v-decorator="['SchoolId']"
          ></a-tree-select>
        </a-form-item>

        <a-form-item label="学科" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-select
            allowClear
            mode="multiple"
            v-decorator="['SubjectIds', { rules: [{ required: true, message: '请选择学科' }] }]"
          >
            <a-select-option v-for="item in SubjectList" :key="item.Id">{{ item.Name }}</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="角色" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-select
            allowClear
            mode="multiple"
            v-decorator="['RoleIdList', { rules: [{ required: false }],initialValue:['1243404500610322432'] }]"
          >
            <a-select-option v-for="item in RoleOptionList" :key="item.Id">{{ item.RoleName }}</a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import moment from 'moment'
export default {
  props: {
    afterSubmit: {
      type: Function,
      default: null
    }
  },
  data() {
    return {
      form: this.$form.createForm(this),
      labelCol: { xs: { span: 24 }, sm: { span: 7 } },
      wrapperCol: { xs: { span: 24 }, sm: { span: 13 } },
      visible: false,
      confirmLoading: false,
      formFields: {},
      entity: {},
      DepartmentIdTreeData: [],
      SchoolTreeData: [],
      SubjectList: [],
      RoleOptionList: []
    }
  },
  methods: {
    add() {
      this.entity = {}
      this.visible = true
      this.form.resetFields()
      this.init()
    },
    edit(id) {
      this.visible = true

      this.$nextTick(() => {
        this.formFields = this.form.getFieldsValue()

        this.$http.post('/Base_Manage/Base_User/GetTheData', { id: id }).then(resJson => {
          this.entity = resJson.Data
          //this.entity.SubjectIds = []
          var setData = {}
          Object.keys(this.formFields).forEach(item => {
            setData[item] = this.entity[item]
          })
          if (setData['Birthday']) {
            setData['Birthday'] = moment(setData['Birthday'])
          }
          this.form.setFieldsValue(setData)
          this.init()
        })
      })

    },
    handleSubmit() {
      this.form.validateFields((errors, values) => {
        //校验成功
        if (!errors) {
          this.entity = Object.assign(this.entity, this.form.getFieldsValue())
          console.log(this.entity)

          this.entity['roleIdsJson'] = JSON.stringify(this.entity['RoleIdList'])
          debugger
          if (this.entity['Birthday']) {
            this.entity['Birthday'] = this.entity['Birthday'].format('YYYY-MM-DD')
          }
          this.confirmLoading = true
          this.$http.post('/Base_Manage/Base_User/SaveDataTeacher', this.entity).then(resJson => {
            this.confirmLoading = false

            if (resJson.Success) {
              this.$message.success('操作成功!')
              this.afterSubmit()
              this.visible = false
            } else {
              this.$message.error(resJson.Msg)
            }
          })
        }
      })
    },
    handleCancel() {
      this.visible = false
    },
    init() {
      this.$http.post('/Base_Manage/Base_Department/GetTreeDataList').then(resJson => {
        if (resJson.Success) {
          this.DepartmentIdTreeData = resJson.Data
        }
      })
      this.$http.post('/Base_Manage/Base_Role/GetDataList').then(resJson => {
        if (resJson.Success) {
          this.RoleOptionList = resJson.Data
        }
      })

      this.$http.post('/School/Exam_School/GetTreeDataList').then(resJson => {
        if (resJson.Success) {
          this.SchoolTreeData = resJson.Data
        }
      })

      this.$http.post('/Base_Manage/Base_DictData/GetListByParentId', { name: '学科' }).then(resJson => {
        if (resJson.Success) {
          this.SubjectList = resJson.Data
        }
      })
    }


  }
}
</script>
