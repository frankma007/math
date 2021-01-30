<template>
  <a-card :bordered="false">
    <div class="table-operator">
      <a-button v-if="hasPerm('Base_User.Add')" type="primary" icon="plus" @click="hanldleAdd()">新建</a-button>
      <a-button
        v-if="hasPerm('Base_User.Delete')"
        type="primary"
        icon="minus"
        @click="handleDelete(selectedRowKeys)"
        :disabled="!hasSelected()"
        :loading="loading"
        >删除</a-button
      >
    </div>

    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="48">
          <a-col :md="4" :sm="24">
            <a-form-item label="搜索">
              <a-select allowClear v-model="queryParam.condition">
                <a-select-option key="PhoneNum">手机</a-select-option>
                <a-select-option key="UserName">用户名</a-select-option>
                <a-select-option key="RealName">姓名</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
           <a-col :md="4" :sm="24">
            <a-form-item>
               <a-input v-model="queryParam.keyword" placeholder="用户名/姓名" />
            </a-form-item>
          </a-col>
         

           <a-col :md="3" :sm="24">
            <a-form-item label="区">
             <a-select allowClear v-model="queryParam.areaid">
                <a-select-option key="1">徐汇区</a-select-option>
                <a-select-option key="2">黄浦区</a-select-option>
                <a-select-option key="3">浦东新区</a-select-option>
                <a-select-option key="4">长宁区</a-select-option>
                <a-select-option key="5">静安区</a-select-option>
                <a-select-option key="6">普陀区</a-select-option>
                <a-select-option key="7">虹口区</a-select-option>
                <a-select-option key="8">杨浦区</a-select-option>
                <a-select-option key="9">闵行区</a-select-option>
                <a-select-option key="10">宝山区</a-select-option>
                <a-select-option key="11">嘉定区</a-select-option>
                <a-select-option key="12">金山区</a-select-option>
                <a-select-option key="13">松江区</a-select-option>
                <a-select-option key="14">青浦区</a-select-option>
                <a-select-option key="15">奉贤区</a-select-option>
                <a-select-option key="16">崇明区</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

           <a-col :md="4" :sm="24">
            <a-form-item>
              <a-input v-model="queryParam.schoolName" placeholder="学校" />
            </a-form-item>
            </a-col>
          
          <a-col :md="6" :sm="24">
            <a-button type="primary" @click="getDataList">查询</a-button>
            <a-button style="margin-left: 8px" @click="() => (queryParam = {})">重置</a-button>
          </a-col>
        </a-row>
      </a-form>
    </div>
 
    <a-table
      ref="table"
      :columns="columns"
      :rowKey="row => row.Id"
      :dataSource="data"
      :pagination="pagination"
      :loading="loading"
      @change="handleTableChange"
      :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
      :bordered="true"
      size="small"
    >
      <span slot="action" slot-scope="text, record">
        <template>
          <template v-if="hasPerm('Base_User.Edit')">
            <a @click="handleEdit(record.Id)">编辑</a>
            <a-divider type="vertical" />
          </template>
          <a v-if="hasPerm('Base_User.Delete')" @click="handleDelete([record.Id])">删除</a>
        </template>
      </span>
    </a-table>

    <edit-form ref="editForm" :afterSubmit="getDataList"></edit-form>
  </a-card>
</template>

<script>
import EditForm from './EditForm'

const columns = [
  { title: '用户名', dataIndex: 'UserName', width: '10%' },
  { title: '手机号码', dataIndex: 'PhoneNum', width: '10%' },
  { title: '姓名', dataIndex: 'RealName', width: '10%' },
  { title: '性别', dataIndex: 'SexText', width: '5%' },
  { title: '出生日期', dataIndex: 'BirthdayText', width: '10%' },
  { title: '所属部门', dataIndex: 'DepartmentName', width: '10%' },
  { title: '所属角色', dataIndex: 'RoleNames', width: '10%' },
  { title: '学校', dataIndex: 'SchoolName', width: '10%' },
  { title: '操作', dataIndex: 'action', scopedSlots: { customRender: 'action' } }
]

export default {
  components: {
    EditForm
  },
  mounted() {
    this.getDataList()
  },
  data() {
    return {
      data: [],
      pagination: {
        current: 1,
        pageSize: 50,
        showTotal: (total, range) => `总数:${total} 当前:${range[0]}-${range[1]}`
      },
      filters: {},
      sorter: { field: 'Id', order: 'asc' },
      loading: false,
      columns,
      queryParam: {},
      visible: false,
      selectedRowKeys: []
    }
  },
  methods: {
    handleTableChange(pagination, filters, sorter) {
      this.pagination = { ...pagination }
      this.filters = { ...filters }
      this.sorter = { ...sorter }
      this.getDataList()
    },
    getDataList() {
      this.selectedRowKeys = []
      
      this.loading = true
      this.$http
        .post('/Base_Manage/Base_User/GetTeacherDataList', {
          PageIndex: this.pagination.current,
          PageRows: this.pagination.pageSize,
          SortField: this.sorter.field || 'Id',
          SortType: this.sorter.order,
          ...this.queryParam,
          ...this.filters
        })
        .then(resJson => {
          this.loading = false
          this.data = resJson.Data
          const pagination = { ...this.pagination }
          pagination.total = resJson.Total
          this.pagination = pagination
        })
    },
    onSelectChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    hasSelected() {
      return this.selectedRowKeys.length > 0
    },
    hanldleAdd() {
      this.$refs.editForm.add()
    },
    handleEdit(id) {
      this.$refs.editForm.edit(id)
    },
    handleDelete(ids) {
      var thisObj = this
      this.$confirm({
        title: '确认删除吗?',
        onOk() {
          return new Promise((resolve, reject) => {
            thisObj.submitDelete(ids, resolve, reject)
          }).catch(() => console.log('Oops errors!'))
        }
      })
    },
    submitDelete(ids, resolve, reject) {
      this.$http.post('/Base_Manage/Base_User/DeleteData', { ids: JSON.stringify(ids) }).then(resJson => {
        resolve()

        if (resJson.Success) {
          this.$message.success('操作成功!')

          this.getDataList()
        } else {
          this.$message.error(resJson.Msg)
        }
      })
    }
  }
}
</script>