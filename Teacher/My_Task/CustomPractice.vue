<template>
  <div>
    <p class="f-s">填写练习/通知属性信息</p>
    <a-form :form="form" :label-col="{ span: 5 }" :wrapper-col="{ span: 8 }" @submit="handleSubmit">
      <a-row :gutter="12">
        <a-col :span="10">
          <a-form-item label="发布班级">
            <a-select
              mode="multiple"
              v-decorator="[
                'class',
                { rules: [{ required: true, message: '必选' }],initialValue: defalutClass },
              ]"
              placeholder="班级选择"
              @change="handleSelectChangeClass"
            >
              <a-select-option v-for="item in classInfor" :key="item.ClassId" :value="item.ClassId">
                {{ item.ClassName }}
              </a-select-option>
            </a-select>
          </a-form-item>
          <a-form-item class="level" label="名称">
            <a-input
              v-model="subName"
              placeholder="请输入练习或通知名称"
            ></a-input>
          </a-form-item>
          <!-- <a-form-item class="level" label="知识点">
            <a-cascader
              :field-names="{ label: 'ChapterName', value: 'ChapterId', children: 'Second' }"
              :options="chapterList"
              change-on-select
              @change="handleChapter"
              @popupVisibleChange="handleChapterPopup"
              placeholder="选择章节"
            />
          </a-form-item> -->
          <a-form-item class="level" label="发布时间">
            <a-radio-group v-model="timeValue" @change="onChange">
              <a-radio :value="1">
                立即发送
              </a-radio>
              <a-radio :value="2">
                自定义时间
              </a-radio>
            </a-radio-group>
          </a-form-item>
          <a-form-item class="level" label="开始时间">
            <span v-if="this.customValue == 1">

              <!-- :disabledDate="disabledDate" -->
              <!-- :default-value="moment(PublishDateTime, 'YYYY-MM-DD HH:mm')" -->
              <a-date-picker
                :showTime="{ format: 'HH:mm' }"
                format="YYYY-MM-DD HH:mm"
                :disabled-time="disabledDateTime"
                :disabledDate="disabledDate"
                :show-time="{ defaultValue: moment('00:00:00', 'HH:mm:ss') }"
                disabled
                v-model="PublishDateTime"
              >

              </a-date-picker></span>
            <span v-else>
              <!-- :default-value="moment(PublishDateTime, 'YYYY-MM-DD HH:mm')" -->
              <a-date-picker
                :showTime="{ format: 'HH:mm' }"
                format="YYYY-MM-DD HH:mm"
                @change="StartTimeChange"
                :disabledDate="disabledDate"
                v-model="PublishDateTime"
              >
              </a-date-picker></span>
          </a-form-item>
          <a-form-item class="level" label="作答截止时间">
            <!-- :default-value="moment(AnswerEndDateTime, 'YYYY-MM-DD HH:mm')" -->
            <a-date-picker
              :showTime="{ format: 'HH:mm' }"
              format="YYYY-MM-DD HH:mm"
              @change="TimeChange"
              :disabledDate="disabledDate1"
              v-model="AnswerEndDateTime"
            >

            </a-date-picker>
          </a-form-item>
        </a-col>
        <a-col :span="10">
          <a-form-item class="level" label="知识点">
            <a-cascader
              :field-names="{ label: 'ChapterName', value: 'ChapterId', children: 'Second' }"
              :options="chapterList"
              change-on-select
              @change="handleChapter"
              @popupVisibleChange="handleChapterPopup"
              placeholder="选择章节"
              v-model="defalutChapterId"
            />
          </a-form-item>
          <!-- <a-form-item label="发布类型">
            <a-select
              v-decorator="[
                'release',
                { rules: [{ required: true, message: '必选' }] },
              ]"
              placeholder="发布类型选择"
              @change="handleSelectChangeRelease"
            >
              <a-select-option value="1">
                作业
              </a-select-option>
              <a-select-option value="0">
                通知
              </a-select-option>
              <a-select-option value="2">
                智能批改
              </a-select-option>
            </a-select>
          </a-form-item> -->
          <a-form-item label="内容">
            <textarea
              v-model="subContent"
              style="height: 200px;width: 400px;border-color: #D6D6D6;border-radius:5px;"
              wrap="physical"
            >
            </textarea>
            <!-- <label for="inputId"><img src="@/assets/teacher/imgload.png" alt=""></label> -->
            <input
              style="display: none"
              id="inputId"
              ref="input"
              type="file"
              @change="uploadImg($event)"
              accept="image/gif, image/jpeg, image/jpg, image/png, image/svg"
            >
          </a-form-item>
          <div class="clearfix">
            <a-form-item label="上传图片">
              <a-upload
                :action="uploadPictureUrl"
                list-type="picture-card"
                :file-list="fileList"
                @preview="handlePreview"
                @change="handleChange"
                :beforeUpload="beforeUpload"
              >
                <div v-if="fileList.length < 5">
                  <a-icon type="plus" />
                  <div class="ant-upload-text">
                    Upload
                  </div>
                </div>
              </a-upload>
              <a-modal :visible="previewVisible" :footer="null" @cancel="handleCancel">
                <img alt="example" style="width: 100%" :src="previewImage" />
              </a-modal>
            </a-form-item></div>
          </a-form-item>

        </a-col>
      </a-row>
    </a-form>
    <p class="handle-btn">
      <span @click="handleBack">返回</span>
      <span @click="handleSubmit">确定</span>
    </p>
    <!-- <img v-for="(item,index) in imgArr" :key="index" :src="item" alt=""> -->
    <footerLogo></footerLogo>
  </div>
</template>

<script>
import footerLogo from '@/components/XuHuiFooter/FooterLogo'
import moment from 'moment'
// function getBase64 (file) {
//   return new Promise((resolve, reject) => {
//     const reader = new FileReader()
//     reader.readAsDataURL(file)
//     reader.onload = () => resolve(reader.result)
//     reader.onerror = error => reject(error)
//   })
// }
export default {
  components: {
    footerLogo
  },
  created () {
    this.userId = localStorage.getItem('UserId')
    this.isAdjust = this.$route.query.isAdjust
    this.paperId = this.$route.query.id
    if (this.isAdjust === '1') {
      this.modifyId = this.$route.query.id
    } else {
      this.modifyId = ''
    }

    console.log('调整跳转进入', this.isAdjust)
    // 获取教师对应班级
    this.getTeacherClass()
    this.handleTime()

    // 回显通知信息
    if (this.isAdjust === '1') {
      this.noticeNews()
    } else {
      // 当天最后时间
      this.getNewDayLastTime()
    }
  },
  mounted () {

  },
  watch: {
    aloneDetails: {
      handler () {

      }
    },
    deep: true
  },
  data () {
    return {
      modifyId: '', // 修改发布id
      paperId: '', // 单题回显详情
      isAdjust: '0', // 是否是调整跳转进入
      aloneDetails: {}, // 回显数据存储
      previewImage: '', // 放大上传图片
      defalutChapterId: [], // 回显的章节
      // 图片预览
      previewVisible: false,
      upClassId: [], // 上传班级
      chapterClassId: '', // 获取章节 班级id
      // 教师Id
      userId: '',
      // 教师对应班级
      classInfor: [],
      form: this.$form.createForm(this),
      defalutClass: '',
      // 章节
      chapterList: [],
      // 时间默认
      timeValue: 1,
      // 作答截止时间
      AnswerEndDateTime: '',
      moment,
      // 切换自定义时间
      customValue: 1,
      // 自定义开始时间
      PublishDateTime: '',
      // 章节id
      ValueChapter: '',
      subName: '',
      subContent: '',
      fileList: [
        // {
        //   uid: '-1',
        //   name: 'image.png',
        //   status: 'done',
        //   thumbUrl: 'https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png'
        // }
        // {
        //   uid: '-1',
        //   name: 'image.png',
        //   status: 'done',
        //   url: 'https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png'
        // }
      ],
      uploadPictureUrl: '',
      imgArr: []
    }
  },
  methods: {
    // 回显通知信息
    noticeNews () {
      // debugger
      this.$uwonhttp.post('/Customer/CustomExercise/GetCustomExerciseDetail', { customExerciseId: this.paperId }).then(res => {
        console.log('单题详情', res)
        this.aloneDetails = res.data.Data
        const arr = res.data.Data.CustomExerciseDetailClass.map(value => {
          return value.ClassId
        })
        this.defalutClass = arr
        this.subName = res.data.Data.PaperName
        this.subContent = res.data.Data.Content
        // 章节
        this.defalutChapterId = [res.data.Data.FirstChapterId, res.data.Data.SecondChapterId]
        // 发布时间
        this.PublishDateTime = res.data.Data.PublishTime
        // 作答截止时间、
        this.AnswerEndDateTime = res.data.Data.EndTime
        this.timeValue = 2
        // 自定义开始时间显示
        this.customValue = 2
        this.imgArr = res.data.Data.PaperImgUrls.split(',')
        console.log('返回图片', res.data.Data.PaperImgUrls.split(','))
        const imgList = []

        this.imgArr.forEach((value, index) => {
          console.log('图片', value)
          const imgObj = {
            uid: index,
            status: 'done',
            thumbUrl: value
            // name: 'image.png'
          }

          imgList.push(imgObj)
          console.log('图片回显', imgList)
        })
        this.fileList = imgList
        if (this.PublishDateTime === moment(new Date()).format('YYYY-MM-DD HH:mm')) {

        }
        console.log('返回发布的时间', this.PublishDateTime)
        console.log('预计发布时间', moment(new Date()).format('YYYY/MM/DD HH:mm:ss'))
        // this.picImgs = res.data.Data.PaperImgUrls.split(',')
      })
    },
    // 获取教师对应班级
    getTeacherClass () {
      this.$uwonhttp.get('/Class/TeacherClassManager/GetTeachClass', { params: { UserId: this.userId } }).then(res => {
        this.classInfor = res.data.Data
        if (this.isAdjust !== '1') {
          this.defalutClass = res.data.Data[0].ClassId
        } else {
          // this.defalutClass = ['1294831679759716352', '1294831679793270784']
        }
        this.upClassId = res.data.Data[0].ClassId
        this.chapterClassId = res.data.Data[0].ClassId
        console.log('老师对应班级---', this.upClassId)
        this.getChapterList()
      })
    },
    // 上传前的函数
    beforeUpload (file) {
      console.log(this.subName)
      console.log('内容', this.subContent)
      console.log('上传前', file)

      if (this.ValueChapter === '' || this.subName === '' || this.subContent === '') {
        // this.$message.success('请填写上述信息,再进行图片上传')
        return false
      }
      return false
      // } else {
      //   this.uploadPictureUrl = `http://uwoomobile.doggod.xyz/Customer/CustomExercise/Submit?ClassIds=${this.defalutClass}&Name=${this.subName}&Content=${this.subContent}&ChapterId=${this.ValueChapter}&CreatorId=${this.userId}&PublishTime=${this.PublishDateTime}&EndTime=${this.AnswerEndDateTime}`
      // }
      // return false
    },
    // 上传图片变化
    handleChange ({ fileList }) {
      debugger
      this.fileList = fileList
      console.log('========', fileList)
    },
    handlePreview (file) {
      console.log(file)
      if (!file.thumbUrl && !file.preview) {
        file.preview = getBase64(file.originFileObj)
      }
      // if (this.isAdjust === '1') {
      //   this.previewImage = file.url
      // }
      this.previewImage = file.thumbUrl || file.preview

      this.previewVisible = true
    },
    handleCancel ({ fileList }) {
      this.previewVisible = false
    },
    uploadImg (e) {
      console.log('上传图片', e.target.files)
      const file = e.target.files[0]
      var reader = new FileReader()
      reader.readAsDataURL(file)
      let url = ''
      const that = this
      reader.onload = function (e) {
        url = this.result.substring(this.result.indexOf(',') + 1)
        // console.log('啊哈哈哈','data:image/png;base64,'+url)
        that.uploadImgInter('data:image/png;base64,' + url)
      }

      // console.log(e.target)
    },
    // 处理当前时间并赋值
    handleTime () {
      const time = new Date()
      const Time = moment(time).format('YYYY-MM-DD HH:mm')
      this.PublishDateTime = Time
      console.log('立即发送时间默认值------' + this.PublishDateTime)
    },
    // 不可选择时间
    disabledDate (current) {
      // 不可选过去时间add(-1, 'd')  subtract(0, 'day')
      return current < moment().add(-1, 'd')
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
    // 获取自定义开始时间
    StartTimeChange (data, dataString) {
      this.PublishDateTime = dataString
      this.comparePublishTime = data._d
      console.log(moment(data._d).valueOf())
      console.log('改变默认值的时间--------' + this.PublishDateTime, data._d)
    },
    // 获取章节id
    getChapterList () {
      this.$uwonhttp.post('/Chapter/TeacherChapter/GetChapterByClass', { ClassId: this.chapterClassId }).then(res => {
        console.log('章节信息', res)
        this.chapterList = res.data.Data
        console.log('章节', this.chapterList)
      })
    },
    // 获取当天时间的 23：59
    getNewDayLastTime () {
      const lastTime = new Date(new Date(new Date().toLocaleDateString()).getTime() + 24 * 60 * 60 * 1000 - 1)
      this.AnswerEndDateTime = moment(lastTime).format('YYYY-MM-DD HH:mm')
      // this.AnswerEndDateTime = '2021/1/20 9:59:00'
      this.compareAnswerEndTime = moment(lastTime)
      console.log(moment(lastTime))

      console.log('获取当天时间的最后时间---', this.AnswerEndDateTime)
    },
    // 获取作答截止时间
    TimeChange (data, dataString) {
      this.AnswerEndDateTime = dataString
      this.compareAnswerEndTime = data._d
      console.log('截止作答时间---------' + this.AnswerEndDateTime, data._d)
    },
    disabledDate1 (current) {
      // 不可选过去时间add(-1, 'd')  subtract(0, 'day')
      return current < moment().subtract(0, 'day')
      // 不可选未来时间
      // return current && current > Date.now()
    },
    // 转换成二进制
    dataURItoBlob (dataURI) {
      var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0] // mime类型
      var byteString = atob(dataURI.split(',')[1]) // base64 解码
      var arrayBuffer = new ArrayBuffer(byteString.length) // 创建缓冲数组
      var intArray = new Uint8Array(arrayBuffer) // 创建视图

      for (var i = 0; i < byteString.length; i++) {
        intArray[i] = byteString.charCodeAt(i)
      }
      return new Blob([intArray], { type: mimeString })
    },
    // 判断是否是base64
    isBase64 (str, list) {
      if (str.indexOf('data:') !== -1 && str.indexOf('base64') !== -1) {
        // debugger
        this.form.validateFields((err, values) => {
          if (!err) {
            console.log('Received values of form: ', values)
            // debugger
            const { fileList } = this
            const formData = new FormData()

            fileList.forEach(file => {
              // debugger

              console.log('循环中', file)
              // debugger
              const bold = this.dataURItoBlob(file.thumbUrl)
              // debugger
              console.log('二进制', bold)
              for (var key in file) {
                console.log('key', key)

                formData.set(key, file[key])
                // console.log('图片数据', formData.get)
              }
              formData.set('file', bold, 'fileName')
              // formData.append('files[]', file)
              // formData.set(, file)
              console.log('图片数据', formData)
            })
            console.log('需要的id', this.$route.query.id)
            formData.set('Id', this.modifyId)
            formData.set('ClassIds', this.upClassId)
            formData.set('PublishTime', this.PublishDateTime)
            formData.set('name', this.subName)
            formData.set('Content', this.subContent)
            if (this.isAdjust === '1') {
              debugger
              const chapId = this.defalutChapterId[1]
              formData.set('ChapterId', chapId)
              // console.log('传入的章节', this.defalutChapterId[1])
            } else {
              formData.set('ChapterId', this.ValueChapter)
            }
            formData.set('EndTime', this.AnswerEndDateTime)
            formData.set('CreatorId', this.userId)

            // const formData = this.fileList[0]
            // { Name: this.subName, Content: this.subContent, ClassIds: this.defalutClass, ChapterId: this.ValueChapter, PublishTime: this.PublishDateTime, EndTime: this.AnswerEndDateTime, CreatorId: this.userId, formData }

            this.$uwonhttp.post('/Customer/CustomExercise/Submit', formData).then(res => {
              console.log('发布练习', res)
              if (!res.data.Success) {
                this.$message.warning('请查看上述内容')
              } else {
                this.$message.success('成功')
                this.$router.push({ path: '/Teacher/IntelligentCorrection/CustomComplete',
                  query: {
                    prcName: this.subName
                  } })
              }
            })
            // this.$http.post('/Notice/NoticeRecord/AddNoticeRecord', { ClassId: values.class, Title: values.name, Content: values.content, TypeId: values.release }).then(res => {
            //   console.log('自定义---', res)
            //   this.$router.push({ path: 'CustomComplete',
            //     query: {
            //       data: JSON.stringify(values),
            //       clre: res.Data
            //     } })
            // })
          }
        })
      } else {
        debugger
        list.map(value => {
          this.$http.post('/Paper/Exam_MicroLesson/Base64Image', { imageUrl: value.thumbUrl }).then(res => {
            debugger
            console.log('', res)
            value.thumbUrl = res.Data
          }).then(() => {
            this.form.validateFields((err, values) => {
              if (!err) {
                console.log('Received values of form: ', values)
                // debugger
                const { fileList } = this
                const formData = new FormData()

                fileList.forEach(file => {
                  // debugger

                  console.log('循环中', file)
                  debugger
                  const bold = this.dataURItoBlob(file.thumbUrl)
                  // debugger
                  console.log('二进制', bold)
                  for (var key in file) {
                    console.log('key', key)

                    formData.set(key, file[key])
                    // console.log('图片数据', formData.get)
                  }
                  formData.set('file', bold, 'fileName')
                  // formData.append('files[]', file)
                  // formData.set(, file)
                  console.log('图片数据', formData)
                })
                console.log('需要的id', this.$route.query.id)
                formData.set('Id', this.modifyId)
                formData.set('ClassIds', this.upClassId)
                formData.set('PublishTime', this.PublishDateTime)
                formData.set('name', this.subName)
                formData.set('Content', this.subContent)
                if (this.isAdjust === '1') {
                  const chapId = this.defalutChapterId[1]
                  formData.set('ChapterId', chapId)
                } else {
                  formData.set('ChapterId', this.ValueChapter)
                }
                formData.set('EndTime', this.AnswerEndDateTime)
                formData.set('CreatorId', this.userId)

                // const formData = this.fileList[0]
                // { Name: this.subName, Content: this.subContent, ClassIds: this.defalutClass, ChapterId: this.ValueChapter, PublishTime: this.PublishDateTime, EndTime: this.AnswerEndDateTime, CreatorId: this.userId, formData }

                this.$uwonhttp.post('/Customer/CustomExercise/Submit', formData).then(res => {
                  console.log('发布练习', res)
                  if (!res.data.Success) {
                    this.$message.warning('请查看上述内容')
                  } else {
                    this.$message.success('成功')
                    this.$router.push({ path: '/Teacher/IntelligentCorrection/CustomComplete',
                      query: {
                        prcName: this.subName
                      } })
                  }
                })

                // this.$http.post('/Notice/NoticeRecord/AddNoticeRecord', { ClassId: values.class, Title: values.name, Content: values.content, TypeId: values.release }).then(res => {
                //   console.log('自定义---', res)
                //   this.$router.push({ path: 'CustomComplete',
                //     query: {
                //       data: JSON.stringify(values),
                //       clre: res.Data
                //     } })
                // })
              }
            })
          })
        })
      }
    },
    // base64Test (list) {
    //   list.map(value => {
    //     this.isBase64(value.thumbUrl, list)
    //   })
    //   console.log('s', list)
    // },
    handleSubmit (e) {
      e.preventDefault()
      // modifyId
      // const that = this
      // console.log('外层图片数据', this.fileList[0].thumbUrl)
      debugger
      console.log('图片', this.fileList)

      // 转换图片形式，完成发布
      this.fileList.forEach(value => {
        this.isBase64(value.thumbUrl, this.fileList)
      })
      // this.base64Test(this.fileList)
      // console.log('dsadasdas ', this.fileList)
    },

    handleSelectChangeClass (value) {
      console.log('班级---', value)
      this.defalutClass = value
      this.chapterClassId = value[value.length - 1]
      this.upClassId = value.join(',')
      console.log('班级字符', this.upClassId)
      this.getChapterList()
    },
    handleSelectChangeRelease (value) {
      console.log('发布类型---', value)
    },
    // 处理章节
    handleChapter (value) {
      this.chapterAll = value
      if (value.length === 1 && value.length !== null) {
        return console.log('hhaha')
      } else {
        this.ValueChapter = value[1]
        console.log('章节选择', value)
      }
    },
    handleChapterPopup (value) {
      // console.log('哈哈哈哈', value)
      if (!value && this.chapterAll.length === 1) {
        this.$message.warning('请选择小章节')
      }
    },
    // 选择发布时间
    onChange (e) {
      console.log(e.target.value)
      this.customValue = e.target.value
      if (e.target.value === 1) {
        const time = new Date()
        const Time = moment(time).format('YYYY-MM-DD HH:mm')
        this.PublishDateTime = Time
      }
    },
    // 返回
    handleBack () {
      this.$router.go(-1)
    }
  }
}
</script>

<style lang="less" scoped>
  .f-s {
    font-size: 20px;
    margin-bottom: 48px;
  }
  // 操作按钮
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
      background-color: #68BB97;
    }
}
/deep/.ant-col-8 {
  width: 44.4%;
}
/deep/.ant-select-selection--single {
    height: 53px;
    line-height: 53px;
    width: 300px;
}
/deep/.ant-select-selection--single .ant-select-selection__rendered {
    height: 53px;
    line-height: 53px;
}
/deep/.ant-select-selection__placeholder {
  margin-top: 0;
  top: 30%;
}
/deep/.ant-input {
  width: 300px;
  height: 53px;
}
</style>
