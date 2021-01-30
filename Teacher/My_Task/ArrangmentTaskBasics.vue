<template>
  <div>
    <div style="display: flex" @click="cancleTitle($event)">
      <div class="subject">
        <h3>
          <p v-show="bigTitleDefault" style="font-size: 16px;">{{ paperName }} <img id="myTitleBtn" @click="editBigTile" class="cur" src="@/assets/teacher/布置作业编辑.png" alt=""></p>
          <input id="myTitle" v-if="BigTitle" v-model="paperName" type="text" />
        </h3>
        <p class="tips">请选择“添加题型”自主制题，也可选择“作业协同/云题库/本地练习导入/我的题库”直接引用练习或题目</p>
        <div>
          <div
            v-for="(item,index) in arr"
            :key="item.Id"
            v-show="wholeEdit"
            class="question-title-type"
          >
            <!-- @mouseleave="handleTitleHide()" -->
            <p
              class="title-edit"
              :class="{ 'qus-type': titleType === item.Id }"
              @mouseenter="handleTitleType(item.Id,item.Name)"

              :selectValueBigQuestion="item.Id">
              {{ index + 1 }}. {{ item.Name.split('题')[0] }}题 (共{{ item.Items.length }}题，共{{ item.Score }}分)
              <span v-show="titleType === item.Id" >
                <span class="z-span" @click="deleteBigQuestion(item.Id)">删除</span>
                <!-- 弹窗的遮罩颜色 -->
                <!-- :maskStyle="{ backgroundColor: '#ccc'}" -->
                <a-modal
                  v-model="deleteBigType"
                  title="温馨提示"
                  width="630px"
                  @ok="handleDeleteBigType(item.Id)"
                  :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.2)'}"
                  centered
                >
                  <div>
                    确定删除吗？
                  </div>
                </a-modal>
                <span class="z-span" @click="editSubject(item.Id,item.TypeId,item.Type)">制作题目</span>
                <!-- :maskStyle="{ backgroundColor: 'rgba(255, 255, 255, 0.5)'}" -->

                <a-drawer
                  title=""
                  placement="right"
                  :visible="MakeTopic"
                  @close="onCloseMakeTopic"
                  width="60%"
                  height="100%"
                  :drawerStyle="{ backgroundColor: '#FAF7F8'}"
                  :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.1)'}"
                  destroyOnClose
                >

                  <div class="make-topic clearfix">
                    <div class="make-title">
                      <span class="f-t bold" v-if="questionId === 2">单选题</span>
                      <span class="f-t bold" v-if="questionId === 10">多选题</span>
                      <span class="f-t bold" v-if="questionId === 11">判断题</span>
                      <span class="f-t bold" v-if="questionId === 5">填空题</span>
                      <span class="f-t bold" v-if="questionId === 23">应用题</span>
                      <p v-if="selectCheckLevel" class="clearfix"><span class="fg" style="vertical-align: middle;">请设置题目难度</span><img class="fg" src="@/assets/teacher/警告.png" alt=""></p>
                      <span class="fg f-t">题目难度设置: <a-rate v-model="aloneLevel" @change="handleAloneLevel"></a-rate></span>
                      <!-- 编写的提示 -->
                      <p class="f-c">
                        <span class="make-sub-title f-t">题干</span>
                        <span style="color: #FF8D60;">
                          <span v-if="titleCheck">
                            <img src="@/assets/teacher/提示编组.png" alt="">&nbsp;
                            <span style="vertical-align: middle;">题干不能为空</span>
                          </span>
                        </span>
                      </p>
                      <!-- <TinyMceEditor @keyup.native="keyUp" :height="220" ref="edit" v-model="subTitle"></TinyMceEditor> -->
                      <wang-edit @keyup.native="keyUp" style="height: 90%" ref="edit" v-model="subTitle"></wang-edit>
                    <!-- <wang-edit v-if="aa === '2'" style="height: 90%" ref="edit" v-model="subjectListTitle"></wang-edit> -->
                    </div>
                    <!-- 选择题 -->
                    <div v-if="questionId === 2 || questionId === 10 || questionId === 11" class="make-select">
                      <p class="f-c f-t">答案选项</p>

                      <!-- <a-radio-group @change="onChangeSlect" v-model="selectValue">
                      <a-radio :value="2">单选</a-radio>
                      <a-radio :value="10">多选</a-radio>
                      <a-radio :value="11">判断题</a-radio>
                    </a-radio-group> -->
                      <div style="margin-top:15px">
                        <p v-for="subOption in subOptions" :key="subOption.id" v-show="questionId === 2">
                          <span class="options" >

                            <a-radio-group @change="onChangeSlect" v-model="alonevalue">
                              <a-radio :value="subOption.Option">{{ subOption.Option }} 选项</a-radio>
                            </a-radio-group>
                            <span v-if="subOption.Content === '' && selectCheck" ><img src="@/assets/teacher/警告.png" alt=""><span style="vertical-align: middle;">选项或答案不能为空</span></span>
                          <!-- {{ subOption.Option }}选项 -->
                          </span>
                          <!-- <span v-if="subOption.Content !== ''" style="color:#FF682C">选项不能为空</span> -->
                          <!-- <TinyMceEditor :height="100" v-model="subOption.Content"></TinyMceEditor> -->
                          <wang-edit v-model="subOption.Content"></wang-edit>
                        </p>
                        <p v-show="questionId === 10">
                          <span class="options" >
                            <a-checkbox-group @change="onChangeduo" :defaultValue="MultipleChoice">
                              <a-row>
                                <a-col v-for="subOption in subOptions" :key="subOption.id" :span="24">
                                  <a-checkbox :value="subOption.Option">
                                    {{ subOption.Option }}选项
                                  </a-checkbox>
                                  <span v-if="subOption.Content === '' && selectCheck"><img src="@/assets/teacher/提示编组.png" alt=""><span style="vertical-align: middle;">选项或答案不能为空</span></span>
                                  <!-- <TinyMceEditor :height="100" v-model="subOption.Content"></TinyMceEditor> -->
                                  <wang-edit v-model="subOption.Content"></wang-edit>
                                </a-col>
                              </a-row>
                            </a-checkbox-group>
                          <!-- {{ subOption.Option }}选项 -->
                          </span>

                        </p>
                        <p v-for="subOption in subJudgeOptions" :key="subOption.id" v-show="questionId === 11">
                          <span class="options" >
                            <a-radio-group @change="onChangeSlect" v-model="judge">
                              <a-radio :value="subOption.Option">{{ subOption.Option }} {{ subOption.Content }}</a-radio>
                            </a-radio-group>
                          <!-- {{ subOption.Option }}选项 -->
                          </span>
                        <!-- <wang-edit v-model="subOption.Content"></wang-edit> -->
                        </p>
                      </div>
                      <!-- <p>答案
                      <wang-edit v-model="answer"></wang-edit>
                    </p> -->
                      <p v-if="questionId === 10 || questionId === 2" style="float: right;font-size: 18px;margin-bottom: 0;">
                        <span class="f-c">选项</span>
                        <a-icon type="plus-circle" @click="AddOptions"/>&nbsp;&nbsp;&nbsp;&nbsp;
                        <a-icon type="minus-circle" @click="ReduceOptions"/>
                      </p>
                    </div>
                    <div v-if="questionId === 23">
                      <p>答案</p>
                      <input id="subjective" type="text" v-model="subjective">
                    </div>
                    <!-- 填空题 -->
                    <div v-if="questionId === 5" class="make-select">
                      
                      <!-- <p class="f-c f-t">答案选项</p> -->
                      <div v-for="( i, j) in 1" :key="j" class="">
                        <!-- 空{{ j + 1 }}
                      <wang-edit v-model="completion[j]"></wang-edit> -->
                        <!-- {{ index }} -->

                        <span class="blanks-infor" v-for="(item,index) in blanksUp" :key="item"> 空{{ index + 1 }} <p>{{ item }}</p></span>
                        <span class="fg agreen-answer" @click="AddSynonym(j)" style="marginTop: 15px">添加同义答案</span>

                        <!-- {{ synonym }} -->
                        <!-- 同义答案 -->
                        <div v-for="(s, t) in counter" :key="t" v-show="synonym && num === j">
                          <SynonymAnswer :sd="num" @get="get"></SynonymAnswer>
                        <!-- v-for="(Syn, k) in EchoSynonymousAnswer" :key="k" -->
                        </div>
                        <span class="sy-blocks" v-show="synonym && num === j">
                          <a-icon type="plus-circle" @click="syAdd"/>&nbsp;&nbsp;&nbsp;&nbsp;
                          <a-icon type="minus-circle" @click="syDel"/>
                        </span>
                      </div>
                      <div class="echo-synonymous-answer">
                        <p>已经添加的同义答案</p>
                        <p class="echo-p" v-for="(Syn, k) in EchoSynonymousAnswer" :key="k">
                          <span>{{ Syn }}</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                          <span @click="handleEchoSynonymousAnswer(Syn)" class="cur">删除</span>
                        </p>
                      </div>
                    <!-- <span class="fg fill-blocks">
                      选项
                      <a-icon type="plus-circle" @click="AddOptions"/>&nbsp;&nbsp;&nbsp;&nbsp;
                      <a-icon type="minus-circle" @click="ReduceOptions"/>
                    </span> -->
                    </div>
                   
                    <div class="make-analysis fl">
                      <p class="f-c">
                        <span class="m-r f-t">解析</span>
                        <span v-if="analysisCheck">
                          <img src="@/assets/teacher/提示编组.png" alt="">&nbsp;
                          <span style="vertical-align: middle;">解析不能为空</span>
                        </span>
                      </p>
                      <!-- <TinyMceEditor :height="200" v-model="subAnalysis"></TinyMceEditor> -->
                      <wang-edit v-model="subAnalysis"></wang-edit>
                    </div>
                  </div>
                  <div class="edit-subject-operation">
                    <span @click="cancelEdit">取消</span>
                    <a-modal
                      v-model="EditCance"
                      title="温馨提示"
                      width="630px"
                      @ok="handleOkEdit"
                      @cancel="handleCancelEdit"
                      :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.2)'}"
                      centered
                    >
                      <p>关闭后，正在编辑的题目将不做任何变更！是否保存</p>
                    </a-modal>
                    <span @click="handleSave">保存</span>
                  </div>
                </a-drawer>
                <!-- ----------------------------------------------------- -->
                <span class="z-span" @click="setScore(item.Id)">批量设定得分</span>
                <a-modal
                  v-model="score"
                  title="设定分数:"
                  width="630px"
                  @ok="handleBigTitleScore()"
                  :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.2)'}"
                  centered
                >
                  <div style="textAlign: center">
                    <p>
                      <span>{{ scoreName }}: &nbsp;&nbsp;&nbsp;</span> <input v-model="multipleScore" class="input-mark" type="text">分 X <span>{{ item.Items.length }}</span>题

                    </p>
                    <p>共计: <span>{{ item.Score }}分</span></p>
                  </div>
                </a-modal>
              </span>
            </p>
            <!-- 题目 -->
            <div
              v-for="(ite, inde) in item.Items"
              :key="inde"
              @mouseenter="IndividualSubject(ite.ItemId)"
              :class="{ 'qus-type': IndividualSettings === ite.ItemId }"
              class="clearfix "
              style="padding: 10px 30px;border-radius:5px;cursor:pointer;margin-bottom: 20px;">
              <p class="fg">{{ ite.Score }}分</p>
              <p class="fl">{{ inde + 1 }}.</p>
              <div class="img-div" v-html="ite.Title" style="padding: 0 20px;cursor:pointer;">
              </div>
              <!-- 带有选项类的题 -->
              <div class="paper-title" v-show="ite.ItemTypeId !== 5">
                <P v-for="(t,n) in ite.Options" :key="n" v-show="t.Type === 1">
                  <span>{{ t.Option }}</span> : <span style="display: inline-block;" v-html="t.Content"></span>
                </P>
              <!-- <P>B : 2</P>
              <P>C : 3</P> -->
              </div>
              <div v-show="IndividualSettings === ite.ItemId">
                <p class="clearfix">
                  <span class="fg z-span" @click="deleteAlone(ite.Id)">删除</span>
                  <a-modal
                    v-model="deleteAloneList"
                    title="温馨提示"
                    width="630px"
                    @ok="handleDeleteAloneType(ite.Id)"
                    centered
                    :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.2)'}"
                  >
                    <div>
                      确定删除吗？
                    </div>
                  </a-modal>
                  <span class="fg z-span" @click="singleEditSubject(ite.ItemId,item.TypeId,item.Type,item.Id,ite.IsQuote)">编辑</span>

                  <span class="fg z-span" @click="SetUp(ite.Id)">设定得分</span>
                </p>
                <p v-for="(answerdata, i) in ite.Options" :key="i.Id" v-show="answerdata.Type === 8" style="font-size: 14px">答案: <span style="display: inline-block;margin-left: 20px" v-html="answerdata.Content"></span></p>
                <p v-for="(e,d) in ite.Options" :key="d" v-show="e.Type === 10" style="font-size: 14px">解析: <span style="margin-left: 20px" v-html="e.Content"></span></p>
              </div>
              <a-modal
                v-model="SetUpFraction"
                title="设定得分"
                @ok="handleFraction(ite.Id)"
                centered
                :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.2)'}"

              >
                <div style="textAlign: center">
                  <p>
                    <span>{{ item.Name }}: &nbsp;&nbsp;&nbsp;</span> <input class="input-mark" v-model="aloneFraction" type="text">&nbsp;&nbsp; 分
                  </p>
                </div>
              </a-modal>
            </div>
          </div>
        </div>
      </div>
      <div class="subject-info">
        <!-- 题目操作 -->
        <div class="subject-operation">
          <p class="title-type"><span @click="handleAddSubject">添加题型</span><span @click="toPracticePreview">练习预览</span></p>
           <a-modal
            v-model="fraction"
            :header="null"
            :closable="false"
            @ok="handleFractionList"
            okText="知道了"
            centered
          >
           总分值不正确，请重新赋分
          </a-modal>
          <a-modal
            v-model="visible"
            title="添加题型"
            @ok="handleOk"
            width="630px"
            centered
          >
            <img style="width: 36px;marginRight: 30px" src="@/assets/teacher/试卷.png" alt="">
            <a-select style="width: 349px" @change="handleChange" placeholder="选择题型">
              <a-select-option v-for="item in subjectList" :key="item.Id" :value="item.Id+ ',' +item.ItemTypeName+ ',' + item.ItemType">
                {{ item.ItemTypeName }}
              </a-select-option>
            </a-select>
          </a-modal>
          <p><span class="basic-btn" style="background-color: #5A7E81;color:#fff;" @click="handleComplete">完成发布</span></p>
          <a-modal
            v-model="time"
            :header="null"
            :closable="false"
            @ok="handleOkTime"
            centered
          >
            <p>指向发布:&nbsp;&nbsp;
              <a-checkbox-group @change="classChange1">
                <a-checkbox v-for="item in classInfor" :key="item.ClassId" :value="item.ClassId">{{ item.ClassName }}</a-checkbox>
              </a-checkbox-group>
            </p>
            <div class="m-b release-time">
              发布时间:&nbsp;&nbsp;
              <a-radio-group v-model="value">
                <a-radio :value="1" @click="handleImmediately">立即发送</a-radio>
                <br>
                <a-radio :value="2" @click="handleClick">
                  自定义时间:
                  <span v-if="value == 1">
                       
                    <!-- :disabledDate="disabledDate" -->
                    <a-date-picker
                      :showTime="{ format: 'HH:mm' }"
                      format="YYYY-MM-DD HH:mm"
                      :disabled-time="disabledDateTime"
                      :disabledDate="disabledDate"
                      :show-time="{ defaultValue: moment('00:00:00', 'HH:mm:ss') }"
                      disabled
                      :default-value="moment(PublishDateTime, 'YYYY-MM-DD HH:mm')"
                    >

                    </a-date-picker></span>
                  <span v-else>
                    <a-date-picker
                      :showTime="{ format: 'HH:mm' }"
                      format="YYYY-MM-DD HH:mm"
                      @change="StartTimeChange"
                      :disabledDate="disabledDate"
                      :default-value="moment(PublishDateTime, 'YYYY-MM-DD HH:mm')"
                    >
                    </a-date-picker></span>
                </a-radio>
              </a-radio-group>
            </div>
            <div class="m-b">
              作答截止时间:&nbsp;&nbsp;
              <a-date-picker
                :showTime="{ format: 'HH:mm' }"
                format="YYYY-MM-DD HH:mm"
                @change="TimeChange"
                :disabledDate="disabledDate1"
                :default-value="moment(AnswerEndDateTime, 'YYYY-MM-DD HH:mm')" >

              </a-date-picker></div>
            <a-radio-group v-model="value1" @change="onChange">
              <a-radio :style="radioStyle" :value="0">
                本练习保密(仅本人可看)
              </a-radio>
              <a-radio :style="radioStyle" value="共享">
                共享这份练习
              </a-radio>
            </a-radio-group>
            <div v-show="Area" style="padding: 0 27px">
              <!-- <a-checkbox-group @change="onChangeAreaSchool" :default-value="MultipleChoice">
              <a-row>
                <a-col>
                  <a-checkbox :value="2">
                    共享到全区
                  </a-checkbox>
                  <br>
                  <a-checkbox :value="1">
                    共享到全校
                  </a-checkbox>
                </a-col>
              </a-row>
            </a-checkbox-group> -->
              <a-radio-group v-model="valueArea" @change="handleArea">
                <a-radio :style="radioStyle" :value="2">
                  共享到全区
                </a-radio>
                <a-radio :style="radioStyle" :value="1">
                  共享到全校
                </a-radio>
              </a-radio-group>
            </div>
          </a-modal>
          <p class="title-source">题目来源</p>
          <p><span @click="showTaskVisible" class="basic-btn bgc">作业协同</span></p>
          <div>
            <a-drawer
              title=""
              placement="right"
              :closable="false"
              :visible="VisibleTask"
              @close="onClose"
              width="60%"
              :drawerStyle="{ backgroundColor: '#FAF7F8' }"
              :bodyStyle="{ padding: '80px'}"
            >
              <div class="task-help">
                <p @click="selectTeacherList" class="select-teacher cur">
                  全部试题
                </p>
                <div v-if="selectTeacher" class="select-te-list">
                  <p style="color:#4D5753; font-size:16px">星标好友</p>
                  <p class="star-infor" v-for="item in starFriendsList" :key="item.FriendsId">
                    <img :src="item.Photo" alt="">
                    <span>
                      <p>{{ item.RealName }}</p>
                      <p>{{ item.SchoolName }}</p>
                      <p>{{ item.Grade }}</p>
                    </span>
                    <span>共享练习{{ item.PracticeCount }}份</span>
                    <img class="cur" style="width: auto;" @click="SelectTheTeacherList(item.FriendsId,$event)" :src="checkIcon" alt="">
                  <!-- <span @click="SelectTheTeacher(item.FriendsId)">√</span> -->
                  </p>
                  <div v-if="starFriendsList.length > 0" style="border-top: 1px dotted #D6D6D6; padding-top: 10px;">
                    <p style="color:#4D5753; font-size:16px">好友列表</p>
                    <p> <img src="@/assets/teacher/school.png" alt=""> {{ teacherName }}</p>
                    <ul class="clearfix">
                      <li v-for="(item,index) in teacherSelectList" :key="index">
                        <p class="cur">{{ item.GradeName }}</p>
                        <!-- 0 互为好友  1 加好友 2 审核中 -->
                        <p v-for="(i,j) in item.FriendList" :key="j"><img src="@/assets/teacher/老师头像.png" alt="">&nbsp;&nbsp;&nbsp;&nbsp; <span>{{ i.RealName }}</span><span>共享练习{{ i.PracticeCount }}份</span>
                          <!-- <img @click="removeStudent(item.UserId,$event)" :src="checkIcon" alt=""> -->
                          <img class="cur" @click="SelectTheTeacherList(i.FriendsId,$event)" :src="checkIcon" alt="">
                        </p>
                      </li>
                    </ul>
                    <a-pagination
                      v-model="current"
                      :total="total"
                      show-quick-jumper
                      :page-size="1"
                      hideOnSinglePage
                      style="text-align: center;"
                      @change="onChangeXin"/>
                    <span class="seacher-confirm" style=" background-color: #B5B5B5;" @click="handleSelectTeacher">取消</span> <span class="seacher-confirm btn-col" @click="handleSelectTeacherDetermineList">确定</span>
                  </div>
                  <div v-if="starFriendsList.length === 0 " class="star-beacon">
                    <img src="@/assets/lack/暂无关注好友.png" alt="">
                    <p>暂无好友信息</p>
                  </div>
                </div>
                <div v-if="starFriendsList.length === 0 && taskTeacherList.length === 0" class="star-beacon">
                  <img src="@/assets/lack/暂无搜索记录.png" alt="">
                  <p>暂无好友，请先添加好友</p>
                  <p style="text-align:center;"><span @click="toAddFriends" class="cur add-friends">去添加好友</span></p>
                </div>
                <div v-for="item in taskTeacherList" :key="item.Id" >
                  <!-- 教师 -->
                  <p class="clearfix teacher-p">
                    <span class="fl">
                      <a-avatar style="width:64px;height:64px;" :size="64" icon="user" :src="item.Photo"/>
                    </span>
                    <span class="fl">
                      <p>{{ item.RealName }}</p>
                      <p>{{ item.SchoolName }} | {{ item.GradeName === null ? '' : item.GradeName }}</p>
                    </span>
                    <span>共享练习{{ item.SharePaperCount }}份</span>
                  </p>
                  <!-- 教师对应试卷 -->
                  <ul class="teacher-paper clearfix">
                    <li v-for="i in taskTeacherListPaper[item.Id]" :key="i.PaperId">
                      <p class="clearfix"><img class="fl" src="@/assets/teacher/矩形.png" alt=""><span @click="handlePreview(i,item)" class="fg cur select-color">预览</span></p>
                      <a-drawer
                        title=""
                        placement="right"
                        :visible="Preview"
                        @close="onPreviewClose"
                        width="60%"
                        :drawerStyle="{ backgroundColor: '#FAF7F8' }"
                        :bodyStyle="{ padding: '80px'}"
                      >
                        <!-- 试卷预览 -->
                        <div class="paper-preview">
                          <!-- <div style="margin-bottom: 25px">搜索</div> -->
                          <!-- 教师数据 -->
                          <p class="clearfix teacher-p">
                            <span class="fl">
                              <a-avatar :size="64" icon="user" :src="PreviewTeacher.Photo"/>
                            </span>
                            <span class="fl">
                              <p>{{ PreviewTeacher.RealName }}</p>
                              <p>{{ PreviewTeacher.SchoolName }} | {{ PreviewTeacher.GradeName === null ? '' : PreviewTeacher.GradeName }}</p>
                            </span>
                            <span>共享练习{{ PreviewTeacher.SharePaperCount }}份</span>
                          </p>
                          <p class="quote"><span @click="backPreviewPaper" style="flex-grow: 1" class="select-color cur">返回练习列表</span><span style="flex-grow: 2">{{ PreviewPaper.Title }} &nbsp;&nbsp;&nbsp; <a-rate style="color: yellow" :default-value="PreviewPaper.DifficultyStar" :count="PreviewPaper.DifficultyStar" disabled />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;已引用{{ PreviewPaper.QuoteCount }}次</span><span @click="OnekeyQuote" class="select-color cur">一键全部引用</span></p>
                          <div>
                            <!-- 试卷 -->
                            <div style="margin-bottom: 70px;border-bottom: 1px dotted #ccc;padding: 50px 20px;position: relative;" v-for="item1 in paperData" :key="item1.ItemId">
                              <span v-html="item1.Title" style="fontSize: 20px;"></span>
                              <!-- 试题选项 -->
                              <div>
                                <a-radio-group>
                                  <a-radio :style="radioStyle" :value="j" v-for="(k, j) in item1.AuditPaperItemAnswers" :key="k.Option" v-show="k.Type === 1">
                                    <span
                                      v-if="k.Type === 1"
                                      style="fontSize: 20px;"
                                    >{{ k.Option }}:&nbsp;&nbsp;</span>
                                    <span v-if="k.Type === 1" v-html="k.Content" style="fontSize: 18px;display:inline-block"></span>
                                  </a-radio>
                                </a-radio-group>
                              </div>
                              <!-- 收藏/选入 -->
                              <div class="collection-select">
                                <!-- 收藏 -->
                                <span style="position: relative" @click="handleCollection(item1.ItemId,$event)" :data-collectionValue="item1.ItemId">
                                  <img :src="collection" alt="">
                                <!-- <img v-show="item1.ItemId === '11182'" style="position: absolute;left: -55px;top:-65px" :src="collectionSuccess" alt="" :data-src="item1.ItemId" > -->
                                </span>

                                <!-- 共享选入 -->
                                <span class="select-title" @click="singleQuote(item1.ItemId, i.PaiperId, $event, item1.Title)">选入</span>
                              </div>
                            </div>
                          </div>

                        </div>
                      </a-drawer>
                      <p></p>
                      <p><img src="@/assets/teacher/hei编组.png" alt=""></p>
                      <p>{{ i.Title }}</p>
                      <p><a-rate :default-value="i.DifficultyStar" :count="i.DifficultyStar" disabled />&nbsp;&nbsp;&nbsp; <span>已引用{{ i.QuoteCount }}次</span></p>
                      <p><img src="@/assets/teacher/三条线.png" alt=""></p>
                    </li>
                  </ul>
                </div>
              </div>
            </a-drawer>
          </div>
          <p><span class="basic-btn" style="background-color:#D8D5D6">云题库</span></p>
          <!-- @click="handleLocalImport" -->
          <p><span style="backgroundColor: rgb(216, 213, 214)" class="basic-btn bgc">本地练习导入</span></p>
          <a-modal
            v-model="localImport"
            :header="null"
            :footer="null"
            :bodyStyle="{ padding: '40px',backgroundColor: '#68BB97'}"
            centered
          >
            <p class="import-practice">
              <span class="fl"><img src="@/assets/teacher/试卷1.png" alt="">导入练习题</span>
              <span class="fg">*文件支持Office/word/wps文档</span>
            </p>
            <div class="upload-word">

              <a-upload-dragger

                name="file"
                :multiple="true"
                :disabled="isDisabledUpload"
                :action="uploadWordUrl"
                @change="handleChangeUpload"
              >
                <img src="@/assets/teacher/云上传.png" alt="">
                <p>点击上传本地文件</p>
                <p>或</p>
                <p>拖拽word文档到此框内上传</p>
                <p style="">注:一次只能导入一张试卷</p>
              </a-upload-dragger>
            </div>
          </a-modal>
          <p><span class="basic-btn bgc" @click="showMyTask">我的题库</span></p>
          <a-drawer
            title=""
            placement="right"
            :visible="MyTask"
            @close="onCloseMyTask"
            width="60%"
            :drawerStyle="{ backgroundColor: '#FAF7F8' }"
            :bodyStyle="{ padding: '50px'}"
          >
            <!-- 题库信息 -->
            <div class="question-bank">
              <p class="span-btn">题库: <span @click="MySubjectBank" :class="{ 'sele': this.MyGetTitleBank === 1}">我的题库</span><span @click="MyCollection" :class="{ 'sele': this.MyGetTitleBank === 0}">我的收藏</span></p>
              <p>年级:&nbsp;&nbsp;&nbsp;
                <a-select
                  style="width: 350px"
                  placeholder="年级选择"
                  @change="handleSelectChangeGrade"
                >
                  <a-select-option v-for="item in gradeInfor" :key="item.Id" :value="item.Id">
                    {{ item.Gradename }}
                  </a-select-option>
                </a-select>
              </p>
              <p>
                知识点:&nbsp;&nbsp;&nbsp;
                <a-cascader
                  :field-names="{ label: 'ChapterName', value: 'ChapterId', children: 'Seconds' }"
                  style="width: 349px"
                  :options="optionsTitle"
                  change-on-select
                  @change="onChangeKnowledge"
                  placeholder="全部题目"/>
              </p>
              <p class="question-type">题型:
                <span :class="{ 'col-title': ItemTypeTitle == 0 }" @click="allItemType(0)">全部</span>
                <span :class="{ 'col-title': ItemTypeTitle == item.Id }" @click="SubjectSelect(item.Id)" v-for="item in subjectList" :key="item.Id">{{ item.ItemTypeName }}</span>
              </p>
              <ul class="difficulty">
                <li>难度:</li>
                <li :class="{ 'yellow': yel === 0 }" @click="Difficulty(0)">全部</li>
                <li :class="{ 'yellow': yel === 1 }" @click="Difficulty(1)">☆</li>
                <li :class="{ 'yellow': yel === 2 }" @click="Difficulty(2)">☆☆</li>
                <li :class="{ 'yellow': yel === 3 }" @click="Difficulty(3)">☆☆☆</li>
                <li :class="{ 'yellow': yel === 4 }" @click="Difficulty(4)">☆☆☆☆</li>
                <li :class="{ 'yellow': yel === 5 }" @click="Difficulty(5)">☆☆☆☆☆</li>
              </ul>
            </div>
            <div class="clearfix">
              <p class="fl"><span class="m-r">默认</span><span>使用次数&nbsp;&nbsp;<span style="fontSize:">↓</span></span></p>
            <!-- <p class="fg select-color">一键全部引用</p> -->
            </div>
            <!-- 我的题库 题库题目 -->
            <div v-if="mySubBank" class="my-title-bank">
              <div
                v-for="(ite, inde) in SubjectBank"
                :key="inde"
                @click="IndividualSubject(ite.ItemId)"
                class="clearfix subject-tip"
              >
                <p class="fl">{{ inde + 1 }}.</p>
                <div class="img-div" v-html="ite.Title" style="padding: 0 20px;">
                </div>
                <!-- 带有选项类的题 -->
                <div class="paper-title" v-show="ite.ItemTypeId !== 5">
                  <P v-for="(t,n) in ite.AuditPaperItemAnswers" :key="n" v-show="t.Type === 1">
                    <span>{{ t.Option }}</span> : <span style="display: inline-block;" v-html="t.Content"></span>
                  </P>
                  <span v-if="ite.IsSelected === 0" @click="selectMySubject(ite.ItemId,ite.TypeId, $event, ite.Title)" class="select-title-itemType">选入</span>
                  <span v-if="ite.IsSelected === 1" @click="deleteMySubject(ite.ItemId,ite.TypeId, $event, ite.Title)" class="select-title-itemType">移除</span>
                </div>
                <p class="title-data"><span>{{ ite.TypeName }}</span>&nbsp;|&nbsp;<a-rate disabled :count="ite.Level" v-model="ite.Level">&nbsp;|&nbsp;</a-rate>&nbsp;|&nbsp;<span>引用次数{{ ite.OrganizationPaperCount }}</span></p>
              </div>
              <a-pagination
                style="text-align:center"
                show-quick-jumper
                :default-current="1"
                :total="totalMyTitle"
                @change="onChangeMyTitle"
                hideOnSinglePage />
            </div>
            <!-- 我的题库 我的收藏 -->
            <div v-if="myCollection" style="background-color: #fff">
              <div v-for="(item,index) in myCollectionList" :key="item.ItemId" class="clearfix subject-tip">
                <p class="fl">{{ index + 1 }}.</p>
                <div v-html="item.Title" style="padding: 0 20px;">
                </div>
                <!-- 带有选项类的题 -->
                <div class="paper-title" v-show="item.ItemTypeId !== 5">
                  <P v-for="(t,n) in item.Options" :key="n" v-show="t.Type === 1">
                    <span>{{ t.Option }}</span> : <span style="display: inline-block;" v-html="t.Content"></span>
                  </P>

                </div>
                <span style="position: absolute; right: 221px;bottom: 9px;" class="cur">
                  <img :src="myCollectionType" alt="">
                </span>
                <span class="select-title-itemType" @click="singleQuote(item.ItemId,item.PaperId,$event)" style="right:116px">选入</span>
                <span class="select-itemType select-title-itemType" @click="deleteMyColletion(item.ItemId)">移除收藏</span>
              </div>
            </div></a-drawer>
        </div>

        <!-- 题目属性 -->
        <div class="subject-attribute">
          <p>试卷属性</p>
          <p>年级: {{ gradeName }}</p>
          <p>知识点: {{ KnowledgeName }}</p>
          <p>题数: <span class="title-num">{{ paperNum }}</span> 题</p>
          <!-- level -->
          <p>难度: <a-rate v-if="defaultStar !== 0" :defaultValue="defaultStar" @change="handleDifficultySelect"></a-rate></p>
          <p>分数: <span class="title-num">{{ paperTotalScore }}</span> 分</p>
          <p>剩余分数: <span class="title-num">{{ surplusScore }}</span> 分</p>
          <p>总分值: 100分</p>
        </div>
        <!-- 题目目录 -->
        <div class="subject-menu">
          <p v-if="this.arr.length !== 0" style="font-size:20px;">题目目录</p>
          <div class="subject-type" v-for="(item,index) in arr" :key="index" @mouseenter="handleOrder(index)">
            <p>{{ index + 1 }}.{{ item.Name }} (共{{ item.Score }}分) <span class="cur fg" @click="removeRightBigTitle(item.Id)"><img src="@/assets/teacher/删除按钮.png" alt=""></span></p>
            <draggable group="article" :value="item.Items" @input="handleListChange($event,item.Items)">

              <span class="mark" v-for="(ite,j) in item.Items" :key="j">
                <span :class="{ 'red': ite.Score === 0 }">{{ j + 1 }}</span>
                <p :class="{ 'red': ite.Score === 0 }">{{ ite.Score }}分</p>
              </span>
            </draggable>
            <a-modal
                  v-model="deleteBigTypeRight"
                  title="温馨提示"
                  width="630px"
                  @ok="handleDeleteBigRightType(item.Id)"
                  :maskStyle="{ backgroundColor: 'rgba(1,1,1,0.2)'}"
                  centered
                >
                  <div>
                    确定删除吗？
                  </div>
                </a-modal>

          </div>

        
        
        <!-- <div class="subject-type">
            <p>二.填空题 (共15分) <span class="fg"><img src="@/assets/teacher/删除按钮.png" alt=""></span></p>
            <span class="mark">
              <span>1</span>
              <p>5分</p>
            </span>
            <span class="mark">
              <span>2</span>
              <p>5分</p>
            </span>
            <span class="mark">
              <span>3</span>
              <p>5分</p>
            </span><span class="mark">
              <span class="red">4</span>
              <p class="red">0分</p>
            </span>
          </div> -->
           
        </div>
      </div>

      <!-- 本地导入试卷,抽屉 a-drawer -->
      <a-drawer
        title=""
        width="55%"
        :visible="importPaperVisible"
        :body-style="{ paddingBottom: '80px' }"
        @close="onImportPaperClose"
      >

        <div class="importPaperTitle">
          <span>{{ localPaperName }}</span>
        </div>

        <div class="clearfix" style="margin-top: 15px">
          <span class="importPaperQuote cur" type="link" @click="OnekeyQuote">
            一键全部引用
          </span>
        </div>
        <!-- 试卷内容 -->

        <!-- 试卷 -->
        <div v-for="(item,index) in LocalUploadpaperData" :key="index">
          <div
            v-for="(ite, inde) in item.Items"
            :key="inde"
            class="clearfix local-practice-test"
            style="">
            <p class="fl">{{ inde + 1 }}.</p>
            <div class="local-practice-img" v-html="ite.Title">
            </div>
            <!-- 带有选项类的题 -->
            <div class="paper-title" v-show="ite.ItemTypeId !== 5">
              <P v-for="(t,n) in ite.Options" :key="n" v-show="t.Type === 1">
                <span>{{ t.Option }}</span> : <span style="display: inline-block;" v-html="t.Content"></span>
              </P>
            </div>
            <!-- 收藏/选入 -->
            <div class="local-collection-select">
              <!-- 收藏 -->
              <span style="position: relative" @click="handleCollection(ite.ItemId,$event)" :data-collectionValue="ite.ItemId">
                <img :src="collection" alt="">
              <!-- <img v-show="item1.ItemId === '11182'" style="position: absolute;left: -55px;top:-65px" :src="collectionSuccess" alt="" :data-src="item1.ItemId" > -->
              </span>

              <!-- 本地练习选入 -->
              <span @click="singleQuote(ite.ItemId,oneQuote,$event)" class="select-title">选入</span>
            </div>
          </div>

        </div>

      </a-drawer>
    </div>
    <p class="footer">Copyright©上海有我科技有限公司，All Rights Reserved</p>
    <!-- <footerLogo></footerLogo> -->
  </div>
</template>

<script>
import footerLogo from '@/components/XuHuiFooter/FooterLogo'
import TinyMceEditor from '@/components/TinyMCE/tinyMce'
// import 'tinymce/icons/default/icons'
import WangEdit from '@/components/WangEditor/WangEditor'
import SynonymAnswer from '@/components/AddSynonym/AddSynonym'
import '@/utils/utils.less'
import moment from 'moment'
import Draggable from 'vuedraggable'

export default {
  components: {
    footerLogo,
    WangEdit,
    SynonymAnswer,
    Draggable,
    TinyMceEditor
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    // this.paperName = this.$route.query.name
    this.level = this.$route.query.level
    this.level = parseInt(this.level)
    this.grade = this.$route.query.grade
    console.log('年级======', this.grade)
    this.paperId = this.$route.query.paperId
    this.name = this.$route.query.name
    this.knowledge = this.$route.query.knowledge
    this.uploadWordUrl = `${this.$rootUrl}/Paper/Exam_Paper/Word?subjectId=2&createId=${this.userId}`
    // 年级
    this.gradeName = this.$route.query.gradeName
    // 知识点
    this.KnowledgeName = this.$route.query.KnowledgeName

    this.importPaperVisible = false

    // 测试
    // this.getPaperInfo('1296631536128364544')
    // 获取题干
    // this.getSubject()
    this.getSubjectInfo()
    // 获取题型
    this.getSubjectType()
  },
  mounted () {

  },
  watch: {
    $route () {
      const ChangId = window.location.href.split('?')[1]
      if (this.$route.fullPath === '/Teacher/My_Task/ArrangmentTaskBasics?' + ChangId) {
        // debugger
        // this.paperName = this.$route.query.name
        this.paperId = this.$route.query.paperId
        this.getSubjectInfo()
      }
    }
    // MakeTopic: {
    //   handler (newValue, old) {
    //     console.log(newValue, old)
    //     if (newValue) {
    //       history.pushState(null, null, location.href)
    //       window.addEventListener('popstate', function (e) {
    //       // alert('我监听到了浏览器的返回按钮事件啦')// 根据自己的需求实现自己的功能
    //         // debugger
    //         this.MakeTopic = false
    //         console.log('编辑题目的---', this.MakeTopic)
    //         window.history.pushState(null, null, location.href)
    //         // stay()
    //       }, false)
    //     }
    //   }
    // }
  },
  data () {
    return {
      // 试题总数
      itemCount: 0,
      // 主观答案
      subjective: '',
      // 某个年级
      gradeName: '',
      // 右侧知识点
      KnowledgeName: '',
      // 单题难度
      aloneLevel: 0,
      // 单题难度提示
      selectCheckLevel: false,
      uploadWordUrl: '',
      isDisabledUpload: false,
      importPaperVisible: false,
      importPaperItems: [],
      BigTitle: false,
      bigTitleDefault: true,
      current: 1,
      total: 0,
      selectTeacher: false,
      handleSelectTeacherDetermine: [],
      friendId: '',
      alonevalue: '',
      moment,
      MultipleChoice: [],
      // 本地上传的试卷信息
      LocalUploadpaperData: [],
      // 本地上传的试卷名称
      localPaperName: '',
      // 判断题的选择
      judge: '',
      // 勾选图标
      checkIcon: require('@/assets/teacher/未勾选.png'),
      // 试卷id
      paperId: '',
      // 试卷名称
      name: '',
      // 知识点id
      knowledge: '',
      // 试卷总分
      paperTotalScore: '',
      // 试卷剩余分数
      surplusScore: '',
      // 试卷题数
      paperNum: '',
      // 题型自身ID
      myTypeQuestionId: '',
      // 题目题干
      subjectListTitle: '',
      // 题目解析
      subjectAnalysis: '',
      arr: [],
      // 单选题所有的
      SingleChoice: [],
      // 单个题的分数
      aloneFraction: '',
      // 题目难度
      level: '',
      // 年级
      grade: '',
      // 制作题目
      MakeTopic: false,
      // 编辑题干
      subTitle: '',
      // 编辑解析
      subAnalysis: '',
      // 编辑选项性质
      selectValue: '',
      // 编辑选择答案
      answer: '',
      // 编辑单个题目设定得分
      SetUpFraction: false,
      // 设定得分师单个题目的id
      singleQuestionId: '',
      // 编辑单个题目的Id
      singleEditQuestionId: '',
      // 单个题目设置
      IndividualSettings: false,
      // 用户id
      userId: '',
      // 自定义试卷名称
      paperName: '',
      // 知识点
      options: [],
      // 默认选择空
      subOptions: [ { Option: 'A' }, { Option: 'B' }, { Option: 'C' } ],
      subJudgeOptions: [{ Option: 'A', Content: '对' }, { Option: 'B', Content: '错' }],
      // 默认填空空
      completion: ['', '', ''],
      // 添加同意答案数量
      counter: [{}],
      // 同义答案的显示隐藏
      synonym: false,
      // 同义答案的回显
      EchoSynonymousAnswer: [],
      num: '',
      // 整体题型编辑
      wholeEdit: true,
      // 取消编辑弹窗
      EditCance: false,
      // 编辑设定多题的得分
      multipleScore: '',
      // 编辑设定多题的得分
      score: false,
      // 编辑设定多题的得分题型ID
      multipleSubjectId: '',
      // 编辑设定多题名称
      scoreName: '',
      // 题型数据
      subjectList: [],
      // 题型编辑显示
      titleType: '',
      // 题型显示
      questionType: '请选择题型',
      // 题型ID
      questionId: '',
      // 题型ItemType
      questionItemType: '',
      // 本地练习导入
      localImport: false,
      // 作业协同
      VisibleTask: false,
      // 作业协同教师数据
      taskTeacherList: [],
      // 作业协同教师数据对应试卷
      taskTeacherListPaper: {},
      // 星标hao'you
      starFriendsList: [],
      teacherSelectList: [],
      // 预览
      Preview: false,
      // 预览老师
      PreviewTeacher: {},
      // 预览试卷
      PreviewPaper: {},
      // 预览试卷数据
      paperData: [],
      // 预览试卷数据收藏
      collection: require('@/assets/teacher/收藏.png'),
      // 预览收藏成功
      collectionSuccess: require('@/assets/teacher/收藏成功.png'),
      // 我的收藏
      myCollectionType: require('@/assets/teacher/已收藏.png'),
      // 我的题库
      MyTask: false,
      yel: 0,
      // 选择题型
      visible: false,
      // 发布时间
      time: false,
      value1: 0,
      valueArea: '全校',
      // 收藏value
      collectionValue: '',
      // 地区
      Area: false,
      // 老师对应班级
      classInfor: [],
      // 发布的班级
      ClassId: [],
      // 自定义发布时间
      value: 1,
      // 自定义开始时间
      PublishDateTime: '',
      comparePublishTime: '',
      // 作答截止时间
      AnswerEndDateTime: '',
      compareAnswerEndTime: '',
      // 是否立即发送
      ImmediatelyPublish: true,
      allSynonymAnswer: [],
      synonymousAnswer: '',
      // 共享位置
      IsShare: 0,
      radioStyle: {
        display: 'block',
        height: '30px',
        lineHeight: '30px'
      },
      blanksUp: [],
      // 一键引用的类型id
      oneQuoteParentId: [],
      // 一键引用的typeId
      oneQuoteTypeId: [],
      // 我的题库
      SubjectBank: [],
      // 我的题库，知识点
      optionsTitle: [],
      // 年级
      gradeInfor: [],
      // 题型选择
      ItemTypeTitle: 0,
      pageNo: 1,
      // 难度
      levelTitle: 0,
      // 我的题库知识点
      chapterTitle: '',
      // 我的题库分页
      totalMyTitle: 1,
      // 是否为引用的题目
      IsQuote: '',
      // 我的题库
      mySubBank: true,
      // 我的收藏
      myCollection: false,
      //
      MyGetTitleBank: 1,
      //  我的题库年级
      myGrade: '',
      myCollectionList: [],
      // 删除大题模块弹窗
      deleteBigType: false,
      deleteBigTypeRight: false,
      // 删除单题的弹窗
      deleteAloneList: false,
      // 改变顺序
      orderIndex: '',
      // 题干的校验
      titleCheck: false,
      // 解析的校验
      analysisCheck: false,
      // 选项的校验
      selectCheck: false,
      // 添加唯一的题型
      isAdd: '',
      // 是否保存
      isSave: 0,
      Selected: '选入',
      re: '',
      // 我的题库年级
      gradeId: 0,
      // 默认右侧难度
      defaultStar: 0,
      fraction: false,
      deleteAloneId: ''

    }
  },
  methods: {
    toAddFriends () {
      this.VisibleTask = false
      this.$router.push({ path: '/Teacher/My_Friends/AddFriend' })
    },
     // 不可选择时间
    disabledDate (current) {
      // debugger
      // 不可选过去时间add(-1, 'd')  subtract(0, 'day')
      return current <  moment().add(-1, 'd')
      // 不可选未来时间
      // return current && current > Date.now()
    },
     disabledDate1 (current) {
      // debugger
      // 不可选过去时间add(-1, 'd')  subtract(0, 'day')
      return current <  moment().subtract(0, 'day')
      // 不可选未来时间
      // return current && current > Date.now()
    },
    // 默认选择发布时间
    disabledDateTime () {
      return {
        disabledHours: () => this.range(0, 24).splice(4, 20),
        disabledMinutes: () => this.range(30, 60),
        disabledSeconds: () => [55, 56]
      }
    },
    // 更新位置下标
    handleOrder (index) {
      console.log('INDEX==', index)
      this.orderIndex = index
    },
    // 更新位置
    handleListChange (event, items) {
      console.log(event)
      this.arr[this.orderIndex].Items = event
      console.log('ITEMS==', items)
      // this.getSubjectInfo()
    },
    // 单题难度
    handleAloneLevel (level) {
      console.log('单题的难度选择333', level)
      this.aloneLevel = level
    },
    // 修改试卷的难度
    handleDifficultySelect (value) {
      console.log('修改试卷的难度ID---', value)
      this.$http.post('/Paper/Exam_Paper/UpdatePaperDifficulty', { paperId: this.paperId, level: value }).then(res => {
        console.log('修改试卷的难度---', res)
      })
    },
    pushHistory () {
      // var state = {
      //   title: 'title',
      //   url: '#'
      // }
      window.history.pushState(null, null, window.location.href)
    },
    // 编辑试卷名称
    editBigTile () {
      this.BigTitle = true
      this.bigTitleDefault = false
    },
    cancleTitle (e) {
      // this.BigTitle = false
      // e.stopPropagation()
      // e.preventDefault()
      var imgs = document.getElementById('myTitleBtn')
      var btn = document.getElementById('myTitle')
      if (btn) {
        if (!btn.contains(e.target) && !imgs.contains(e.target)) { // 按钮.app-download以外的区域
          this.BigTitle = false
          this.bigTitleDefault = true
          console.log(this.paperName)
          this.$http.post('/Paper/Exam_Paper/ChangePaperTitle', { paperid: this.paperId, paperName: this.paperName }).then(res => {
            console.log('修改试卷名称---', res)
            this.getSubjectInfo()
          })
        }
      }
    },
    selectTeacherList () {
      this.selectTeacher = !this.selectTeacher
      if (this.selectTeacher === true) {
        this.friendId = ''
        this.handleSelectTeacherDetermine = []
      }
    },
    keyUp () {
      // console.log(this.subTitle)
      const re = /（(.*?)）/g
      let temp = []
      const a = []
      setTimeout(() => {
        while (temp = re.exec(this.subTitle)) {
          // debugger
          // console.log(temp[1])
          // debugger
          a.push(temp[1])
          // console.log(a)
        }
      }, 300)
      this.blanksUp = a
      // console.log(this.blanksUp)
      // console.log(this.subTitle)
      // const regex = /\（.+?\）/g
      // console.log(this.subTitle.match(regex))
      // /（(.*)）/g.exec(this.subTitle)
    },
    // 共享全区和全校的选项
    onChangeAreaSchool (checkedValues) {
      console.log('共享全区和全校的选项', checkedValues)
    },
    onChangeduo (checkedValues) {
      console.log('checked = ', checkedValues)
      this.answer = checkedValues
    },
    onChangeSlect (e) {
      console.log(e.target.value)
      this.answer = e.target.value
      console.log('答案---', this.answer)
    },
    // 设置多题得分
    setScore (id) {
      console.log('题型id---', id)
      this.score = true
      this.multipleSubjectId = id
    },
    // 设置多题得分确定
    handleBigTitleScore () {
      console.log('题目类型ID---', this.multipleSubjectId)
      this.$http.post('/Paper/Exam_PaperItemMapping/BatchUpdateScore', { questionTypeId: this.multipleSubjectId, score: this.multipleScore }).then(res => {
        if (res.Success) {
          console.log('批量设定得分---', res)
          this.score = false
          this.getSubjectInfo()
        }
      })
    },
    // 设定单个题的得分
    SetUp (id) {
      this.SetUpFraction = true
      console.log('当前题目id---', id)
      this.singleQuestionId = id
      // console.log('题型---', this.questionType)
    },
    // 确认单个题的得分
    handleFraction () {
      console.log('单个题的得分ID', this.singleQuestionId)
      this.$http.post('/Paper/Exam_PaperItemMapping/UpdateScore', { id: this.singleQuestionId, score: this.aloneFraction }).then(res => {
        console.log('设定单个题目分数---', res)
        if (res.Success) {
          this.SetUpFraction = false
          this.getSubjectInfo()
        }
      })
    },
    // 单题设置操作按钮控制
    IndividualSubject (itemId) {
      this.IndividualSettings = itemId
      // console.log('单题设置操作按钮控制', this.IndividualSettings)
      // this.aa = '2'
    },
    // 整体题目编辑
    editSubject (id, TypeId, itemType) {
      this.subTitle = ''
      this.subAnalysis = ''
      // debugger
      this.answer = ''
      this.aloneLevel = 0
      this.titleCheck = false
      this.analysisCheck = false
      this.IsQuote = 0
      // 选项的校验
      this.selectCheck = false
      // this.subOptions.Content = ''
      this.subOptions = [ { Option: 'A', Content: '' }, { Option: 'B', Content: '' }, { Option: 'C', Content: '' } ]
      this.subJudgeOptions = [{ Option: 'A', Content: '对' }, { Option: 'B', Content: '错' }]
      this.MultipleChoice = []
      this.alonevalue = ''
      // 判断题的选择
      this.judge = ''
      this.blanksUp = []
      console.log(this.subOptions.Content)
      // this.subOption.Content = ''
      // this.aa = '1'
      this.MakeTopic = true
      this.singleEditQuestionId = ''
      this.questionId = TypeId
      // itemType
      this.questionItemType = itemType
      console.log(id)
      // 题型自身Id
      this.myTypeQuestionId = id
      // 默认类型
      this.selectValue = +TypeId
    },
    // 单个题目编辑
    singleEditSubject (id, TypeId, itemType, parentId, IsQuote) {
      this.MakeTopic = true
      this.questionId = TypeId
      // this.answer = ''
      console.log('当前题目id--', id)
      this.singleEditQuestionId = id
      // parentId
      this.myTypeQuestionId = parentId
      // itemType
      this.questionItemType = itemType
      // itemtype保存
      // 默认类型
      this.selectValue = +TypeId
      // 是否是引用的题目
      this.IsQuote = IsQuote
      console.log('是否是引用的题目--', this.IsQuote)
      // 回显单个题目题干
      this.handleEchoSubject(id)
    },
    // 回显单个题目题干
    handleEchoSubject (id) {
      this.$http.post('/Paper/Exam_Item/GetTheData', { id }).then(res => {
        console.log('题干---', res)
        this.subTitle = ''
        this.subTitle = res.Data.Title
        this.aloneLevel = res.Data.Level
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetAnalysisDataByItemId', { itemId: id }).then(res => {
        console.log('解析---', res)
        this.subAnalysis = ''
        this.subAnalysis = res.Data.Content
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetAnswerDataByItemId', { itemId: id }).then(res => {
        console.log('答案---', res)
        this.answer = ''
        this.alonevalue = ''
        this.judge = ''
        this.answer = res.Data.Content
        this.alonevalue = res.Data.Content
        this.judge = res.Data.Content
        debugger
        if (res.Data.Content.length > 1) {
          this.MultipleChoice = res.Data.Content.split('|')
        }
        console.log('多选答案---', this.MultipleChoice)
      })
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetOptionsByItemId', { itemId: id }).then(res => {
        console.log('选项---', res)
        this.subOptions = '' // []
        this.subOptions = res.Data
      })
      this.$http.post('/Paper/Exam_Item/GetMoreItemAnswer', { itemId: id }).then(res => {
        console.log('同义答案---', res)
        // this.subOptions = res.Data
        this.EchoSynonymousAnswer = ''
        this.EchoSynonymousAnswer = res.Data
      })
      // 题目详情（难度）
      this.$http.post('/Paper/Exam_Item/GetTheData', { Id: id }).then(res => {
        console.log('题目难度以及详情---', res)
      })
    },
    // 单选题增加
    AddOptions () {
      switch (this.subOptions.length) {
        case 0:
          this.subOptions.push({
            Option: 'A',
            Content: ''
          })
          break
        case 1:
          this.subOptions.push({
            Option: 'B',
            Content: ''
          })
          break
        case 2:
          this.subOptions.push({
            Option: 'C',
            Content: ''
          })
          break
        case 3:
          this.subOptions.push({
            Option: 'D',
            Content: ''
          })
          break
        case 4:
          this.subOptions.push({
            Option: 'E',
            Content: ''
          })
          break
        case 5:
          this.subOptions.push({
            Option: 'F',
            Content: ''
          })
          break
        case 6:
          this.subOptions.push({
            Option: 'G',
            Content: ''
          })
          break
        default:
          this.$message.success('最多加到G哦')
      }
      // 填空增加
      this.completion.push('')
    },
    // 减少选项
    ReduceOptions () {
      // 选择减少
      this.subOptions.pop()
      // 填空减少
      this.completion.pop('')
    },
    // 添加同义答案按钮
    AddSynonym (index) {
      this.num = index
      this.synonym = true
      this.counter = [{}]
      console.log(this.num)
      if (this.synonymousAnswer !== '') {
        this.allSynonymAnswer.push({
          index: this.synonymousAnswers
        })
        console.log('所有同义答案的值--', this.allSynonymAnswer)
      }
      console.log('修改后的所有空值', this.mulAnswer)
    },
    get (data, sd) {
      console.log(data, sd)
      const obj = {}
      obj['Content'] = data
      this.allSynonymAnswer.push(obj)
      console.log('对应填空题的同义答案---', this.allSynonymAnswer)
    },
    // 增加同义答案的输入框
    syAdd () {
      this.counter.push({})
    },
    // 减少同义答案的输入框
    syDel () {
      this.counter.pop()
      this.allSynonymAnswer.pop()
      console.log('对应填空题的同义答案---', this.allSynonymAnswer)
      if (this.counter.length === 0) {
        this.counter = [{}]
      }
    },
    // 删除同义答案
    handleEchoSynonymousAnswer (syn) {
      this.$http.post('/Paper/Exam_Item/RemoveAnswer', { itemId: this.singleEditQuestionId, value: syn }).then(res => {
        console.log('删除同义答案---', res)
        if (res.Success) {
          this.handleEchoSubject(this.singleEditQuestionId)
        }
      })
    },
    // 保存编辑题目
    handleSave () {
      debugger
      console.log('题干---', this.subTitle)
      console.log('选项类型---', this.selectValue)
      console.log('解析---', this.subAnalysis)
      console.log('选项---', this.subOptions)
      console.log('填空---', this.completion)
      console.log('答案---', this.answer)
      console.log('题型ItemId---', this.questionItemType)
      console.log('自身类型id---', this.myTypeQuestionId)
      console.log('编辑单个题目id--', this.singleEditQuestionId)
      console.log('同意答案---', this.allSynonymAnswer)
      console.log('引用的id---', this.IsQuote)
      console.log('主观答案---', this.subjective)
      // console.log(this.answer.split(''))
      if (this.selectValue === 23) {
        this.answer = this.subjective
      }
      // 答案的格式编写
      this.saveAnswerWrite()
      // 保存提示
      this.checkSaveRule()
      // const reg = /(<br\s*\/?>)|\s/gi
      // this.subTitle = this.subTitle.replace(reg, '<br/>')
      console.log('保存里的isSave---', this.isSave)
      if (this.isSave === 0) {
        return false
      } else {
        if (this.IsQuote === 1) {
        // debugger
        // 更改引用保存题目
          this.$http.post('/Paper/Exam_Item/UpdateQuoteItem', {
            parentId: this.myTypeQuestionId,
            Analysis: this.subAnalysis,
            Answer: this.answer,
            ItemType: this.questionItemType,
            Title: this.subTitle,
            Options: this.subOptions,
            OtherAnswer: this.allSynonymAnswer,
            Id: this.singleEditQuestionId,
            Chapter: this.knowledge,
            PaperId: this.paperId,
            GradeTerm: this.grade,
            TypeId: this.selectValue
          }).then(res => {
            console.log('引用题目的编辑-----', res)
            if (res.Success) {
              this.$message.success('保存成功')
              this.getSubjectInfo()
              this.MakeTopic = false
            } else {
              this.$message.error(res.Msg)
            }
          })
        } else {
          // 初始保存题目
          console.log('保存里的答案---', this.answer)
          this.$http.post('/Paper/Exam_Item/SaveItemInfo', {
            analysis: this.subAnalysis,
            itemType: this.questionItemType,
            title: this.subTitle,
            options: this.subOptions,
            gradeTerm: this.grade,
            typeId: this.selectValue,
            answer: this.answer,
            parentId: this.myTypeQuestionId,
            paperId: this.paperId,
            // 知识点id
            chapter: this.knowledge,
            // 单题的id（用于修改）
            id: this.singleEditQuestionId,
            // 同意答案
            OtherAnswer: this.allSynonymAnswer,
            // 题目难度
            level: this.aloneLevel
          }).then(res => {
            console.log('保存---', res)
            if (res.Success) {
              this.$message.success('保存成功')
              this.getSubjectInfo()
              this.MakeTopic = false
            } else {
              this.$message.error(res.Msg)
            }
          })
        }
      }
    },
    // 校验保存规则
    checkSaveRule () {
      console.log(this.selectValue)
      // debugger
      const aaa = this.subOptions.some(item => {
        return item.Content === ''
      })
      this.selectCheck = true
      if (this.subTitle === '') {
        this.titleCheck = true
        this.isSave = 0
      } else if (this.subTitle !== '') {
        this.titleCheck = false
      }
      if (this.selectValue === 5) {
        this.isSave = 1
      } else {
        // if (this.subTitle)
        if (aaa) {
          this.isSave = 0
        } else if (!aaa) {
          this.isSave = 1

          // if (this.subAnalysis === '') {
          //   this.analysisCheck = true
          //   this.isSave = 0
          // } else if (this.subAnalysis !== '') {
          //   this.analysisCheck = false
          // }
        }
      }
      if (this.aloneLevel === 0) {
        this.isSave = 0
        this.selectCheckLevel = true
      } else {
        this.isSave = 1
        this.selectCheckLevel = false
      }
      // debugger
    },
    // 保存时答案格式编写
    saveAnswerWrite () {
      if (this.selectValue === 5) {
        // const blanks = this.completion.substring(heade + 1, end).split('').join('|')
        const blanks = this.completion.map(item => {
          return item.replace(/<[^<>]+>/g, '')
        })
        const subBlanks = blanks.join('|')
        console.log(subBlanks)
        this.answer = subBlanks
      }
      console.log('填空修改答案---', this.answer)
      debugger
      console.log(typeof (this.answer))
      if (this.answer.length > 1 && this.selectValue === 10 && typeof (this.answer) !== 'string') {
        if (this.answer === '') {
          return
        } else if (this.answer.length === 1) {
          return this.$message.success('多选题答案不能唯一')
        } else {
          this.answer = this.answer.join('|')
        }
      }
      if (this.selectValue === 11) {
        this.subOptions = this.subJudgeOptions
      }
      console.log('修改的答案--', this.answer)
    },
    // 获取试卷的信息 重新渲染
    getSubjectInfo () {
      this.$http.post('/Paper/Exam_PaperItemMapping/GetPaperInfoById', {
        paperid: this.paperId
      }).then(res => {
        console.log('保存后的试卷题目---', res)
        this.itemCount = res.Data.ItemCnt
        this.defaultStar = res.Data.DifficultyStar
        console.log('默认的难度---', this.defaultStar)
        this.arr = res.Data.QuestionTypes
        // const reg = /(#@)/g
        const reg = /(#&\d+@)/g
        this.arr.forEach(value => {
          value.Items.forEach(val => {
            val.Title = val.Title.replace(reg, ' ')
          })
        })
        // 试卷总分
        this.paperTotalScore = res.Data.TotalScore
        // 剩余分数
        this.surplusScore = 100 - this.paperTotalScore
        // 试卷的题数
        this.paperNum = res.Data.ItemCnt
        // 试卷名称
        this.paperName = res.Data.PaperName
        // console.log('保存后的试卷题目---', this.arr)
        this.oneQuoteParentId = res.Data.QuestionTypes.map(item => {
          return item.Id
        })
        this.oneQuoteParentId = this.oneQuoteParentId.join(',')
        this.oneQuoteTypeId = res.Data.QuestionTypes.map(item => {
          return item.TypeId
        })
        this.oneQuoteTypeId = this.oneQuoteTypeId.join(',')
        // console.log('==========', this.oneQuoteParentId)
      })
    },
    // 删除单个题目
    deleteAlone (id) {
      this.deleteAloneList = true
      this.deleteAloneId = id
      console.log('单个题目的id', id)
    },
    // 删除单个题目确定
    handleDeleteAloneType (id) {
      
      debugger
      console.log('确定单个题目的id', id)
      this.$http.post('/Paper/Exam_PaperItemMapping/Remove', { id: this.deleteAloneId }).then(res => {
        console.log('删除单个题目---', res)
        if (res.Success) {
          this.$message.success('删除成功')
          this.deleteAloneList = false
          this.getSubjectInfo()
        } else {
          this.$message.error('删除失败')
        }
      })
    },
    // 关闭编辑题目
    onCloseMakeTopic () {
      this.MakeTopic = false
      this.$forceUpdate()
    },
    // 取消题目编辑
    cancelEdit () {
      this.EditCance = true
      // this.MakeTopic = false
    },
    // 取消弹框题目编辑
    handleCancelEdit () {
      this.EditCance = false
      setTimeout(() => { this.MakeTopic = false }, 300)
    },
    // 确定题目编辑
    handleOkEdit () {
      debugger
      if (this.subTitle === '' || this.answer === '') {
        this.$message.error('请填写完整内容')
        // this.EditCance = false
        // setTimeout(() => { this.MakeTopic = false }, 300)
      } else {
        this.handleSave()
        this.EditCance = false
        setTimeout(() => { this.MakeTopic = false }, 300)
      }
    },
    // 获取题干内容
    // getSubject (id) {
    //   this.$http.post('/Paper/Exam_Item/GetTheData', { id: '11174' }).then(res => {
    //     console.log(res)
    //     this.subjectListTitle = res.Data.Title
    //   })
    //   // console.log(this.$route.query.id)
    // },
    // 获取单题的解析
    getSubjectAnalysis () {
      this.$http.post('/Paper/Exam_ItemOptiAnswer/GetAnalysisDataByItemId', { id: '11174' }).then(res => {
        console.log('题目解析')
      })
    },
    // 题型操作显示
    handleTitleType (Id, scoreName) {
      // e.stopPropagation()
      console.log('点击p----', Id)
      this.titleType = Id
      this.scoreName = scoreName
    },
    handleTitleHide (e) {
      console.log('离开---', 2)
      this.titleType = ''
    },
    // 处理当前时间并赋值
    handleTime () {
      const time = new Date()
      const Time = moment(time).format('YYYY-MM-DD HH:mm')
      this.PublishDateTime = Time
      console.log('立即发送时间默认值------' + this.PublishDateTime)
    },
    // 获取当天时间的 23：59
    getNewDayLastTime () {
      const lastTime = new Date(new Date(new Date().toLocaleDateString()).getTime() + 24 * 60 * 60 * 1000 - 1)
      this.AnswerEndDateTime = moment(lastTime).format('YYYY-MM-DD HH:mm')
      this.compareAnswerEndTime = moment(lastTime)
      console.log(moment(lastTime))

      console.log('获取当天时间的最后时间---', this.AnswerEndDateTime)
    },
    // 获取自定义开始时间
    StartTimeChange (data, dataString) {
      this.PublishDateTime = dataString
      this.comparePublishTime = data._d
      console.log(moment(data._d).valueOf())
      console.log('改变默认值的时间--------' + this.PublishDateTime, data._d)
    },
    // 获取作答截止时间
    TimeChange (data, dataString) {
      this.AnswerEndDateTime = dataString 
      this.compareAnswerEndTime = data._d
      console.log('截止作答时间---------' + this.AnswerEndDateTime, data._d)
    },
    // 立即发送
    handleImmediately () {
      this.ImmediatelyPublish = true
      console.log('是否立即发送---', this.ImmediatelyPublish)
    },
    // 切换自定义时间
    handleClick () {
      this.ImmediatelyPublish = false
      console.log('是否立即发送---', this.ImmediatelyPublish)
    },
    // 多选框，获取选中的班级
    classChange1 (checkedValues) {
      this.ClassId = checkedValues
      console.log('选中班级----', checkedValues)
    },
    // 显示发布时间
    handleComplete () {
      if (this.paperTotalScore !== 100) {
        return this.$message.error('总分值不正确，请重新赋分')
      }
      this.time = true
      // 获取老师对应班级
      this.$http.post('/HighFrequency/HighFrequency/GetClassByUserGrade', { userId: this.userId,grade: this.grade }).then(res => {
        this.classInfor = res.Data
        console.log('老师对应班级---', this.classInfor)
      })
      this.handleTime()
      // 当天最后时间
      this.getNewDayLastTime()
    },
    // 发布时间完成
    handleOkTime () {
      if (this.ClassId.length === 0) {
        return this.$message.error('请选择需要发布的班级')
      }
      if (this.PublishDateTime === '' || this.AnswerEndDateTime === '') {
        return this.$message.error('请注意发布时间的选择')
      }
      if (moment(this.comparePublishTime).valueOf() > moment(this.compareAnswerEndTime).valueOf()) {
        return this.$message.error('截止时间不能小于发布时间')
      }
      this.time = false
      // ItemCount 试题数量
      // ItemInfo  试题引用详情
      this.$http.post('/Paper/Exam_Paper/PublishPaper', { PaperId: this.paperId, ClassIds: this.ClassId, ImmediatelyPublish: this.ImmediatelyPublish, PublishDateTime: this.PublishDateTime, AnswerEndDateTime: this.AnswerEndDateTime, IsShare: this.IsShare, DifficultyStar: this.defaultStar, ItemCount: this.itemCount, ItemInfo: '' }).then(res => {
        console.log('完成发布---', res)
      })
      this.$router.push({ path: '/Teacher/My_Task/PracticeReleaseComplete',
        query: {
          paperId: this.paperId
        } })
    },
    // 添加题目
    handleAddSubject () {
      this.visible = true
    },
    // 获取题型
    getSubjectType () {
      this.$http.post('/Base_Manage/Exam_ItemType/GetDataList', { PageIndex: '1', PageRows: '50' }).then(res => {
        console.log('题型---', res)
        this.subjectList = res.Data
      })
    },
    // 添加题目完成
    handleOk (e) {
      // console.log('添加题目完成', e.target.value)
      this.visible = false
      // this.wholeEdit = true
      console.log('添加题目完成', this.arr)
      this.arr.some(ite => {
        console.log(this.questionType)
        // debugger
        if (this.questionType === ite.Name) {
          this.isAdd = 1
          return this.isAdd
        } else {
          this.isAdd = 0
          return this.isAdd
        }
      })
      console.log('是否添加---', this.isAdd)
      // 获取题型id,进行关联题目
      this.getSubjectId(this.questionType, this.questionId, this.isAdd)
    },
    // 添加题目选择
    handleChange (value) {
      console.log('添加题目选择---', value)
      const item = value.split(',')

      this.questionType = item[1]
      this.questionId = item[0]
      this.questionItemType = item[2]

      // console.log('选择题型title--', this.questionType)
      // console.log('Id', this.questionId)
      console.log('ItemType----', this.questionItemType) // { key: "lucy", label: "Lucy (101)" }
    },
    // 获取题型id,进行关联题目
    getSubjectId (title, itemId, isAdd) {
      console.log(title, itemId)
      if (isAdd === 1) {
        this.$message.success('不可重复添加')
      } else {
        this.$http.post('/Paper/Exam_PaperItemMapping/AddItemClassify', { paperid: this.paperId, title: title, itemTypeId: itemId }).then(res => {
        // console.log('获取题型Id---', res.Data)
        // this.oneQuoteParentId.push(res.Data)
        // console.log('==============', this.oneQuoteParentId)
          this.getSubjectInfo()
        })
      }
    },
    // 删除大题弹窗显示
    deleteBigQuestion (id) {
      console.log('未删除--', this.arr)
      this.deleteBigType = true
      // this.deleteBigId = id
    },
    // 删除大题确认
    handleDeleteBigType (id) {

      console.log(id)
      debugger
      this.$http.post('/Paper/Exam_PaperItemMapping/Remove', { id: this.titleType }).then(res => {
        console.log('删除大标题题目---', res)
        if (res.Success) {
          // const arr1 = this.arr.filter(item => {
          //   console.log(item)
          //   return item.Id !== id
          // })
          // this.arr = arr1
          this.getSubjectInfo()
          this.deleteBigType = false
          this.$message.success('删除成功')
        } else {
          this.$message.error('删除失败')
        }
      })
    },
    handleDeleteBigRightType (id) {

      console.log(id)
      debugger
      this.$http.post('/Paper/Exam_PaperItemMapping/Remove', { id }).then(res => {
        console.log('删除大标题题目---', res)
        if (res.Success) {
          // const arr1 = this.arr.filter(item => {
          //   console.log(item)
          //   return item.Id !== id
          // })
          // this.arr = arr1
          this.getSubjectInfo()
          this.deleteBigTypeRight = false
          this.$message.success('删除成功')
        } else {
          this.$message.error('删除失败')
        }
      })
    },
    // 删除右边的大题
    removeRightBigTitle (id) {
      debugger
      this.deleteBigTypeRight = true
     
    },
    // 选择是否仅本人查看
    onChange (e) {
      console.log(e.target.value)
      this.IsShare = e.target.value
      if (e.target.value === '共享') {
        this.Area = true
      } else {
        this.Area = false
      }
    },
    // 区校选择
    handleArea (e) {
      this.IsShare = e.target.value
      console.log(e.target.value)
    },
    // 练习预览
    toPracticePreview () {
      const aaa = this.arr.some(item => {
        item.Items.some(ite => {
          return ite.Score === 0
        })
      })
      if (this.arr.length === 0) {
        return this.$message.error('请先设置题目')
      }
      console.log(aaa)
      if (this.paperTotalScore !== 100) {
        // this.$message.success('试卷总分不足100')
        this.fraction = true
      } else if (aaa) {
        this.$message.success('请确认试题的分数全部赋值')
      } else {
        this.$router.push({ path: '/Teacher/My_Task/PracticePreview',
          query: {
            id: this.paperId,
            gradeId: this.grade
          } })
      }
    },
    handleFractionList () {
      this.fraction = false
    },
    // 显示作业协同
    showTaskVisible () {
      this.selectTeacher = false
      this.VisibleTask = true
      console.log(this.visibleTask)
      this.getTaskHelp()
      this.getStarBeaconFriend()
      this.getTeacherList()
    },
    onChangeXin (current) {
      this.current = current
      console.log(this.current)
      this.getTeacherList()
    },
    SelectTheTeacher (id) {
      console.log('老师的id', id)
      // this.handleSelectTeacherDetermine.push(id)
    },
    SelectTheTeacherList (id, e) {
      console.log('老师的id', id)

      if (this.handleSelectTeacherDetermine.indexOf(id) === -1) {
        this.handleSelectTeacherDetermine.push(id)
      }
      // this.handleSelectTeacherDetermine.push(id)
      if (e.target.src === require('@/assets/teacher/未勾选.png')) {
        e.target.src = require('@/assets/teacher/勾选.png')
        console.log(e.target.src)
      } else {
        e.target.src = require('@/assets/teacher/未勾选.png')
        const idx = this.handleSelectTeacherDetermine.indexOf(id)
        if (idx !== -1) {
          this.handleSelectTeacherDetermine.splice(idx, 1)
        }
      }
      this.friendId = this.handleSelectTeacherDetermine.join(',')
      console.log('1111111111111111113=----', this.friendId)
    },
    handleSelectTeacher () {
      this.selectTeacher = false
    },
    // 作业协同搜索
    handleSelectTeacherDetermineList () {
      // console.log('sdsadsadsad ', this.handleSelectTeacherDetermine)
      this.friendId = this.handleSelectTeacherDetermine.join(',')
      this.getTaskHelp()
      this.selectTeacher = false
      this.friendId = ''
    },
    // 好友列表s
    getTeacherList () {
      // this.checkIcon = require('@/assets/teacher/未勾选.png')
      this.$http.post('/Friend/FriendRelationShip/GetFriendList', { userId: this.userId, pageNo: this.current }).then(res => {
        console.log('好友列表--', res)
        this.teacherSelectList = res.Data.Items[0].GradeList
        this.teacherName = res.Data.Items[0].SchoolName
        this.total = res.Data.TotalItems
        console.log('页数', this.total)
      })
    },
    // 预览
    handlePreview (i, item1) {
      debugger
      this.Preview = true
      console.log('item', item1)
      this.PreviewTeacher = item1
      this.oneQuote = i.PaperId
      console.log('i', i)
      this.PreviewPaper = i
      this.$http.get('/Paper/Exam_Item/GetAuditPaperItemsList',
        {
          params: {
            paperId: i.PaperId
          }
        }
      ).then(res => {
        console.log(res)
        this.paperData = res.Data
        // 处理填空
        this.paperData.forEach((item, index) => {
          const reg = /(#&\d+@)/g
          const inputele = '<input  type="text" itemId= "' + item.ItemId + '" style="width: 60px;border: 0px; BACKGROUND-COLOR: transparent;text-align:center;border-bottom:1px solid grey;" name="' + 'input' + '"/>'
          const stem = item.Title.replace(reg, inputele)
          item.Title = stem
        })
      })
    },
    // 处理填空
    handleFillBanks (data) {
      data.forEach((item, index) => {
        const reg = /(#&\d+@)/g
        const inputele = ' '
        const stem = item.Title.replace(reg, inputele)
        item.Title = stem
      })
    },
    // 预览收藏
    handleCollection (id, e) {
      debugger
      console.log('收藏ID---', id)
      console.log('---------', e.target)
      console.log(e)
      // e.target.src = require('@/assets/teacher/已收藏.png')
      // let collection = e.target.src
      if (e.target.src === require('@/assets/teacher/收藏.png')) {
        e.target.src = require('@/assets/teacher/已收藏.png')
        this.$http.post('/CollectItem/Teacher_CollectItem/AddCollectItem', { itemid: id, teacherid: this.userId, paperId: this.oneQuote }).then(res => {
          console.log('收藏试题---', res)
          this.$message.success('收藏成功')
        })
      } else {
        e.target.src = require('@/assets/teacher/收藏.png')
        this.$message.success('取消收藏')
      }
    },
    // 返回预览练习
    backPreviewPaper () {
      this.Preview = false
    },
    // 关闭预览
    onPreviewClose () {
      this.Preview = false
    },
    // 获取作业协同试卷数据
    async getTaskHelp () {
      console.log('==============', this.friendId)
      const res = await this.$http.post('/Paper/Exam_Item/GetCollaborationUserList', { pageNo: '1', userId: this.userId, friendId: this.friendId })
      console.log(res)
      this.taskTeacherList = res.Data.Items
      console.log('作业协同教师', this.taskTeacherList)
      const list = this.taskTeacherList.map(item => {
        return item.Id
      })
      const List = list.join(',')
      this.$http.post('/Paper/Exam_Item/GetCollaborationPaperListByUser', { userId: this.userId, userList: List }).then(res => {
        console.log('教师对应的试卷---', res)
        this.taskTeacherListPaper = res.Data
        console.log('教师对应的试卷---', this.taskTeacherListPaper[1927])
      })
    },
    // 关闭作业协同
    onClose () {
      this.VisibleTask = false
    },
    // 获取星标好友
    getStarBeaconFriend () {
      this.$http.post('/Friend/FriendRelationShip/GetFriendRelationshipList', { userId: this.userId }).then(res => {
        console.log('星标好友--', res)
        this.starFriendsList = res.Data
      })
    },
    // 显示我的题库
    showMyTask () {
      this.MyTask = true
      this.getGrade()
      // 获取年级知识点
      this.getGradeKnowledge()
      // 获取题型
      this.$http.post('/Base_Manage/Exam_ItemType/GetDataList', { PageIndex: '1', PageRows: '50' }).then(res => {
        console.log('题型---', res)
        this.subjectList = res.Data
      })
      // 获取题目
      this.getMySubjectBank()
    },
    // 关闭我的题库
    onCloseMyTask () {
      this.MyTask = false
    },

    // 选择班级(我的题库)
    handleClass (value) {
      console.log(value)
    },

    // 本地练习导入
    handleLocalImport () {
      this.localImport = true
    },
    // 上传word文件
    handleChangeUpload (arg1) {
      var thisObj = this
      this.isDisabledUpload = true
      if (arg1.file.status == 'done') {
        if (arg1.file.response.Success) {
          const paperid = arg1.file.response.Data
          this.visible = false
          this.isDisabledUpload = false
          this.localImport = false
          console.log('上传成功')
          this.importPaperVisible = true
          this.getPaperInfo(paperid)
        } else {
          thisObj.$message.error(arg1.file.response.Msg, 2)
          this.isDisabledUpload = false
        }
      }
    },
    // 本地上传试卷
    getPaperInfo (paperid) {
      this.$http.post('/Paper/Exam_PaperItemMapping/GetPaperInfoById', { paperid: paperid }).then(res => {
        console.log('本地上传试卷---', res)
        this.localPaperName = res.Data.PaperName
        this.LocalUploadpaperData = res.Data.QuestionTypes
        this.oneQuote = res.Data.PaperId
        console.log('本地上传试卷---', this.oneQuote)
        // if (res.Success) {
        //   var items = []
        //   for (var i = 0; i < res.Data.QuestionTypes.length; i++) {
        //     const questionTypes = res.Data.QuestionTypes[i]
        //     for (var x = 0; x < questionTypes.Items.length; x++) {
        //       var item = questionTypes.Items[i]
        //       items.push(item)
        //     }
        //   }
        //   this.importPaperItems = items
        // } else {
        //   debugger
        //   this.$message.error(res.Msg, 2)
        // }
      })

      // this.importPaperVisible = true
    },
    // 本地练习一键引用
    OnekeyQuote () {
      console.log('111111111111')
      debugger
      this.$http.post('/Paper/Exam_PaperItemMapping/AddQuoteItemList', { paperId: this.paperId, parentId: this.oneQuoteParentId, quotePaperId: this.oneQuote, typeId: this.oneQuoteTypeId }).then(res => {
        console.log('一键引用---', res)
        if (res.Success) {
          // this.importPaperVisible = false
          this.$message.success('一键引用成功')
          this.getSubjectInfo()
        }
      })
    },
    // 单个题目引用
    singleQuote (itemId, quotePaperId, e,title, show) {
      // debugger
      console.log('单体引用的试卷id---', this.oneQuote)
      // console.log('单题引用的id呃呃呃---', itemId)
      if (show === 1) {
        const info = document.getElementsByClassName('select-title-itemType')
        for (const key of info) {
          console.log('key', key.src)
          key.innerText = '选入'
        }
      } else {
        console.log('单题选入的名称---', e.target.innerText)
      if (e.target.innerText === '选入') {
        e.target.innerText = '移除'
        this.$http.post('/Paper/Exam_PaperItemMapping/AddQuoteItem', { paperId: this.paperId, parentId: this.oneQuoteParentId, quotePaperId: this.oneQuote, quoteItemId: itemId, typeId: this.oneQuoteTypeId }).then(res => {
          console.log('单个题目的引用---', res)
          if (res.Success) {
            // this.importPaperVisible = false
            this.$message.success('成功选入')
            this.getSubjectInfo()
          } else {
            this.$message.error('选入失败')
          }
        })
      } else if (e.target.innerText === '移除') {
        e.target.innerText = '选入'
         this.arr.forEach(item => {
          console.log('第一层的----', item)
            debugger

          item.Items.filter(ite => {
            console.log('第二层的----', ite)
            console.log('第二层的Title----', ite.Title)
            console.log('第二层的title----', title)
            console.log('else if-----TypeId', ite.ItemTypeId)
            if (ite.Title === title) {
              console.log('if-----ite', ite)
              this.re = ite.Id
            } else if (ite.ItemTypeId === 5) {
              console.log('else if-----TypeId')
              var anwers = ''
              ite.Options.forEach(answerItem => {
                if (answerItem.Type === 8) {
                  anwers = answerItem.Content
                }
                console.log('answer', anwers)
              })
              console.log('anwers', ite.Title.replace('（' + anwers + '）', ''))
              console.log('anwers', title.replace('( )', ''))
              debugger
              if (ite.Title.replace('（' + anwers + '）', '') === title.replace('( )', '')) {
                console.log('this.re', ite.Id)
                this.re = ite.Id
              }
            }
          })

          // return item.Name === '单项选择题'
          // console.log(item.Items)
        })
        this.handleDeleteAloneType(this.re)
      }
      }
      
    },
    onImportPaperClose () {
      this.importPaperVisible = false
    },
    // 获取年级知识点
    getGradeKnowledge (id) {
      this.$http.post('/Paper/Exam_SubjectChapter/GetChapterTreeByUserId', { userId: this.userId }).then(res => {
        console.log('年级对应知识点---', res)
        this.optionsTitle = ''
        this.optionsTitle = res.Data[0].Firsts
        console.log(this.options)
      })
    },
    // 获取年级
    getGrade () {
      this.$uwonhttp.get('/Class/TeacherClassManager/GetGradeList').then(res => {
        console.log(res)
        this.gradeInfor = res.data.Data
      })
    },
    // 我的题库选择年级
    handleSelectChangeGrade (value) {
      this.myGrade = value
      console.log('我的题库年级选择', value)
      this.gradeId = value
      if (this.MyGetTitleBank === 1) {
        // 我的题库
        this.getMySubjectBank()
      } else {
        // 我的收藏
        this.getMyCollection()
      }
    },
    // 我的题库选入
    selectMySubject (itemId, typeId, e, title) {
      console.log('我的题库选入---', e.target.innerText)
      debugger
      this.$http.post('/Paper/Exam_PaperItemMapping/AddQuoteForMyItem', { paperId: this.paperId, parentId: this.oneQuoteParentId, quoteItemId: itemId, typeId: this.oneQuoteTypeId }).then(res => {
        console.log('我的题库选入---', res)
        this.getSubjectInfo()
        this.getMySubjectBank()
      })
    },
    deleteMySubject (itemId, typeId, e, title) {
       this.arr.forEach(item => {
          console.log('第一层的----', item)
          item.Items.filter(ite => {
            console.log('第二层的----', ite)
            console.log('第二层的Title----', ite.Title)
            console.log('第二层的title----', title)
            console.log('else if-----TypeId', ite.ItemTypeId)
            if (ite.Title === title) {
              console.log('if-----ite', ite)
              this.re = ite.Id
            } else if (ite.ItemTypeId === 5) {
              console.log('else if-----TypeId')
              var anwers = ''
              ite.Options.forEach(answerItem => {
                if (answerItem.Type === 8) {
                  anwers = answerItem.Content
                }
                console.log('answer', anwers)
              })
              console.log('anwers', ite.Title.replace('（' + anwers + '）', ''))
              console.log('anwers', title.replace('( )', ''))
              if (ite.Title.replace('（' + anwers + '）', '') === title.replace('( )', '')) {
                console.log('this.re', ite.Id)
                this.re = ite.Id
              }
            }
          })

          // return item.Name === '单项选择题'
          // console.log(item.Items)
        })
        console.log('re', this.re)
        // console.log('移除的本张试卷id000', re)
        this.handleDeleteAloneType(this.re)
    },
    // 获取选入题目，需要移除本试卷的单题ID
    getNewAloneId () {

    },
    // 我的题库题型---全部
    allItemType (id) {
      this.ItemTypeTitle = id
      console.log(this.ItemTypeTitle)
      if (this.MyGetTitleBank === 1) {
        this.getMySubjectBank()
      } else {
        this.getMyCollection()
      }
    },
    // 我的题库题型 --- 选择
    SubjectSelect (id) {
      this.ItemTypeTitle = +id
      console.log(this.ItemTypeTitle)
      if (this.MyGetTitleBank === 1) {
        this.getMySubjectBank()
      } else {
        this.getMyCollection()
      }
    },
    // 难度
    Difficulty (id) {
      this.yel = id
      this.levelTitle = id
      console.log('难度颜色---', id)
      if (this.MyGetTitleBank === 1) {
        this.getMySubjectBank()
      } else {
        this.getMyCollection()
      }
    },
    // 知识点选择(我的题库)
    onChangeKnowledge (value) {
      this.chapterTitle = value.join(',')
      console.log('知识点', this.chapterTitle)
      if (this.MyGetTitleBank === 1) {
        this.getMySubjectBank()
      } else {
        this.getMyCollection()
      }
    },
    // 分页
    onChangeMyTitle (pageNumber) {
      // this.singleQuote(0,0,0,1)
      console.log('Page: ', pageNumber)
      this.pageNo = pageNumber
      if (this.MyGetTitleBank === 1) {
        this.getMySubjectBank()
      } else {
        this.getMyCollection()
      }
    },
    // 我的题库
    MySubjectBank () {
      this.mySubBank = true
      this.myCollection = false
      this.MyGetTitleBank = 1
      this.getMySubjectBank()
    },
    // 我的收藏
    MyCollection () {
      this.mySubBank = false
      this.myCollection = true
      this.MyGetTitleBank = 0
      this.ItemTypeTitle = 0
      this.myGrade = 0
      this.getMyCollection()
    },
    // 获取我的题库数据   itemType 题型
    getMySubjectBank () {
      debugger
     
      this.$http.post('/Paper/Exam_Item/GetMyItemList', { userId: this.userId, pageNo: this.pageNo, chapterId: this.chapterTitle, itemType: this.ItemTypeTitle, level: this.levelTitle, gradeId: this.gradeId, newPaperId: this.paperId }).then(res => {
        console.log('我的题库数据---', res)
        this.SubjectBank = res.Data.Items
        // const reg = /(#@)/g
        const reg = /(#&\d+@)/g
        this.SubjectBank.forEach(value => {
          value.Title = value.Title.replace(reg, ' ')
        })
        this.totalMyTitle = res.Data.TotalItems
        console.log('总数--', this.totalMyTitle)
        this.handleFillBanks(res.Data.Items)
        // if (res.Data.Items.length === 0) {
        //   this.noData = true
        // } else {
        //   this.noData = false
        // }
      })
    },
    // 获取我的题库，我的收藏
    getMyCollection () {
      this.$http.post('/CollectItem/Teacher_CollectItem/GetTeacherCollectItems', { page: this.pageNo, pagesize: 5, teacherid: this.userId, grade: this.myGrade, chapterid: this.chapterTitle, itemTypeId: this.ItemTypeTitle, itemLevel: this.levelTitle }).then(res => {
        console.log('我的收藏---', res)
        this.myCollectionList = res.Data
        this.totalMyTitle = res.Data.length
        this.handleFillBanks(res.Data)
      })
    },
    // 移除我的收藏
    deleteMyColletion (id) {
      this.$http.post('/CollectItem/Teacher_CollectItem/RemoveCollectItem', { itemid: id, teacherid: this.userId }).then(res => {
        console.log('移除收藏---', res)
        if (res.Success) {
          this.$message.success('移除收藏成功')
          this.getMyCollection()
        }
      })
    }
  }
}
</script>

<!--样式-->
<style lang="less" scoped>
  .img-div {
    /deep/img {
      width: 200px;
    }
  }
  @colorgrn: #A5D263;
  /deep/.ant-checkbox-group {
    width: 100%;
  }
  * {
    box-sizing: border-box;
  }
  .bgc {
    background-color: @colorgrn;
  }
  .red {
    color: red;
  }
  .m-b {
      margin-bottom: 20px;
  }
  .m-l {
    margin-left: 16px;
  }
  .m-r {
    margin-right: 30px;
  }
  .yellow {
    color: #eac718;
  }
  .btn-col {
    background-color: #68BB97;
  }
  .col-title {
    color: #6BBD9A;
  }
  .f-t {
    font-size: 18px;
  }
  .bold {
    font-weight: 700;
  }
  #subjective {
    width: 49%;
    height: 80px;
    border: 1px solid #ddd;
    border-radius: 5px;

  }
  .title-edit {
    padding: 8px 24px;
    border-radius: 5px;
  }
  // .aaa {
  //   pointer-events: none;
  // }
  // 作业协同缺省页
  .star-beacon {
    text-align: center;
    color: #ccc;
  }
  // 我的题库单题信息

  .title-data {
    margin-top: 28px;
  }
  .select-teacher {
    border: 1px solid #ccc;
    height: 35px;
    line-height: 35px;
    border-radius: 5px;
    text-indent: 15px;
  }
  .blanks-infor {
    display: inline-block;
    text-align: center;
    margin-right: 15px;
    p {
      width: 60px;
      height: 50px;
      line-height: 50px;
      font-size: 16px;
      background-color: #ccc;
      border-radius: 5px;
      border: 1px solid #000;
      color: #fff;
    }
  }
  // t添加同义答案
  .echo-synonymous-answer {
    p:nth-child(1) {
      font-weight: 700;
    }
    .echo-p {
      margin-top: 15px;
      span:nth-child(2) {
        display: inline-block;
        padding: 5px;
        background-color: rgba(30, 118, 233);
        color: #fff;
        border-radius: 5px;
      }
    }
  }
  // 题目数量处理
  .title-num {
    font-size: 16px;
    color: #000;
    font-weight: 700;
  }
  // 制作题目
  /deep/.editor-wrapper {
    height: 77%;
  }
   /deep/.w-e-text-container {
      height: 89% !important;
      border: none;
    }
    /deep/.w-e-text {
      overflow-y: auto;
    }
  .make-topic {
    // display: flex;
    padding: 20px;

    .make-title {
      // width: 49%;
      // height: 615px;
      height: 326px;
      margin-right: 10px;
      margin-bottom: 20px;
      padding: 20px;
      background-color: #fff;
      border-radius: 5px;
      .make-sub-title {
        margin-right: 30px;
      }
    }
    // 选择
    .make-select {
      // width: 49%;
      // min-height: 615px;
      padding: 20px;
      background-color: #fff;
      border-radius: 5px;
      /deep/.w-e-text img {
          height: 30px;
      }
      // 同意答案的增加
      .sy-blocks {
        font-size: 18px;
      }
      // 增加填空
      .fill-blocks {
        font-size: 18px;
      }
      // 添加同意按钮
      .agreen-answer {
        cursor: pointer;
        margin-top: 10px;
        padding: 10px;
        color: #fff;
        background-color: #68BB9A;
        border-radius: 5px;
      }
    }
    // 解析
    .make-analysis {
      width: 100%;
      height: 258px;
      padding: 20px;
      margin-top: 30px;
      background-color: #fff;
      /deep/.ant-editor-wang {
        height: 90%;
      }
    }
  }
  // 题目
  .paper-title {
    padding-left: 20px;
  }
  // 题型边框
  .qus-type {
    border: 1px dotted #D6D6D6;
    img {
      width: 200px;
    }
  }
  // 选中颜色
  .select-color {
    color: #68BB97;
  }
  // 选入题目
  .subject {
    width: 74%;
    // height: 100%;
    margin-right: 5px;
    padding: 0 32px;
    background-color: #fff;

    h3 {
      padding: 64px 0 16px 0;
      text-align: center;
      border-bottom: 1px solid #ccc;
    }
    .tips {
      font-size: 12px;
      text-align: center;
      color: #B5B5B5;
    }
    // 题型操作
    .question-title-type {
      font-size: 18px;
      padding: 8px 24px;
      margin-bottom: 20px;
      border-radius: 5px;
      cursor: pointer;

      .z-span {
        font-size: 14px;
        float: right;
        margin-right: 32px;
      }
      img {
        width: 50%;
        height: 50%;
      }
    }

  }
   // 题目操作按钮
      .edit-subject-operation {
        margin-right: 50px;
        span {
          display: inline-block;
          width: 114px;
          height: 60px;
          line-height: 60px;
          text-align: center;
          margin-left: 48px;
          margin-top: 40px;
          border-radius: 5px;
          cursor: pointer;
        }
        span:nth-child(1) {
          background-color: #fff;
        }
        span:nth-child(2) {
          background-color: #68BB97;
        }
      }
  // 分数选择
    .input-mark {
      width: 96px;
      border: 1px solid #D6D6D6;
      border-radius: 5px;
    }
  // 作业协同
  .task-help {
    div {
      padding: 16px 0 16px 16px;
    }
    div:nth-child(2) {
      // height: 90px;
      // margin-top: 63px;

      border-radius: 5px;
      // background-color: #fff;
    }
    .teacher-p {
       padding: 16px 0;
      text-indent: 30px;
      background-color: #fff;
      /deep/.ant-avatar > img {
        width: 64px;
        height: 64px;
      }
    }
    p {
      span:nth-child(1) {
        margin-right: 15px;
      }
      span:nth-child(3) {
        display: inline-block;
        height: 72px;
        line-height: 72px;
        margin-left: 70px;
        color: #68BB97;
        margin-right: 26px;
      }

    }
    // 教师对应试卷
    .teacher-paper {
        margin-bottom: 70px;
      li {
        float: left;
        width: 196px;
        height: 240px;
        margin-right: 48px;
        margin: 42px 48px 0 0;
        padding: 16px;
        border-radius: 5px;
        background-color: #fff;
        p {
          margin-bottom: 16px;
          overflow: hidden;
          white-space: nowrap;
          text-overflow: ellipsis;
        }
        p:nth-child(1) {
          margin-bottom: 8px;
          font-size: 12px;
        }
        p:nth-child(2) {
          height: 1px;
          width: 120px;
          border-bottom: 1px dotted #ccc;
        }
        /deep/.anticon-star {
          font-size: 12px;
        }
      }
      li:nth-child(4n) {
        margin-right: 0;
      }
    }
  }
  // 预览作业协同
  .paper-preview {
    .teacher-p {
       padding: 16px 0;
      text-indent: 30px;
      background-color: #fff;
    }
    // 试卷基本信息
    .quote {
      display: flex;
      flex-flow: row nowrap;
      justify-content: space-between;
      margin-bottom: 24px;
    }
    /deep/ .anticon-star {
      color: yellow;
    }
    // 收藏/选入
    .collection-select {
      position: absolute;
      right: 70px;
      img {
        margin-right: 30px;
        cursor: pointer;
      }
      .select-title {
        display: inline-block;
        width: 72px;
        height: 36px;
        line-height: 36px;
        text-align: center;
        border-radius: 20px;
        color: #fff;
        background-color: #6BBB9B;
        cursor: pointer;
      }
    }
  }
  // 题目内容
  .subject-info {
    width: 25%;
    background-color: #fff;
    // 题目操作
    .subject-operation {
      text-align: center;
      // 题型 预览
      .title-type {
        margin-top: 48px;
        text-align: center;
        span {
          display: inline-block;
          width: 114px;
          height: 50px;
          line-height: 50px;
          border-radius: 5px;
          cursor: pointer;
        }
        span:nth-child(1) {
          margin-right: 24px;
          margin-bottom: 10px;
          background-color: @colorgrn;
        }
        span:nth-child(2) {
          background-color: #B8D9D0;
        }
      }
      // 题目来源
      .title-source {
        margin-top: 47px;
        margin-bottom: 31px;
        font-size: 20px;
        text-align: left;
        text-indent: 80px;
        color: #49534E;
      }
    }
    // 题目属性
    .subject-attribute {
      padding-left: 80px;
      margin-top: 55px;
      font-size: 15px;
      p:nth-child(1) {
        font-size: 20px;
        color: #49534E;
      }
    }
    // 题目目录
    .subject-menu {
      margin-top: 47px;
      padding-left: 47px;
      padding-right: 47px;
      .subject-type {
        padding: 24px;
        font-size: 15px;
      }
      .subject-type:hover {
        border: 1px dotted #ddd;
        border-radius: 5px;
      }
      .mark {
        display: inline-block;
        text-align: center;
        margin-right: 32px;
        cursor: move;
        span {
          display: inline-block;
          width: 40px;
          height: 40px;
          line-height: 40px;
          font-size: 16px;
          border-radius: 5px;
          border: 1px solid #ccc;
        }
      }
    }
  }
   // 导入练习题
    .import-practice {
      margin-bottom: 35px;
      color: #fff;
      span:nth-child(2) {
        font-size: 12px;
      }
    }
  // 上传word
  .upload-word {
    background-color: #fff;
    border-radius: 5px;
    padding: 59px 100px;
    img {
      margin-bottom: 20px;
    }
    p:nth-child(2) {
      color: #68BB97;
    }
    p:nth-child(3) {
      margin: 16px;
    }
    p:nth-child(5) {
      color: #B5B5B5;
      font-size: 12px;
      margin-top: 52px;
    }
  }
  // 模态框
  /deep/.ant-modal-title {
    color: #fff;
  }
      /deep/.ant-modal-header {
        text-align: center;
        background-color: #68BB97;
      }
      /deep/.ant-modal-footer button + button {
        background-color: #68BB97;
        border: none;
      }
      // /deep/ .ant-select-selection__rendered {
      //   height: 52px;
      //   line-height: inherit;
      // }
  // 按钮框
  .basic-btn {
    display: inline-block;
    width: 252px;
    height: 52px;
    margin-bottom: 10px;
    line-height: 52px;
    font-size: 16px;
    border-radius: 8px;
    cursor: pointer;
  }
  // 题库信息
  .question-bank {
    padding: 36px 0 32px 56px;
    margin-bottom: 41px;
    border-radius: 5px;
    background-color: #fff;
    p {
      margin-bottom: 32px;
    }
    .span-btn {
      span {
        display: inline-block;
        width: 76px;
        height: 22px;
        line-height: 22px;
        margin: 0 16px;
        text-align: center;
        border-radius: 10px;
        cursor: pointer;
      }
      // span:nth-child(1) {
      //   background-color: #68BB97;
      // }
      // span:nth-child(2) {
      //   background-color: #A2C240;
      // }
      .sele {
        color: #fff;
        background-color: #68BB97;
      }
    }
  }
  // 题型
  .question-type {
    // span:nth-child(1) {
    //   color: #68BB97;
    // }
    span {
      margin-right: 16px;
      cursor: pointer;
    }
  }
  // 难度
  .difficulty {
    display: flex;
     span:nth-child(1) {
      color: #68BB97;
    }
    span {
      margin-right: 30px;
    }
     li {
      font-size: 16px;
      margin-right: 20px;
      cursor: pointer;
    }
  }
  // 题库题目
  .question-bank-info {
    padding: 36px 0 32px 56px;
    border-radius: 5px;
    background-color: #fff;
  }
  // 作业协同搜索
  .select-te-list {
    margin: 0px;
    background-color: #fff;
    margin-bottom: 40px;
    // 搜索按钮确认
      .seacher-confirm {
        display: inline-block;
        width: 80px;
        height: 32px;
        color: #fff;
        // background-color: #ddd;
        text-align: center;
        line-height: 32px;
        border-radius: 5px;
        margin-left: 10px;
        cursor: pointer;
      }
    // 星标好友
   .star-infor {
        img {
        width: 60px;
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
  .f-c {
    font-weight: 700;
    color: #000;
  }
  }

  .importPaperTitle{
    margin:10px;
    font-size:20px;
    font-weight: bold;
    text-align: center;
  }

  .importPaperQuote{
    float:right;
    margin-right: 20px;
    color: #68BB97;
  }
  // 本地练习上传的收藏/选入
  .local-practice-test {
    position: relative;
    padding: 10px 30px;
    border-radius: 5px;
    margin-bottom: 20px;
    .local-practice-img {
      padding: 0 20px;
      cursor: pointer;
      // /deep/img {
      //   width: 50%;
      // }
    }
    .local-collection-select {
      position: absolute;
      right: 47px;
      bottom: 30px;
      span {
        margin-right: 20px;
        cursor: pointer;
      }
      span:nth-child(2) {
        display: inline-block;
        width: 72px;
        height: 36px;
        line-height: 36px;
        text-align: center;
        color: #fff;
        background-color: #68BB97;
        border-radius: 15px;
      }
    }
  }
  // 我的题库
  .my-title-bank {
    background-color: #fff;
    padding: 0 8px;
    padding-bottom: 8px;
  }
  // 我的题库选入
  .subject-tip {
    position: relative;
    padding: 23px 30px;
    // border-radius:5px;
    margin-bottom: 20px;
    padding-bottom: 40px;
    border-bottom: 1px solid #ccc;
    .select-itemType {
      background-color: #ccc;
    }
  }
  .select-title-itemType {
      position: absolute;
      right: 25px;
      width: 72px;
      height: 36px;
      line-height: 36px;
      text-align: center;
      color: #fff;
      background-color: #68BB97;
      border-radius: 18px;
      cursor: pointer;
  }
   /deep/.ant-pagination-item:focus, .ant-pagination-item:hover {
      border-color: #68BB97;
      border-color: #68BB97;
    }
    /deep/.ant-pagination-item-active a {
      color: #000;
      background-color: #68BB97;
    }
    .add-friends {
      display: inline-block;
      width: 130px;
      height: 38px;
      line-height: 38px;
      text-align: center;
      color: #fff;
      background-color: #68BB97;
      border-radius: 30px;
    }
</style>
