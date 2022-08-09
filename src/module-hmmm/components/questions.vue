<template>
  <div>
    <el-card
      ><search-form @onSubmit="onSubmit"></search-form>
      <question-table
        @chagePages="changePages"
        :tableData="tableData"
        :counts="counts"
      ></question-table
    ></el-card>
  </div>
</template>

<script>
import QuestionTable from './QuestionTable.vue'
import SearchForm from './SearchForm.vue'
import { list } from '@/api/hmmm/questions'
export default {
  created () {
    this.getQuestionList()
  },
  data () {
    return {
      tableData: [],
      counts: 0,
      pageData: {
        page: 1,
        pagesize: 5
      }
    }
  },
  methods: {
    async getQuestionList (data) {
      const res = await list({ ...data, ...this.pageData })
      console.log(res)
      this.tableData = res.data.items
      this.counts = res.data.counts
    },
    async onSubmit ($event) {
      console.log($event)
      const res = await this.getQuestionList($event)
      console.log(res)
    },
    async changePages ($event) {
      this.pageData = $event
      await this.getQuestionList()
    }
  },
  computed: {},
  watch: {},
  filters: {},
  components: { SearchForm, QuestionTable }
}
</script>

<style scoped lang='less'>
.box-card {
  padding: 20px;
  margin: 10px;
}
</style>
