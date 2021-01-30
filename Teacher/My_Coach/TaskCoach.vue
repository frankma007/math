<template>
  <div>
    <!-- 学生信息 -->
    <div class="student-info">
      <span>
        <a-avatar size="large" icon="user" class="fl"/>
        <div class="fl">
          <p>刘毅清</p>
          <p>高小一中</p>
        </div>
      </span>
      <span class="fg">
        <p>125</p>
        <p>总辅导数</p>
      </span>
      <span class="fg">
        <p>3</p>
        <p>近七天辅导数</p>
      </span>
    </div>
    <!-- 搜索内容 -->
    <div class="seacher-data clearfix">
      <div class="fl">
        <a-cascader
          :options="options"
          @change="onChange"
          placeholder="全部练习"
          style="marginRight: 30px;"
        />
        <a-select style="width: 220px; marginRight: 20px;" placeholder="所有日期" show-search @change="handleChange">
          <a-select-option value="lucy">
            近一周
          </a-select-option>
          <a-select-option value="一小节">
            近一个月
          </a-select-option>
        </a-select>
      </div>
      <!-- 直接搜索内容 -->
      <div class="seacher-data-info fg">
        <input class="input" type="text" placeholder="搜索提问内容" v-model="content" @keyup.enter="getIuputContent">
        <span @click="getIuputContent">搜索</span>
        <span @click="handleReset">重置</span>
      </div>
    </div>
    <!-- 辅导题目 -->
    <div class="coach-text">
      <div style="marginBottom: 20px;">
        <span class="title-num">1. &nbsp;</span>
        <span v-html="subject"></span>
      </div>
      <span v-for="item in option" :key="item.Id">
        <span>{{ item.Option }}. </span>
        <span v-html="item.Content"></span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      </span>
      <!-- 正确率统计 -->
      <div class="accuracy">
        <a-table
          :columns="columns"
          :data-source="data"
          bordered
          :pagination="pagination">
        </a-table>
        <!-- 正确人数统计 -->
        <p class="accuracy-num">正确21人 &nbsp;<a-progress :percent="60" :showInfo="showInfo" :strokeWidth="strokeWidth"/>
          &nbsp;错误14人&nbsp;&nbsp;&nbsp;&nbsp;
          <span @click="toSeeWhole">查看全部</span>
          <!-- <span class="open" @click="handleTable" >{{ packUp }}</span>&nbsp; -->
          <!-- <a-icon v-show="packUp === '展开详情'" type="down" />
          <a-icon v-show="packUp === '收起详情'" type="up" /> -->
          <transition name="slide-fade">
            <a-table
              style="margin-top: 15px"
              v-show="showAccuracy"
              :columns="column"
              :data-source="dataDetails"
              :pagination="pagination"
              bordered></a-table>
          </transition>
        </p>
      </div>
    </div>
    <!-- 辅导题目 -->
    <div class="coach-text">
      <div style="marginBottom: 20px;">
        <span class="title-num">1. &nbsp;</span>
        <span v-html="subject"></span>
      </div>
      <span v-for="item in option" :key="item.Id">
        <span>{{ item.Option }}. </span>
        <span v-html="item.Content"></span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      </span>
      <!-- 正确率统计 -->
      <div class="accuracy">
        <a-table
          :columns="columns"
          :data-source="data"
          bordered
          :pagination="pagination">
        </a-table>
        <!-- 正确人数统计 -->
        <p class="accuracy-num">正确21人 &nbsp;<a-progress :percent="60" :showInfo="showInfo" :strokeWidth="strokeWidth"/>
          &nbsp;错误14人&nbsp;&nbsp;&nbsp;&nbsp;
          <span class="open" @click="handleTable" >{{ packUp }}</span>&nbsp;
          <a-icon v-show="packUp === '展开详情'" type="down" />
          <a-icon v-show="packUp === '收起详情'" type="up" />
          <transition name="slide-fade">
            <a-table
              style="margin-top: 15px"
              v-show="showAccuracy"
              :columns="column"
              :data-source="dataDetails"
              :pagination="pagination"
              bordered></a-table>
          </transition>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import '@/utils/utils.less'

const columns = [
  {
    title: '统计',
    dataIndex: 'name',
    key: 'name',
    align: 'center'
  },
  {
    title: '我班',
    dataIndex: 'age',
    key: 'age',
    align: 'center'
  },
  {
    title: '我校',
    dataIndex: 'address',
    key: 'address',
    align: 'center'
  },
  {
    title: '全区',
    dataIndex: 'area',
    key: 'area',
    align: 'center'
  }
]
const data = [
  {
    key: '1',
    name: '平均正确率',
    age: '85%',
    address: '65%',
    area: '75%'
  }
]
const column = [
  {
    title: '选项',
    dataIndex: 'name',
    key: 'name',
    align: 'center'
  },
  {
    title: '人数',
    dataIndex: 'num',
    key: 'num',
    align: 'center'
  },
  {
    title: '学生',
    dataIndex: 'student',
    key: 'student',
    align: 'center'
  }
]
const dataDetails = [
  {
    key: '1',
    name: 'A',
    num: '6人',
    student: ['王小兰、', '易晓天、', '王诗怡、', '思敏韩、', '田宇琦']
  },
  {
    key: '2',
    name: 'B',
    num: '5人',
    student: ['田思丽、', '苟思明、', '王丽清、', '而小丽、', '钱忆锋']
  },
  {
    key: '3',
    name: 'C',
    num: '7人',
    student: ['吴思明、', '而思量、', '田十五、', '周箱梁、', '齐玉良、', '田士璐、', '吴提宇']
  }
]
// import fabric from '@/utils/fabric.min.js'
export default {
  created () {
    // 获取辅导题目
    this.getCoachText()
    // 获取辅导选项
    this.getCoachOptions()
  },
  data () {
    return {
      // 测试-----------
      column,
      dataDetails,
      options: [
        {
          value: '一年级',
          label: '一年级',
          children: [
            {
              label: '本学期',
              value: '本学期'
            },
            {
              value: '一10以内的数',
              label: '一10以内的数'
            },
            {
              value: '二10以内的数及加减法',
              label: '二10以内的数及加减法'
            }
          ]
        },
        {
          value: '二年级',
          label: '二年级',
          children: [
            {
              value: '一20以内的数及加减法',
              label: '一20以内的数及加减法'
            },
            {
              value: '二识别图形',
              label: '二识别图形'
            }
          ]
        }
      ],
      // 正确率行数据
      columns,
      // 正确率列数据
      data,
      // 测试-----------
      // 搜索框内容
      content: '',
      // 辅导题干
      subject: '',
      // 辅导选项
      option: [],
      // table分页
      pagination: false,
      // 显示进度条数值或图标
      showInfo: false,
      // 进度条宽度
      strokeWidth: 20,
      // 展开作答详情
      packUp: '展开详情',
      showAccuracy: false
    }
  },
  methods: {
    // 展开/收起 详情
    handleTable () {
      this.showAccuracy = !this.showAccuracy
      if (this.packUp === '展开详情') {
        this.packUp = '收起详情'
      } else {
        this.packUp = '展开详情'
      }
    },
    // 联级选择框
    onChange (value) {
      console.log(value)
    },
    // 选择框
    handleChange (value) {
      console.log(`selected ${value}`)
    },
    // 获取选择框内容
    getIuputContent () {
      console.log('搜索框内容---', this.content)
    },
    // 重置
    handleReset () {
      this.content = ''
    },
    // 获取辅导的题目
    getCoachText () {
      // 5804c11a-9309-42ec-962b-c22180ca7d51
      this.$http.post('/Paper/Exam_Item/GetTheData', {
        id: '5804c11a-9309-42ec-962b-c22180ca7d51'
      }).then(res => {
        this.subject = res.Data.Title
        console.log('辅导的题目---', this.subject)
      })
    },
    // 获取辅导选项
    getCoachOptions () {
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetOptionsByItemId', {
        itemId: '5804c11a-9309-42ec-962b-c22180ca7d51'
      }).then(res => {
        this.option = res.Data
        console.log('题目选项---', this.option)
      })
    },
    // 单题辅导
    toSeeWhole () {
      this.$router.push({ path: '/Teacher/My_Coach/SinglQeuestionCoach' })
    }
  }
}
</script>

<style lang="less" scoped>
  // 表格
  /deep/.ant-table-thead {
    /deep/ .ant-table-align-center{
      color: #fff;
      background-color: #68BB97;
    }
  }
   /deep/.ant-table-tbody tr:nth-child(odd) {
      background-color: #F4F4F4;
  }
  // 学生信息
  .student-info {
    width: 100%;
    height: 127px;
    margin-bottom: 30px;
    padding: 40px 50px 0 30px;
    border-radius: 5px;
    background-color: #fff;
    span:nth-child(3) {
      text-align: center;
    }
    span:nth-child(2) {
      text-align: center;
    }
    p {
      margin-left: 10px;
      margin-right: 10px;
      // text-align: center;
    }
  }
  // 搜索内容
  .seacher-data {
    margin-bottom: 25px;
    .seacher-data-info {
      margin-right: 80px;
      span {
        display: inline-block;
        width: 50px;
        height: 35px;
        line-height: 35px;
        margin-left: 10px;
        margin-right: 10px;
        text-align: center;
        border-radius: 5px;
        background-color: #ddd;
        cursor: pointer;
      }
      .input {
        width: 240px;
        height: 40px;
        text-indent: 20px;
        border-radius: 5px;
        border: 1px solid #ccc;
      }
      .input:focus {
        border-radius: 5px;
        border: 1px solid #00aaff;
        outline:none;
      }
    }
  }
  // 辅导题目
  .coach-text {
    margin-left: 20px;
    margin-bottom: 20px;
    background-color: #fff;
    padding: 15px 25px;
    border-radius: 5px;

    .title-num {
       font-size: 20px;
    }
    span:nth-child(2) {
      margin-bottom: 15px;
    }
  }
  // 正确率统计
  .accuracy {
    width: 600px;
    margin-bottom: 15px;
  }
  // 进度条
  .ant-progress-line {
    width: 350px;
  }
  .ant-progress-inner {
    background-color: #ddd;
  }
  // .accuracy-num /deep/ .ant-progress-inner {
  //   // background-color: red;
  // }
  // 展开按钮
  .accuracy-num {
    margin-top: 15px;
    margin-bottom: 15px;
    cursor: pointer;
  }
  .open {
    color: #68BB97;
  }
  .ant-progress-bg {
    background-color: #1890ff;
  }
</style>
