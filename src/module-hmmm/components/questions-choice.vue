<template>
  <div>
    <el-card class="box-card">
      <search-form @onSubmit="onSubmit"></search-form>
      <!-- 切换栏 -->
      <el-tabs v-model="activeName" @tab-click="handleClick" type="card">
        <el-tab-pane label="全部" name="first">
          <question-table
            @chagePages="changePages"
            :tableData="tableData"
            :counts="counts"
          ></question-table>
        </el-tab-pane>
        <el-tab-pane label="待审核" name="second">
          <question-table
            @chagePages="changePages"
            :tableData="tableData"
            :counts="counts"
          ></question-table>
        </el-tab-pane>
        <el-tab-pane label="已审核" name="third">
          <question-table
            @chagePages="changePages"
            :tableData="tableData"
            :counts="counts"
          ></question-table>
        </el-tab-pane>
        <el-tab-pane label="已拒绝" name="fourth">
          <question-table
            @chagePages="changePages"
            :tableData="tableData"
            :counts="counts"
          ></question-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
  </div>
</template>

<script>
import QuestionTable from './QuestionTable.vue'
import { choice } from '@/api/hmmm/questions.js'
import SearchForm from './SearchForm.vue'

export default {
  created () {
    this.getQuestionList()
  },
  data () {
    return {
      activeName: 'first',
      tableData: [],
      counts: 0,
      pageData: {
        page: 1,
        pagesize: 5
      },
      // tab
      chkState: null,
      searchForm: {}
    }
  },
  methods: {
    async getQuestionList (data) {
      const res = await choice({ ...data, ...this.pageData })
      console.log(res)
      this.tableData = res.data.items
      this.counts = res.data.counts
    },
    async onSubmit ($event) {
      console.log($event)
      const res = await this.getQuestionList($event)
      console.log(res)
    },
    // 切换tab
    async handleClick (tab, event) {
      console.log(tab, event)
      if ((tab.index - 0) === 0) {
        this.chkState = null
      } else {
        this.chkState = tab.index - 1
      }
      await this.getQuestionList({ chkState: this.chkState })
    },
    async changePages ($event) {
      this.pageData = $event
      await this.getQuestionList()
    }
  },
  computed: {
  },
  watch: {

  },
  filters: {},
  components: { QuestionTable, SearchForm }
}
</script>

<style scoped lang='less'>
.box-card {
  padding: 20px;
  margin: 10px;
}
</style>
