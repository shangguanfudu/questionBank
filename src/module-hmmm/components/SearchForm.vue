<template>
  <div>
    <el-row type="flex" justify="end" class="no1"
      ><el-button
        type="success"
        icon="el-icon-edit"
        size="mini"
        @click="$router.push('/questions/new')"
        >试题录入</el-button
      ></el-row
    >
    <el-row>
      <el-form label-width="auto" ref="searchForm">
        <el-col :span="6">
          <el-form-item label="学科">
            <el-select
              v-model="searchForm.subjectID"
              placeholder="请选择"
              @change="changeSubjectSimple"
              clearable
              @clear="clear"
            >
              <el-option
                v-for="item in subjectSimpleList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="二级目录">
            <el-select
              v-model="searchForm.directionsValue"
              placeholder="请选择"
              @change="changeDirections"
            >
              <el-option
                v-for="item in directionsList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="标签">
            <el-select v-model="searchForm.tagsValue" placeholder="请选择">
              <el-option
                v-for="item in tagsList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="关键字">
            <el-input
              v-model="searchForm.keyword"
              placeholder="根据题干搜索"
            ></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="试题类型">
            <el-select
              v-model="searchForm.questionTypeValue"
              placeholder="请选择"
            >
              <el-option
                v-for="item in questionTypeList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="难度">
            <el-select
              v-model="searchForm.difficultyValue"
              placeholder="请选择"
            >
              <el-option
                v-for="item in difficultyList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="方向">
            <el-select v-model="searchForm.directionList" placeholder="请选择">
              <el-option
                v-for="item in directionList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="录入人">
            <el-select v-model="searchForm.memberValue" placeholder="请选择">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="题目备注">
            <el-input v-model="searchForm.questionNoteValue"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="企业简称">
            <el-input v-model="searchForm.companyValue"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="城市">
            <el-col :span="11">
              <el-select
                v-model="cityValue1"
                placeholder="请选择"
                @change="changeCity"
              >
                <el-option
                  v-for="item in cityList1"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                >
                </el-option>
              </el-select>
            </el-col>
            <el-col :span="11">
              <el-select v-model="searchForm.cityValue" placeholder="请选择">
                <el-option
                  v-for="item in cityList2"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                >
                </el-option>
              </el-select>
            </el-col>
          </el-form-item>
        </el-col>
        <el-col :span="6" style="display: flex; justify-content: end">
          <el-form-item>
            <el-button size="mini" @click="reset">清除</el-button>
            <el-button size="mini" type="primary" @click="onSubmit"
              >搜索</el-button
            >
          </el-form-item>
        </el-col>
      </el-form>
    </el-row>
  </div>
</template>

<script>
import { simple as subjectSimple } from '@/api/hmmm/subjects'
import { simple as directionsSimple } from '@/api/hmmm/directorys'
import { simple as tagsSimple } from '@/api/hmmm/tags'
import { difficulty, direction, questionType } from '@/api/hmmm/constants'
import { citys, provinces } from '@/api/hmmm/citys'
export default {
  created () { this.getSimpleList() },
  data () {
    return {
      options: [],
      searchForm: {
        subjectID: null,
        directionsValue: null,
        questionTypeValue: null,
        keyword: null,
        tagsValue: null,
        difficultyValue: null,
        directionValue: null,
        memberValue: null,
        questionNoteValue: null,
        companyValue: null,
        cityValue: null
      },
      cityValue1: '',
      // 学科
      subjectSimpleList: [],
      // 二级目录
      directionsList: [],
      // tags
      tagsList: [],
      // 试题类型
      questionTypeList: questionType,
      // 难度类型
      difficultyList: difficulty,
      // 二级城市列表
      cityList2: []
    }
  },
  methods: {
    async getSimpleList () {
      const res = await subjectSimple()
      this.subjectSimpleList = res.data
    },
    async onSubmit () {
      console.log('search', this.searchForm)
      this.$emit('onSubmit', this.searchForm)
    },
    // 科目清空
    async clear () {
      this.searchForm.subjectID = null
      this.$emit('onSubmit', this.searchForm)
    },
    // 清除
    reset () {
      this.searchForm = {
        subjectID: null,
        directionsValue: null,
        questionTypeValue: null,
        keyword: null,
        tagsValue: null,
        difficultyValue: null,
        directionValue: null,
        memberValue: null,
        questionNoteValue: null,
        companyValue: null,
        cityValue: null

      }
      this.$emit('onSubmit', this.searchForm)
    },
    // 简单学科选择
    async changeSubjectSimple (value) {
      this.searchForm.directionsValue = null
      this.searchForm.tagsValue = null
      const res = await directionsSimple({ subjectID: value })
      this.directionsList = res.data
    },
    // 二级目录选择
    async changeDirections (value) {
      const res = await tagsSimple({ subjectID: value })
      this.tagsList = res.data
    },
    // 城市二级目录选择
    changeCity (value) {
      const res = citys(value)
      this.cityList2 = this.traverse(res)
    },
    // 遍历成对象数组
    traverse (arr) {
      var list = []
      arr.forEach(key => {
        const item = {
          value: key,
          label: key
        }
        list.push(item)
      })
      return list
    }
  },
  computed: {
    directionList () {
      return this.traverse(direction)
    },
    cityList1 () {
      const citys = provinces()
      return this.traverse(citys)
    }
  },
  watch: {},
  filters: {},
  components: {}
}
</script>

<style scoped lang='scss'>
.no1 {
  margin-bottom: 15px;
}
.el-col {
  padding: 3px;
}
</style>
