<template>
  <el-card class="card">
    <!-- 提示信息 -->
    <el-alert type="info" show-icon :closable="false" class="alert">
      <template slot="title">
        <span>共{{ list.length }}条记录</span>
      </template>
    </el-alert>
    <!-- 表格信息 -->
    <el-table :data="list">
      <el-table-column label="操作类型" align="center" prop="id" />
      <el-table-column label="操作人" align="center" prop="username" />
      <el-table-column label="执行结果" prop="shortName" />
      <el-table-column label="操作时间" align="center" prop="addDate" />
      <el-table-column label="描述" align="center" prop="remarks" />
    </el-table>
    <!-- 分页信息 -->
    <el-row type="flex" justify="end">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage4"
        :page-sizes="[2, 5, 10]"
        :page-size="page.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        background
        :total="10"
      >
      </el-pagination>
    </el-row>
  </el-card>
  <!--  -->
</template>

<script>
import { list } from '@/api/hmmm/companys'
export default {
  created () {
    this.getList()
  },
  data () {
    return {
      list: [], // 接用户数据的
      currentPage4: 1,
      page: {
        page: 1, // 当前页码
        pagesize: 5, //
        total: 0 // 总数
      }
    }
  },
  methods: {
    handleCurrentChange (val) {
      this.page.page = val
      this.getList()
    },
    handleSizeChange (val) {
      this.page.pagesize = val
      this.getList()
    },
    async getList () {
      const res = await list(this.page)
      console.log(res.data.items)
      this.list = res.data.items
      console.log(this.list)
    }
  },
  watch: {},
  filters: {},
  components: {}
}
</script>

<style scoped lang='less'>
.card {
  margin: 20px;
}
.alert {
  margin-bottom: 20px;
}
/deep/.el-table th {
  background-color: #fafafa;
}
.el-pagination {
  margin-top: 20px;
}
</style>
