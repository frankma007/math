<template>
  <div>
    <div v-if="IsVip == '0'" class="noLack">
      <img src="@/assets/weekly/none.png" alt />
      <p>您的学校暂未开放该功能</p>
    </div>

    <div class="wrapper" v-else>
      <div class="head">
        <div class="select-info">
          <h3>学校周报</h3>
          <!-- <span class="desc"> {{ SummaryData.WeeklyTime }} </span> -->
          <div v-if="isHistoryLoading" class="select-box week-time"></div>
          <a-select class="select-box week-time" v-model="WeeklyId" @change="changeWeekly" v-else>
            <a-select-option v-for="item in historyList" :key="item.Id" :value="item.Id">
              {{ item.WeeklyTime }}
            </a-select-option>
          </a-select>
          <div v-if="isPracticeLoading" class="select-box"></div>
          <a-select class="select-box" v-model="PracticeId" @change="changePractice" v-else>
            <a-select-option v-for="item in practiceList" :key="item.PracticeId" :value="item.PracticeId">
              {{ item.PracticeName }}
            </a-select-option>
          </a-select>
          <div class="select-box" v-if="isClassLoading">
            <a-spin size="small" />
          </div>
          <!-- changeGrade -->
          <a-select class="select-box" v-model="gradeId" @change="changeGrade" v-else>
            <a-select-option v-for="item in gradeList" :key="item.Id" :value="item.Id">
              {{ item.Gradename }}
            </a-select-option>
          </a-select>
        </div>
        <div class="history-info" @click="toHistoryWeekly()">
          <img class="img" src="@/assets/weekly/weekly.png" alt="" /> 历史周报
        </div>
      </div>

      <div class="left-mod container">
        <div class="numb-info" v-if="isSummaryLoading">
          <div class="example">
            <a-spin />
          </div>
        </div>
        <div class="numb-info" v-else>
          <span class="big"> 推送次数 </span>
          <span>{{ SummaryData.PracticeCount }}次区练习 </span>

          <a-divider type="vertical" />
          <span class="big"> 完成率 </span>

          <span>校{{ SummaryData.SchoolFinish }}%</span>
          <span v-show="PracticeId == '1'">区{{ SummaryData.AreaFinish }}%</span>
          <a-divider type="vertical" />
          <span class="big"> 正确率 </span>

          <span>校{{ SummaryData.SchoolAccuracy }}%</span>
          <span v-show="PracticeId == '1'">区{{ SummaryData.AreaAccuracy }}%</span>

          <a-divider type="vertical" />
          <span class="big"> 平均用时 </span>

          <span> 校{{ SummaryData.SchoolAvgDoTime }} </span>
          <span v-show="PracticeId == '1'"> 区{{ SummaryData.AreaAvgDoTime }} </span>
        </div>
        <div class="summary-info">
          <h3>本周总结</h3>
          <div class="desc" v-if="isSummaryLoading">
            <div class="example">
              <a-spin />
            </div>
          </div>
          <div class="desc" v-else>
            <p v-show="PracticeId == '1'">
              学校本周共推送<span class="red">{{ SummaryData.PracticeCount }}次</span>区级练习，班级完成率<span
                :class="{
                  red: SummaryData.FinishContent == '低于',
                  yellow: SummaryData.FinishContent == '持平',
                  green: SummaryData.FinishContent == '优于',
                }"
                >{{ SummaryData.FinishContent }}</span
              >全区，正确率<span
                :class="{
                  red: SummaryData.AccuracyContent == '低于',
                  yellow: SummaryData.AccuracyContent == '持平',
                  green: SummaryData.AccuracyContent == '优于',
                }"
                >{{ SummaryData.AccuracyContent }}</span
              >全区，平均用时<span
                :class="{
                  red: SummaryData.DoTimeContent == '慢于',
                  yellow: SummaryData.DoTimeContent == '持平',
                  green: SummaryData.DoTimeContent == '快于',
                }"
                >{{ SummaryData.DoTimeContent }}</span
              >全区。
            </p>
            <p v-show="PracticeId == '2'">
              学校本周共推送<span class="red">{{ SummaryData.PracticeCount }}次</span>校级练习， 整体完成率为{{
                SummaryData.SchoolFinish
              }}%，整体正确率为{{ SummaryData.SchoolAccuracy }}%，平均用时为{{ SummaryData.SchoolAvgDoTime }}。
            </p>
            <p>
              学校周报所有推送练习中，成绩优秀的班级<span class="green">{{ SummaryData.ExcellentCount }}个</span
              >，成绩须努力的班级<span class="red">{{ SummaryData.StriveCount }}个</span>，优势知识点<span class="green"
                >{{ SummaryData.GoodKnowledge }}个</span
              >，薄弱知识点<span class="red">{{ SummaryData.WeakKnowledge }}个</span>， 发现
              <span v-if="isWeekLoading">
                <a-spin size="small" />
              </span>
              <span class="red" v-else> {{ weekHighList.length }}道</span>高频错题。
            </p>
          </div>
        </div>
        <div class="situation-info">
          <h3>做题情况分析</h3>
          <a-row
            :gutter="{ xl: 16, xxl: 16 }"
            type="flex"
            justify="space-between"
            align="middle"
            class="content-box ant-col-xl-24 ant-col-xxl-24"
          >
            <a-col :xl="12" :xxl="8" class="circle-info">
              <h4>练习知识点分布</h4>
              <div class="chart" v-if="isTwoChart">
                <div class="example">
                  <a-spin />
                </div>
              </div>
              <Annular
                v-else-if="KnowledgesList"
                :key="TwoChartKey"
                :list="KnowledgesList"
                @ev_getAnnularItem="ev_getAnnularItem"
                class="chart"
              >
              </Annular>
              <div v-else class="chart">
                <div class="example">
                  <a-spin />
                </div>
              </div>
            </a-col>

            <a-col :xl="12" :xxl="16" class="circle-info">
              <h4>做题时段分布</h4>
              <div v-if="isTwoChart" class="chart">
                <div class="example">
                  <a-spin />
                </div>
              </div>
              <Bar
                v-else-if="DoItemTimeIntervalsList"
                :key="TwoChartKey"
                @ev_getBarItem="ev_getBarItem"
                :list="DoItemTimeIntervalsList"
                class="chart"
              ></Bar>
              <div v-else class="chart">
                <div class="example"><a-empty /></div>
              </div>
            </a-col>
          </a-row>
        </div>
      </div>
      <!-- 右边部分 -->
      <div class="anlysis-mod container">
        <h3>校学情分析</h3>
        <a-row
          :gutter="{ xl: 16, xxl: 16 }"
          type="flex"
          justify="space-between"
          align="middle"
          class="content-box ant-col-xl-24 ant-col-xxl-24"
        >
          <a-col :xl="12" :xxl="10" class="item">
            <div class="title-info">
              <h4>
                练习整体详情
                <a-tooltip placement="right" title="正确率：(班/区) 平均用时：(班/区)" :auto-adjust-overflow="false">
                  <a-icon class="red" type="question-circle" />
                </a-tooltip>
              </h4>
            </div>

            <a-table
              class="table-info-whole"
              rowKey="PaperTitle"
              :loading="isWholeLoading"
              :columns="columnsWhole"
              :data-source="WholeDetailList"
              :pagination="false"
            >
            </a-table>
          </a-col>
          <a-col :xl="12" :xxl="14" class="item">
            <div class="title-info">
              <h4>练习成绩趋势分析</h4>
            </div>
            <div class="chart" v-if="isLineChartLoading">
              <div class="example">
                <a-spin />
              </div>
            </div>

            <Lines
              v-else-if="lineChartList"
              :key="lineKey"
              :lineChartList="lineChartList"
              :PracticeId="PracticeId"
              class="chart"
            ></Lines>
            <div v-else class="chart">
              <a-empty />
            </div>
          </a-col>
        </a-row>

        <!-- 第二段1 -->

        <a-row
          :gutter="{ xl: 16, xxl: 16 }"
          type="flex"
          justify="space-between"
          align="middle"
          class="content-box ant-col-xl-24 ant-col-xxl-24"
        >
          <a-col :xl="12" :xxl="12" class="item">
            <div class="title-info">
              <h4>各班正确率对比分析</h4>
            </div>
            <div class="chart" v-if="isBarLineLoading">
              <div class="example">
                <a-spin />
              </div>
            </div>

            <BarLine v-else-if="barLineList" :key="barLineKey" :list="barLineList" class="chart"></BarLine>
            <div v-else class="chart">
              <div class="example"><a-empty /></div>
            </div>
          </a-col>
          <!-- 222 -->
          <a-col :xl="12" :xxl="12" class="item">
            <div class="title-info">
              <h4>各教师正确率对比分析</h4>
            </div>
            <div class="chart" v-if="isTeacherBarLineLoading">
              <div class="example">
                <a-spin />
              </div>
            </div>

            <BarLine
              v-else-if="TeacherBarLineList"
              :key="TeacherBarLineKey"
              :list="TeacherBarLineList"
              class="chart"
            ></BarLine>
            <div v-else class="chart">
              <div class="example"><a-empty /></div>
            </div>
          </a-col>
        </a-row>
        <!--           <div class="titles">本周各班正确率对比分析</div>
          <div class="anlysis-chart" v-if="isBarLineLoading">
            <div class="example">
              <a-spin />
            </div>
          </div>
          <BarLine v-else-if="barLineList" :key="barLineKey" :list="barLineList" class="anlysis-chart"></BarLine>
          <div v-else class="anlysis-chart">
            <a-empty />
          </div> -->
      </div>

      <div class="bottom-info">
        <h3>学科能力分析</h3>
        <a-row
          :gutter="{ xl: 16, xxl: 16 }"
          type="flex"
          justify="space-between"
          align="middle"
          class="bottomList ant-col-xl-24 ant-col-xxl-24"
        >
          <a-col :xl="12" :xxl="6" class="item">
            <div class="title-info">
              <h4>
                知识点掌握情况
                <a-tooltip
                  placement="right"
                  title="薄弱知识点为班级对应正确率小于校正确率"
                  :auto-adjust-overflow="false"
                >
                  <a-icon class="red" type="question-circle" />
                </a-tooltip>
              </h4>
            </div>

            <a-table
              class="table-info"
              rowKey="ChapterName"
              :loading="isKnowledgeLoading"
              :columns="knowledgeColumns"
              :data-source="knowledgePointList"
              :pagination="false"
            >
              <template slot="Accuracy" slot-scope="text"> {{ text }}% </template>
              <template slot="action" slot-scope="text, res">
                <a @click="topop(res.ChapterId, '1')" href="javascript:;"
                  >{{ text }}个班级落后<a-icon type="right"
                /></a>
              </template>
            </a-table>
          </a-col>
          <!-- 第二个 -->
          <a-col :xl="12" :xxl="6" class="item">
            <div class="title-info">
              <h4>
                学习水平分析
                <a-tooltip
                  placement="right"
                  title="薄弱学习水平为班级对应正确率小于校正确率"
                  :auto-adjust-overflow="false"
                >
                  <a-icon class="red" type="question-circle" />
                </a-tooltip>
              </h4>
            </div>

            <a-table
              class="table-info"
              rowKey="ExamineName"
              :loading="isLevelLoading"
              :columns="levelColumns"
              :data-source="learnLevel"
              :pagination="false"
            >
              <template slot="level" slot-scope="text, res">
                {{ text }}
                <span class="red" v-if="res.IsWeak">[薄弱]</span>
              </template>
              <template slot="Accuracy" slot-scope="text"> {{ text }}% </template>
              <template slot="action" slot-scope="text, res">
                <a @click="topop(res.Examine, '3')" href="javascript:;">{{ text }}个班级落后<a-icon type="right" /></a>
              </template>
            </a-table>
          </a-col>

          <!-- 三 -->
          <a-col :xl="12" :xxl="6" class="item">
            <div class="title-info">
              <h4>
                题型版块分析
                <a-tooltip
                  placement="right"
                  title="薄弱题型版块为班级对应正确率小于校正确率"
                  :auto-adjust-overflow="false"
                >
                  <a-icon class="red" type="question-circle" />
                </a-tooltip>
              </h4>
              <!-- <div class="desc">
                <a-icon class="red" type="question-circle" />
                薄弱题型版块为班级对应正确率小于校正确率
              </div> -->
            </div>

            <a-table
              class="table-info"
              :loading="isQuestionLoading"
              rowKey="PlateName"
              :columns="questionColumns"
              :data-source="questionList"
              :pagination="false"
            >
              <template slot="level" slot-scope="text, res">
                {{ text }}
                <span class="red" v-if="res.IsWeak">[薄弱]</span>
              </template>

              <template slot="Accuracy" slot-scope="text"> {{ text }}% </template>
              <template slot="action" slot-scope="text, res">
                <a @click="topop(res.PlateId, '2')" href="javascript:;">{{ text }}个班级落后<a-icon type="right" /></a>
              </template>
            </a-table>
          </a-col>

          <!-- 四 -->
          <a-col :xl="12" :xxl="6" class="item">
            <div class="title-info">
              <h4>
                班级高频错题
                <a-tooltip placement="right" title="高频错题为班级正确率低于80%的题目" :auto-adjust-overflow="false">
                  <a-icon class="red" type="question-circle" />
                </a-tooltip>
              </h4>
            </div>

            <div class="chart" v-if="isWeekLoading">
              <div class="example">
                <a-spin />
              </div>
            </div>
            <div class="chart" v-else-if="weekHighList.length">
              <div class="box-container">
                <div class="box" v-for="(item, index) in weekHighList" :key="index">
                  <AnnularSmall
                    @ev_getSmallAnnular="ev_getSmallAnnular"
                    :key="item.ItemId"
                    :value="item"
                    class="chart-area"
                  >
                  </AnnularSmall>

                  <p>第{{ index + 1 }}题</p>
                </div>
              </div>
            </div>
            <div class="chart" v-else-if="!weekHighList.length">
              <div class="example">
                <a-empty />
              </div>
            </div>
          </a-col>
        </a-row>
      </div>
      <transition
        enter-active-class="animate__animated animate__fadeIn"
        leave-active-class="animate__animated animate__fadeOut"
      >
        <PopUp v-if="isPopUp">
          <template v-slot:close>
            <div class="close" @click.stop="isPopUp = false">
              <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
            </div>
          </template>
          <template v-slot:title>
            <div class="title">
              <h6>{{ popTableData.Name }}</h6>
              <div class="desc">
                <span class="big">平均正确率{{ popTableData.AvgAccuracy }}%</span>
                {{ popTableData.ExcellentCnt }}个班级<span class="green">优秀</span>，
                {{ popTableData.StriveCnt }}个班级<span class="error">落后</span>
              </div>
            </div>
          </template>
          <template v-slot:table>
            <a-table
              class="table-info"
              bordered
              rowKey="StudentName"
              :loading="isPopLoading"
              :columns="columns_popTable"
              :data-source="popTableData.List"
              :pagination="false"
            >
              <template slot="Accuracy" slot-scope="text"> {{ text }}% </template>
              <template slot="MasterCondition" slot-scope="text">
                <span :class="{ yellow: text == '落后' }">{{ text }}</span>
              </template>
            </a-table>
          </template>
        </PopUp>
      </transition>
      <transition
        enter-active-class="animate__animated animate__fadeIn"
        leave-active-class="animate__animated animate__fadeOut"
      >
        <PopUp v-if="isPopTime">
          <template v-slot:close>
            <div class="close" @click.stop="isPopTime = false">
              <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
            </div>
          </template>
          <template v-slot:title>
            <div class="title">
              <h6>{{ PopDataTitle }}</h6>
            </div>
          </template>
          <template v-slot:table>
            <a-table
              class="table-info"
              bordered
              :columns="columns"
              :data-source="TimeDetailList"
              rowKey="ClassName"
              :pagination="false"
            >
            </a-table>
          </template>
        </PopUp>
      </transition>
      <transition
        enter-active-class="animate__animated animate__fadeIn"
        leave-active-class="animate__animated animate__fadeOut"
      >
        <pop-list v-if="isPopList">
          <template v-slot:close>
            <div class="close" @click.stop="isPopList = false">
              <img src="@/assets/teacher/IntelligentCorrection/close.png" alt />
            </div>
          </template>
          <!-- AnnularParams 循环数据 入参  -->
          <template v-slot:title>
            <div class="title">
              <h6>{{ popAnnularTitle }}</h6>
            </div>
          </template>
          <template v-slot:list>
            <ul class="list" v-if="isPopAnnularLoading">
              <div class="example">
                <a-spin />
              </div>
            </ul>
            <ul class="list" v-else>
              <li class="item" v-for="(item, index) in PopAnnularList" :key="index">
                <div class="state" v-show="item.PaperType != 0 && item.EndTime !== '' && item.TipText !== '待发布'">
                  {{ item.PracticeStatesText }}
                </div>
                <div class="content-mod">
                  <div class="top-info">
                    {{ item.PaperTitle }}
                    <div class="btn">{{ item.TipText }}</div>
                  </div>

                  <div class="content">
                    <div class="info">
                      <p>所属单元：{{ item.ChapterName }}</p>
                      <p>
                        {{ item.GradeName }} <span>出题人：{{ item.TrueName }}</span
                        ><img src="@/assets/weekly/icon_play.png" alt />
                      </p>
                    </div>
                    <a-rate class="rate" :default-value="item.DifficultyStar" disabled />
                    <div class="Progress">
                      <div class="items">
                        <!-- :format="percent => `${percent}/100`" -->
                        <a-progress
                          type="circle"
                          :percent="(item.DoCount / item.TotalCount) * 100"
                          :width="49"
                          :format="(percent) => `${item.DoCount}/${item.TotalCount}`"
                        />
                        <p>做卷人数</p>
                      </div>
                      <div class="items">
                        <a-progress type="circle" :percent="Number(item.Accuracy)" :width="49" />
                        <p>正确率</p>
                      </div>
                    </div>
                  </div>

                  <div class="bottom-info">
                    推送时间：{{ item.CreatedTime }} <a-divider type="vertical" /> 截止时间：<span
                      v-show="!item.EndTime"
                      >--</span
                    >
                    <span v-show="item.EndTime">{{ item.EndTime }}</span>
                  </div>
                </div>
                <div class="right-mod">
                  <div class="text">{{ item.ClassName }}</div>
                  <div
                    class="btn"
                    @click="toCheck(item.PaId, item.ChapterId)"
                    v-if="
                      item.PracticeStates === 1 ||
                      item.PracticeStates === 3 ||
                      item.TipText === '已完成' ||
                      item.TipText === '已发布'
                    "
                  >
                    检查作业
                  </div>
                </div>
              </li>
            </ul>
          </template>
        </pop-list>
      </transition>
    </div>
  </div>
</template>

<script>
// import echarts from 'echarts'
import Annular from './components/Annular.vue'
import AnnularSmall from './components/AnnularSmall'
import Bar from './components/Bar'
import Lines from './components/Line'
import BarLine from './components/BarLine'
import PopUp from './components/PopUp'
import PopList from './components/PopList'

const questionColumns = [
  {
    title: '题型版块',
    dataIndex: 'PlateName',
    key: 'PlateName',
    scopedSlots: { customRender: 'level' },
    width: '30%',
    ellipsis: true,
  },
  {
    title: '正确率',
    dataIndex: 'Accuracy',
    key: 'Accuracy',
    scopedSlots: { customRender: 'Accuracy' },
  },
  {
    title: '标准差',
    dataIndex: 'StandardDeviation',
    key: 'StandardDeviation',
    ellipsis: true,
  },
  {
    title: '班级情况',
    dataIndex: 'StriveCnt',
    key: 'StriveCnt',
    width: '26%',
    ellipsis: true,
    scopedSlots: { customRender: 'action' },
  },
]

const levelColumns = [
  {
    title: '学习水平',
    dataIndex: 'ExamineName',
    key: 'ExamineName',
    scopedSlots: { customRender: 'level' },
    width: '30%',
    ellipsis: true,
  },
  {
    title: '正确率',
    dataIndex: 'Accuracy',
    key: 'Accuracy',
    scopedSlots: { customRender: 'Accuracy' },
  },
  {
    title: '标准差',
    dataIndex: 'StandardDeviation',
    key: 'StandardDeviation',
    ellipsis: true,
  },
  {
    title: '班级情况',
    dataIndex: 'StriveCnt',
    key: 'StriveCnt',

    ellipsis: true,
    scopedSlots: { customRender: 'action' },
    width: '27%',
  },
]
const knowledgeColumns = [
  {
    title: '知识点',
    dataIndex: 'ChapterName',
    key: 'ChapterName',
    width: '30%',
    ellipsis: true,
  },
  {
    title: '正确率',
    dataIndex: 'Accuracy',
    key: 'Accuracy',

    scopedSlots: { customRender: 'Accuracy' },
    ellipsis: true,
  },
  {
    title: '标准差',
    dataIndex: 'StandardDeviation',
    key: 'StandardDeviation',
    ellipsis: true,
  },
  {
    title: '班级情况',
    dataIndex: 'StriveCnt',
    key: 'StriveCnt',
    width: '27%',
    ellipsis: true,
    scopedSlots: { customRender: 'action' },
  },
]

const columns = [
  {
    title: '班级',
    dataIndex: 'ClassName',
    key: 'ClassName',
  },
  {
    title: '做题人数',
    dataIndex: 'DoCount',
    key: 'DoCount',
  },
]
const columns_popTable = [
  {
    title: '班级名称',
    dataIndex: 'ClassName',
    key: 'ClassName',
  },
  {
    title: '正确率',
    dataIndex: 'Accuracy',
    key: 'Accuracy',
    scopedSlots: { customRender: 'Accuracy' },
  },
  {
    title: '掌握情况',
    dataIndex: 'MasterCondition',
    key: 'MasterCondition',
    scopedSlots: { customRender: 'MasterCondition' },
  },
]

const columnsWhole = [
  {
    title: '试卷名',
    dataIndex: 'PaperTitle',
    key: 'PaperTitle',
    width: 100,
    ellipsis: true,
  },
  {
    title: '正确率',
    dataIndex: 'Accuracy',
    // key: 'Accuracy',
    ellipsis: true,
  },
  {
    title: '平均用时',
    dataIndex: 'DoTime',
    // key: 'DoTime',
    ellipsis: true,
  },
  {
    title: '做题人数',
    dataIndex: 'DoCount',
    // key: 'DoCount',
    ellipsis: true,
  },
]

export default {
  data() {
    return {
      columns,
      loading: false,
      knowledgeColumns,
      levelColumns,
      questionColumns,
      //是否展示弹窗
      isPopUp: false,
      //弹窗列表页
      isPopList: false,
      isPopTime: false,
      UserId: '',
      //班级列表
      gradeList: [],
      gradeId: '',
      // isLoading: false,
      practiceList: [],
      PracticeId: '',
      SummaryData: {},
      WholeDetailList: [],
      // 折线图数据
      lineChartList: [],
      //柱状折线图
      barLineList: [],
      // 知识点
      knowledgePointList: [],
      // 题型
      questionList: [],
      // 学习水平
      learnLevel: [],
      // 高频错题
      weekHighList: [],
      //子组件key
      lineKey: 0,
      // barKey:1,
      //环形图1
      KnowledgesList: [],
      //倒柱状图表
      DoItemTimeIntervalsList: [],
      //环形图弹窗入参
      // AnnularParams: {},
      // 时间弹窗数据?
      // PopDataList: [],
      PopDataTitle: '',
      //总体情况分析
      columnsWhole,
      req: {},
      // 周报id
      WeeklyId: '',
      //弹窗数据
      popTableData: {},
      //三个弹窗的列名
      columns_popTable,
      //弹窗的缓冲
      isPopLoading: false,
      //折线图缓冲
      isLineChartLoading: false,
      // 总结缓冲
      isSummaryLoading: false,
      //两个图表
      isTwoChart: false,
      //整体
      isWholeLoading: false,
      //
      isBarLineLoading: false,
      //知识点
      isKnowledgeLoading: false,
      //题型
      isQuestionLoading: false,
      //学习水平
      isLevelLoading: false,
      // 高频错题
      isWeekLoading: false,
      //班级列表加载中
      isClassLoading: false,
      //练习列表加载中
      isPracticeLoading: false,
      // 图表key
      TwoChartKey: 0,
      barLineKey: 0,
      IsVip: 0,
      //点击环形图弹窗加载中
      isPopAnnularLoading: false,
      //环形弹窗列表
      PopAnnularList: [],
      // 环形弹窗名称
      popAnnularTitle: '',
      //时间柱状图弹窗的loading
      isTimeDetailLoading: false,
      //时间柱状图的数据
      TimeDetailList: [],
      //教师正确率对比
      isTeacherBarLineLoading: false,
      TeacherBarLineList: [],
      TeacherBarLineKey: 100,
      // 历史周报
      isHistoryLoading: false,
      historyList: [],
    }
  },
  created() {},
  methods: {
    async init() {
      this.IsVip = localStorage.IsVip
      if (this.IsVip == '0') {
        return
      }

      this.UserId = localStorage.UserId
      this.WeeklyId = this.$route.query.weeklyId
      await this.getHistoryData()
      await this.getTeachClass()
      await this.getPracticeType()
      this.req = {
        Id: this.WeeklyId,
        UserId: this.UserId,
        Grade: this.gradeId,
        PracticeId: this.PracticeId,
      }
      await this.getInit()
    },
    toHistoryWeekly() {
      this.$router.push({
        path: '/SchoolManager/LearningEvaluation/HistoryWeekly',
      })
    },
    // 获取班级信息
    async getTeachClass() {
      this.isClassLoading = true
      try {
        // /Class/TeacherClassManager/GetGradeList
        const res = await this.$uwonhttp.get('/Class/TeacherClassManager/GetGradeList')
        this.gradeList = res.data.Data
        this.gradeId = this.gradeList[0].Id
      } catch (error) {
        throw error
      }

      this.isClassLoading = false
    },
    // 获取是区练习 还是校练习
    async getPracticeType() {
      const req = {
        Id: this.WeeklyId,
        UserId: this.UserId,
        Grade: this.gradeId,
      }
      this.isPracticeLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageWeeklyPracticeType', req)

        this.practiceList = res.data.Data
        this.PracticeId = this.practiceList[0].PracticeId
      } catch (error) {
        throw error
      }
      this.isPracticeLoading = false
    },
    // /Weekly/TeacherWeekly/GetTeacherWeeklySummary
    async getInit() {
      await this.getLineChart()
      await this.getSummary()
      await this.gettwoChart()
      await this.getWholeDetail()
      await this.getBarLineChart()
      await this.getTeacherBarLine()
      await this.getKnowledgePoint()
      await this.getLearnLevel()
      //题型
      await this.getQuestion()
      await this.getWeekHigh()
    },
    //折线图
    async getLineChart() {
      this.isLineChartLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageWeeklyLineChart', this.req)

        this.lineChartList = res.data.Data
        this.lineKey++
      } catch (error) {
        throw error
      }
      this.isLineChartLoading = false
    },
    async getSummary() {
      //总结
      this.isSummaryLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageWeeklySummary', this.req)
        this.SummaryData = res.data.Data
      } catch (error) {
        throw error
      }
      this.isSummaryLoading = false
    },
    async gettwoChart() {
      // 做题情况分析
      this.isTwoChart = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageDoItemTimeInterval', this.req)
        //做题情况数据
        this.KnowledgesList = res.data.Data.Knowledges
        this.DoItemTimeIntervalsList = res.data.Data.DoItemTimeIntervals
        this.TwoChartKey++
      } catch (error) {
        throw error
      }
      this.isTwoChart = false
    },
    async getWholeDetail() {
      // 整体详情
      this.isWholeLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManagePaperAccuracy', this.req)
        this.WholeDetailList = res.data.Data
      } catch (error) {
        throw error
      }
      this.isWholeLoading = false
    },
    async getBarLineChart() {
      // 折线柱状图 本周班级正确率对比分析
      this.isBarLineLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageClassAccuracy', this.req)
        this.barLineList = res.data.Data
        this.barLineKey++
      } catch (error) {
        throw error
      }
      this.isBarLineLoading = false
    },

    async getKnowledgePoint() {
      // 底部第一个
      this.isKnowledgeLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageWeekChapterAnalysis', this.req)
        this.knowledgePointList = res.data.Data
      } catch (error) {
        throw error
      }
      this.isKnowledgeLoading = false
    },
    async getQuestion() {
      //题型 底部第三个
      this.isQuestionLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageWeekPlateAnalysis', this.req)
        this.questionList = res.data.Data
      } catch (error) {
        throw error
      }
      this.isQuestionLoading = false
    },
    async getLearnLevel() {
      // 学习水平 底部第二个
      this.isLevelLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageWeekExamineAnalysis', this.req)
        this.learnLevel = res.data.Data
      } catch (error) {
        throw error
      }
      this.isLevelLoading = false
    },
    async getWeekHigh() {
      //高频错题
      this.isWeekLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageWeekHighItems', this.req)
        this.weekHighList = res.data.Data
      } catch (error) {
        throw error
      }
      this.isWeekLoading = false
    },
    //环形弹窗服务===
    async popAnnularInfo(ChapterId) {
      const req = {
        Id: this.WeeklyId,
        UserId: this.UserId,
        Grade: this.gradeId,
        PracticeId: this.PracticeId,
        ChapterId: ChapterId,
      }
      this.isPopAnnularLoading = true
      try {
        // /Weekly/TeacherWeekly/GetTeacherWeekChapterAnalysisListWeb
        // /HomePageDataView/TeacherManage/GetManageWeekChapterListWeb
        const res = await this.$http.post('/HomePageDataView/TeacherManage/GetManageWeekChapterListWeb', req)
        // debugger
        this.PopAnnularList = res.Data
      } catch (error) {
        throw error
      }
      this.isPopAnnularLoading = false
    },

    // 做题情况分析
    async ev_getAnnularItem(params) {
      //  入参
      // this.AnnularParams = params
      // console.log(e,"aaaaaaa");

      this.popAnnularInfo(params.ChapterId)
      this.popAnnularTitle = params.name
      //获取弹窗数据
      this.isPopList = true
    },
    ev_getBarItem(params, title) {
      // this.PopDataList = list
      this.PopDataTitle = title
      /*  if (params.BeginTime) {
        this.isPopTime = true
      } */
      // console.log(this.PopDataList, this.PopDataTitle)
      this.getTimeDetail(params)
      this.isPopTime = true
    },

    async getTimeDetail(params) {
      const req = {
        Id: this.WeeklyId,
        UserId: this.UserId,
        Grade: this.gradeId,
        PracticeId: this.PracticeId,
        // ChapterId: ChapterId,
        BeginTime: params.BeginTime,
        EndTime: params.EndTime,
      }
      this.isTimeDetailLoading = true
      try {
        // /Weekly/TeacherWeekly/GetTeacherWeekChapterAnalysisListWeb
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageDoItemTimeIntervalInfo', req)
        // debugger
        this.TimeDetailList = res.data.Data
      } catch (error) {
        throw error
      }
      this.isTimeDetailLoading = false
    },

    //本周正确率对比分析
    // /Weekly/ManageWeekly/GetManageTeachersAccuracy
    async getTeacherBarLine() {
      const req = {
        Id: this.WeeklyId,
        UserId: this.UserId,
        Grade: this.gradeId,
        PracticeId: this.PracticeId,
      }
      this.isTeacherBarLineLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageTeachersAccuracy', req)
        // if (res.Success) {
        this.TeacherBarLineList = res.data.Data
        console.log(this.TeacherBarLineList)
        // debugger
        this.TeacherBarLineKey++
        // }
      } catch (error) {
        throw error
      }
      this.isTeacherBarLineLoading = false
    },

    changeGrade(e) {
      this.gradeId = e
      this.req = {
        Id: this.WeeklyId,
        UserId: this.UserId,
        Grade: this.gradeId,
        PracticeId: this.PracticeId,
      }
      this.getInit()
    },
    changePractice(e) {
      this.PracticeId = e
      this.req = {
        Id: this.WeeklyId,
        UserId: this.UserId,
        Grade: this.gradeId,
        PracticeId: this.PracticeId,
      }
      this.getInit()
    },
    async topop(TypeId, type) {
      //获取弹窗入参
      console.log(arguments)
      // res
      this.isPopUp = true
      const req = {
        Type: type,
        TypeId: TypeId,
        WeeklyId: this.WeeklyId,
        PracticeId: this.PracticeId,
        Grade: this.gradeId,
        UserId: this.UserId,
      }
      this.isPopLoading = true
      const res = await this.$uwonhttp.post('/Weekly/ManageWeekly/GetManageSubjectAbilityList', req)

      if (res.data.Success) {
        this.popTableData = res.data.Data
      }
      this.isPopLoading = false
    },
    toCheck(id, chapterNum) {
      this.$router.push({
        path: '/Teacher/My_Task/InspectionWorkData',
        query: {
          paperId: id,
          Grade: this.gradeId,
          chapterNum: chapterNum,
          showgradeId: '1',
          UserId: this.UserId,
        },
      })
    },
    ev_getSmallAnnular(obj) {
      console.log(obj)
      if (typeof obj.chapterId == 'undefined') {
        return
      }
      this.$router.push({
        path: '/Teacher/My_Task/InspectionWorkData',
        query: {
          paperId: obj.PaperId,
          gradeId: this.gradeId,
          chapterNum: obj.chapterId,
          showgradeId: '1',
          color: 2,
        },
      })
    },
    async getHistoryData() {
      var req = {
        pageNo: 1,
        pageIndex: 35,
      }

      this.isHistoryLoading = true
      try {
        const res = await this.$uwonhttp.post('/Weekly/StudentWeekly/GetHistoryStudentWeekly', req)
        this.historyList = res.data.Data.Items
        this.historyList = this.historyList.slice(0, 7)
        this.WeeklyId = this.historyList[0].Id
        // this.TotalItems = res.data.Data.TotalItems
      } catch (error) {
        console.log(error)
        throw error
      }

      this.isHistoryLoading = false
    },
    async changeWeekly(Id) {
      this.WeeklyId = Id
      await this.getTeachClass()
      await this.getPracticeType()
      this.req = {
        Id: this.WeeklyId,
        UserId: this.UserId,
        ClassId: this.ClassId,
        PracticeId: this.PracticeId,
      }
      await this.getInit()
    },
  },
  activated() {
    this.init()
  },
  components: { Annular, AnnularSmall, Bar, Lines, BarLine, PopUp, PopList, PopList },
  watch: {},
}
</script>

<style lang="less" scoped>
@black: #333;
.noLack {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 500px;
  flex-direction: column;
  color: #909090;
}
.wrapper {
  // *box-sizing: padding-box;
  margin: 16px auto 24px;
  width: 100%;
  .example {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
  }
  .red {
    color: #f87175;
  }
  .head {
    padding: 16px;

    width: 100%;
    background-color: #fff;
    // height: 134px;
    display: flex;
    align-items: center;
    justify-content: space-between;

    flex-direction: row;

    color: @black;

    border-radius: 8px;
    .select-info {
      display: flex;
      align-items: center;
      justify-items: flex-start;

      h3 {
        font-size: 18px;
      }
      .desc {
        font-size: 14px;
        margin-left: 24px;

        color: @black;
      }
      .select-box {
        width: 140px;
        // border: none;

        border-radius: 2px;
        font-size: 14px;
        margin-left: 14px;
        color: @black;
        text-align: center;
        /deep/.ant-select-selection {
          // border: none;
          border: 1px solid #ccd0db;
          .ant-select-arrow {
            color: @black;
          }
          .ant-select-selection-selected-value {
            float: none;
            text-align: center;
          }
        }
        &.week-time {
          width: 240px;
        }
      }
    }
    .history-info {
      cursor: pointer;
      font-size: 16px;
      padding-right: 9px;
      img {
        width: 30px;
        margin-right: 9px;
      }
    }
  }
  .container {
    // display: flex;
    margin-top: 14px;
    // justify-content: space-between;
    // align-items: flex-start;
    // height: 666px;
    // flex: 1;
    &.left-mod {
      height: 100%;
      width: 100%;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: flex-start;
      //  flex: 1 1 53%;
      margin-right: 16px;
      .numb-info {
        width: 100%;
        display: flex;
        justify-content: flex-start;
        align-items: center;
        padding: 18px 16px;
        background: #ffffff;
        border-radius: 8px;
        color: @black;
        font-size: 14px;
        line-height: 28px;
        span {
          margin-right: 6px;
          &.big {
            font-size: 14px;
            color: @black;
            margin-right: 14px;
          }
        }
        .ant-divider,
        .ant-divider-vertical {
          width: 1px;
          height: 20px;
          background: #777;
        }
      }
      .summary-info {
        width: 100%;
        margin-top: 14px;

        background: #ffffff;
        border-radius: 8px;
        padding: 16px;

        .desc {
          line-height: 1.8;
          padding: 8px 0 14px;
          font-size: 14px;
          color: @black;

          .yellow {
            color: #ffb718;
          }
          .green {
            color: #71b6a1;
          }
        }
      }
      .situation-info {
        background: #ffffff;
        border-radius: 8px;
        margin-top: 14px;
        width: 100%;

        padding: 16px;

        .content-box {
          // display: flex;
          // flex-direction: row;
          // justify-content: space-between;
          // align-items: center;

          .circle-info {
            // cursor: pointer;
            //   flex:1 1 48%;
            // border-right: 1px #efefef solid;
            // display: flex;
            // flex-direction: column;
            // justify-content: center;
            // align-items: center;
            h4 {
              font-size: 15px;
              line-height: 2px;
              margin: 16px 0;
              align-self: flex-start;
              font-weight: bold;
              color: @black;
            }
            /*             &:first-child {
              flex-basis: 46%;
              flex-grow: 0;
              flex-shrink: 0;
              align-items: flex-start;
            }
            &:last-child {
              border-right: none;
              flex: 1 1 auto;
              h4 {
                align-self: flex-start;
                margin-left: 24px;
              }
            } */
            .chart {
              //   margin: 14px 0 0;

              width: 100%;

              height: 400px;
              background: #f1f5f6;
              border-radius: 2px;
              /*  &.histogram {
                width: 92%;
                height: 356px;
              } */
            }
          }
        }
      }
    }
    &.anlysis-mod {
      height: 100%;
      width: 100%;

      /*       flex: 1;
      min-height: 0; */

      padding: 16px;
      background: #fff;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: flex-start;
      border-radius: 8px;

      font-size: 14px;

      color: @black;

      .content-box,
      .content-box2 {
        // width: 100%;
        // display: flex;
        // align-items: center;
        // justify-content: space-between;
        margin-top: 14px;
        .item {
          .title-info {
            display: flex;
            justify-content: space-start;
            align-items: center;
            height: 18px;
            margin: 10px 0;
            h4 {
              font-size: 15px;
              line-height: 2px;
              color: @black;
              font-weight: bold;
            }
          }
          .table-info-whole {
            width: 100%;
            height: 300px;
            overflow-x: auto;
            overflow-y: auto;

            padding: 10px;
            margin-top: 10px;
            background: #f1f5f6;
            border-radius: 2px;
            text-align: center;

            /deep/ .ant-table-content {
              width: 100%;

              min-width: 100%;
              // min-height: 185px;
              overflow-x: auto;
              overflow-y: auto;
              .ant-table-thead {
                > tr {
                  > th {
                    line-height: 2;
                    color: @black;
                    font-size: 14px;
                    font-weight: bold;
                    border-bottom: none;
                    text-align: center;
                    background: none;
                    padding: 5px;
                    height: 48px;
                    white-space: nowrap;
                    overflow: hidden;
                    text-overflow: ellipsis;
                    &:first-child {
                      width: 95px;
                    }
                  }
                }
              }
              .ant-table-tbody {
                > tr {
                  > td {
                    text-align: center;
                    font-size: 14px;
                    padding: 5px;
                    background: none;
                    height: 40px;
                    white-space: nowrap;
                    overflow: hidden;
                    text-overflow: ellipsis;
                  }

                  &:last-child {
                    > td {
                      border-bottom: none;
                    }
                  }
                }
              }
            }
          }
          .chart {
            width: 100%;
            height: 300px;
            background: #f1f5f6;
            border-radius: 2px;
            margin-top: 10px;
            padding: 10px 0;
          }
        }
        /*        .chart-mod {
          flex: 1;

          .title-info {
            display: flex;
            justify-content: flex-start;
            align-items: center;
            height: 18px;
            margin: 10px 0;
            h4 {
              font-size: 14px;
              line-height: 2px;
               color:@black;
            }
          }

          .chart {
            width: 100%;
            height: 223px;
            background: #f1f5f6;
            border-radius: 2px;
            margin-top: 10px;
            padding: 10px 0;
          }
        } */
        /*         &.second {
          flex: 1;
          .item,
          .chart-mod {
            //  flex: 1;
            .chart {
              height: 300px;
              // height: 250px;
            }
          }
          /* .chart-mod {
            .chart {
              height: 281px;
            }
          } 
        } */
      }
      /*       h5 {
        font-size: 15px;
        line-height: 2px;
        padding: 24px 0;
        font-weight: bold;
        color: @black;
      }
      .anlysis-chart {
        width: 100%;
        height: 400px;
        background: #f1f5f6;
        border-radius: 2px;
      } */
    }
  }

  .bottom-info {
    margin-top: 14px;
    padding: 16px 16px 24px;

    width: 100%;
    background-color: #fff;
    flex-direction: column;
    color: @black;
    border-radius: 8px;
    overflow: hidden;
    .bottomList {
      display: flex;
      align-items: center;
      justify-content: space-between;
      .item {
        // flex: 0 0 24%;
        margin-bottom: 14px;
        .title-info {
          display: flex;
          justify-content: flex-start;
          align-items: center;
          height: 18px;
          margin: 10px 0;
          // font-size: 8px;
          h4 {
            font-size: 15px;
            line-height: 2px;
            color: @black;
            font-weight: bold;
            .red {
              cursor: pointer;
            }
          }
          /*           .desc {
            transform: scale(0.8, 0.9);
            transform-origin: right;
          } */
        }
        .table-info {
          width: 100%;
          height: 200px;
          overflow: auto;
          padding: 10px;
          margin-top: 10px;
          background: #f1f5f6;
          border-radius: 2px;
          text-align: center;
          /deep/ .ant-table-content {
            width: 100%;
            // height: 154px;
            min-width: 100%;
            min-height: 154px;
            overflow-x: auto;
            overflow-y: auto;

            .ant-table-thead {
              > tr {
                > th {
                  line-height: 2;
                  color: @black;
                  font-size: 14px;
                  font-weight: bold;
                  border-bottom: none;
                  text-align: center;
                  background: none;
                  padding: 5px;
                  height: 40px;
                }
              }
            }
            .ant-table-tbody {
              > tr {
                > td {
                  text-align: center;
                  font-size: 14px;
                  padding: 5px;
                  background: none;
                  height: 32px;
                  white-space: nowrap;
                  overflow: hidden;
                  text-overflow: ellipsis;
                  a {
                    color: #0ab7ef;
                    margin-left: 2px;
                  }
                  i {
                    vertical-align: middle;
                  }
                }

                &:last-child {
                  > td {
                    border-bottom: none;
                  }
                }
              }
            }
          }
        }
        .chart {
          width: 100%;

          height: 200px;
          background: #f1f5f6;
          border-radius: 2px;
          margin-top: 10px;
          padding: 10px;
          /*      display: flex;
          align-items: center;
          flex-wrap: wrap; */

          justify-content: space-around;

          overflow: auto;
          position: relative;
          .box-container {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            display: flex;
            // display: flex;
            align-items: center;
            flex-wrap: wrap;
            justify-content: space-around;
            flex-direction: row;

            .box {
              width: 33%;
              height: 100%;
              display: flex;
              align-items: center;
              flex-direction: column;
              justify-content: center;
              // flex: 0 0 96px;
              .chart-area {
                width: 100%;
                height: 96px;
              }
              p {
                margin-top: 1em;
              }
            }
          }
        }
      }
    }
  }

  h3 {
    font-size: 18px;
    line-height: 2;

    color: #0ab7ef;
    font-weight: bold;
    font-family: PingFangSC-Medium, PingFang SC;
  }
}

::-webkit-scrollbar {
  /*滚动条整体样式*/
  width: 8px; /*高宽分别对应横竖滚动条的尺寸*/
  height: 1px;
}

::-webkit-scrollbar-thumb {
  /*滚动条里面小方块*/
  border-radius: 8px;
  -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  background: #ccd0db;
}

::-webkit-scrollbar-track {
  /*滚动条里面轨道*/
  -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  background: #ededed;
}
</style>
