<template>
  <div>
    <el-dialog
      title="题目预览"
      :visible="dialogVisible"
      width="75%"
      :before-close="handleClose"
    >
      <el-form label-width="120px" :model="questionForm">
        <el-row type="flex" justify="start">
          <el-col :span="6">
            <el-form-item label="【题型】:">
              <el-input :value="questionForm.questionType | formatQuestion">
                <template> </template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="【编号】:">
              <el-input :value="questionForm.id"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="【难度】:">
              <el-input
                :value="questionForm.difficulty | formatQuestion"
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="【标签】:">
              <el-input :value="questionForm.remarks"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row type="flex" justify="start">
          <el-col :span="6">
            <el-form-item label="【学科】:">
              <el-input :value="questionForm.subjectName"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="【目录】:">
              <el-input :value="questionForm.directoryName"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="【方向】:">
              <el-input :value="questionForm.direction"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-divider></el-divider>
        <el-form-item label="【题干】:">
          <el-input
            :value="questionForm.question | formatQuestion1"
            class="question"
          ></el-input>
          <p>
            {{ questionForm.questionType | formatQuestion }}
            选项：（以下选中的选项为正确答案）
          </p>
          <template v-if="questionForm.questionType === '1'">
            <el-radio-group v-model="radio">
              <el-radio
                v-for="(item, index) in answer"
                :key="index"
                :label="item.isRight"
                disabled
                >{{ item.code }}:{{ item.title }}</el-radio
              >
            </el-radio-group>
          </template>
          <template v-if="questionForm.questionType === '2'">
            <el-checkbox-group v-model="checkList">
              <el-checkbox
                v-for="(item, index) in answer"
                :key="index"
                :label="item.isRight"
                disabled
                >{{ item.code }}:{{ item.title }}</el-checkbox
              >
            </el-checkbox-group>
          </template>
          <template v-if="questionForm.questionType === '3'">
            <el-radio-group v-model="radio">
              <el-radio
                v-for="(item, index) in answer"
                :key="index"
                :label="item.code"
                >{{ item.code }}</el-radio
              >
            </el-radio-group>
          </template>
        </el-form-item>
        <el-divider></el-divider>
        <el-form-item label="【参考答案】:">
          <el-button type="danger" @click="videoShow = !videoShow" videoShow
            >视频答案播放</el-button
          >
          <div class="video" v-if="videoShow">
            <video width="320" height="240" controls>
              <source
                :src="
                  questionForm.videoUrl ? questionForm.videoUrl : 'movie.mp4'
                "
                type="video/mp4"
              />
            </video>
          </div>
        </el-form-item>
        <el-divider></el-divider>
        <el-form-item label="【答案解析】:">
          <el-input :value="questionForm.answer | formatQuestion1"></el-input>
        </el-form-item>
        <el-divider></el-divider>
        <el-form-item label="【题目备注】:">
          <el-input :value="questionForm.remarks"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="$emit('update:dialogVisible', false)">
          关闭
        </el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import * as constants from '@/api/hmmm/constants'
import { detail } from '@/api/hmmm/questions'
export default {
  props: {
    dialogVisible: {
      type: Boolean,
      required: true
    },
    questionID: {
      type: Number
    }
  },
  created () {
    this.getQuestionDetial()
  },
  data () {
    return {
      videoShow: false,
      questionForm: {},
      radio: 1,
      checkList: [1],
      answer: []
    }
  },
  methods: {
    async getQuestionDetial () {
      const res = await detail({ id: this.questionID })
      this.questionForm = res.data
      this.answer = res.data.options.sort((a, b) => a.code.localeCompare(b.code))
      console.log(res)
    },
    handleClose (done) {
      this.$confirm('确认关闭？')
        .then(_ => {
          this.$emit('update:dialogVisible', false)
          done()
        })
        .catch(_ => { })
    }
  },
  computed: {},
  watch: {},
  filters: {
    formatQuestion (val) {
      var newVal = null
      if (val) {
        newVal = constants.questionType.find(item => item.value === Number(val)).label
      }
      return newVal
    },
    formatQuestion1 (val) {
      var newVal = null
      if (val) {
        newVal = val.replace(/<[^>]+>/g, '')
      }
      return newVal
    }
  },
  components: {}
}
</script>

<style scoped lang='scss'>
::v-deep .el-dialog .el-input__inner {
  border: 0px;
}
.question ::v-deep .el-input__inner {
  color: blue;
}
</style>
