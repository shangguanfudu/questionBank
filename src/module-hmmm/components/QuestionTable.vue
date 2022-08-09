<template>
  <div>
    <el-alert :title="title" type="info" show-icon> </el-alert>
    <el-table :data="tableData" style="width: 100%">
      <el-table-column prop="number" label="试题编号" width="250">
      </el-table-column>
      <el-table-column prop="subject" label="学科" width="100">
      </el-table-column>
      <el-table-column prop="catalog" label="目录" width="100">
      </el-table-column>
      <el-table-column
        prop="questionType"
        label="题型"
        width="100"
        :formatter="formatQuestion"
      >
      </el-table-column>
      <el-table-column
        prop="question"
        label="题干"
        width="250"
        :formatter="formatQuestion1"
      >
      </el-table-column>
      <el-table-column prop="addDate" label="录入时间" width="150">
        <template slot-scope="scope">
          {{ $dayjs(scope.row.addDate).format("YYYY-MM-DD HH:MM:ss") }}
        </template>
      </el-table-column>
      <el-table-column
        prop="difficulty"
        label="难度"
        width="100"
        :formatter="formatQuestion"
      >
      </el-table-column>
      <el-table-column prop="creator" label="录入人" width="120">
      </el-table-column>
      <!-- 基础题库 -->
      <template v-if="$route.path === '/questions/list'">
        <el-table-column label="操作" width="200">
          <template slot-scope="scope">
            <el-row type="flex" justify="space-between">
              <el-button
                circle
                type="primary"
                icon="el-icon-view"
                @click="preview(scope.row.id)"
              ></el-button>
              <el-button
                type="success"
                circle
                icon="el-icon-edit"
                @click="
                  $router.push({
                    name: 'questions-new',
                    query: { id: scope.row.id },
                  })
                "
              ></el-button>
              <el-button
                circle
                type="danger"
                icon="el-icon-delete"
                @click="onDelete(scope.row)"
              ></el-button>
              <el-button
                circle
                type="warning"
                icon="el-icon-check"
                @click="add(scope.row)"
              ></el-button
            ></el-row>
          </template> </el-table-column
      ></template>
      <!-- 精选题库 -->
      <template v-else>
        <el-table-column
          prop="chkState"
          label="审核状态"
          width="120"
          :formatter="formatQuestion"
        >
        </el-table-column>
        <el-table-column prop="chkRemarks" label="审核意见" width="250">
        </el-table-column>
        <el-table-column prop="chkUser" label="审核人" width="120">
        </el-table-column>
        <el-table-column
          prop="publishState"
          label="发布状态"
          width="100"
          :formatter="formatQuestion2"
        >
        </el-table-column>
        <el-table-column fixed="right" label="操作" width="200">
          <template slot-scope="scope">
            <el-row type="flex" justify="space-between">
              <el-link
                type="primary"
                @click="preview(scope.row.id)"
                :underline="false"
                >预览</el-link
              >
              <el-link
                :type="scope.row.chkState === 1 ? 'info' : 'primary'"
                :disabled="scope.row.chkState === 1"
                :underline="false"
                @click="audit(scope)"
                >审核</el-link
              >
              <el-link
                :type="
                  scope.row.chkState === 1 && scope.row.publishState === 1
                    ? 'info'
                    : 'primary'
                "
                :disabled="
                  scope.row.chkState === 1 && scope.row.publishState === 1
                "
                :underline="false"
                @click="
                  $router.push({
                    name: 'questions-new',
                    query: { id: scope.row.id },
                  })
                "
                >修改</el-link
              >
              <el-link
                type="primary"
                :underline="false"
                @click="open(scope.row)"
                >{{
                  scope.row.chkState === 1 && scope.row.publishState === 0
                    ? "上架"
                    : "下架"
                }}</el-link
              >
              <el-link
                @click="onDelete(scope.row)"
                :type="
                  scope.row.chkState === 1 && scope.row.publishState === 1
                    ? 'info'
                    : 'primary'
                "
                :disabled="
                  scope.row.chkState === 1 && scope.row.publishState === 1
                "
                :underline="false"
                >删除</el-link
              ></el-row
            >
          </template>
        </el-table-column>
      </template>
    </el-table>
    <el-row type="flex" justify="end" class="pagination">
      <el-pagination
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="pageData.page"
        :page-sizes="[5, 10, 20, 50]"
        :page-size="pageData.pagesize"
        layout=" prev, pager, next, sizes,jumper"
        :total="counts"
      >
      </el-pagination>
    </el-row>
    <!-- 预览框 -->
    <question-pre-dailog
      v-if="dialogVisible"
      :dialogVisible.sync="dialogVisible"
      :questionID="questionID"
    ></question-pre-dailog>
    <!-- 审核框 -->
    <el-dialog
      title="题目审核"
      :visible.sync="dialogAuditVisible"
      width="30%"
      @close="rest"
    >
      <el-form ref="auditForm">
        <el-form-item v-model="auditForm">
          <el-radio-group v-model="auditForm.chkState">
            <el-radio :label="1">通过</el-radio>
            <el-radio :label="2">拒绝</el-radio>
          </el-radio-group></el-form-item
        >
        <el-form-item>
          <el-input type="textarea" v-model="auditForm.chkRemarks"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="auditSubmit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import * as constants from '@/api/hmmm/constants'
import QuestionPreDailog from './questionPreDailog.vue'
import { choiceCheck, choicePublish, remove, choiceAdd } from '@/api/hmmm/questions'
export default {

  props: {
    tableData: {
      type: Array
    },
    counts: {
      type: Number,
      default: 0
    }
  },
  created () {
  },
  data () {
    return {
      pageData: {
        page: 1,
        pagesize: 5
      },
      dialogVisible: false,
      dialogAuditVisible: false,
      questionID: null,
      auditForm: {
        chkState: 1,
        chkRemarks: null
      }
    }
  },
  methods: {
    handleSizeChange (val) {
      this.pageData.pagesize = val
      this.$emit('chagePages', this.pageData)
      console.log(2, this.pageData)
    },
    handleCurrentChange (val) {
      this.pageData.page = val
      this.$emit('chagePages', this.pageData)
    },
    formatQuestion (row, column, cellValue, index) {
      var back = null
      if (constants[column.property]) {
        back = constants[column.property].find(item => item.value === Number(cellValue)).label
      }
      return back
    },
    formatQuestion1 (row, column, cellValue, index) {
      return cellValue.replace(/<[^>]+>/g, '')
    },
    formatQuestion2 (row, column, cellValue, index) {
      if (cellValue === 0) {
        if (row.chkState === 0) {
          return '待发布'
        } else {
          return '已下架'
        }
      } else {
        return '已发布'
      }
    },
    // 预览
    preview (val) {
      this.questionID = val
      this.dialogVisible = true
    },
    // 审核
    audit (scope) {
      this.auditForm.id = scope.row.id
      this.dialogAuditVisible = true
    },
    // 审核提交
    async auditSubmit (id) {
      const res = await choiceCheck(this.auditForm)
      console.log(res)
      // console.log(this.auditForm)
      this.$emit('chagePages', this.pageData)
      this.dialogAuditVisible = false
    },
    // 列表重置
    rest () {
      this.$refs.auditForm.resetField()
    },
    // 上下架
    open (row) {
      const a = row.chkState === 1 && row.publishState === 0
      const publishState = a ? 1 : 0
      const action = a ? '上架' : '下架'
      this.$confirm('是否' + action + '这个题目？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        await choicePublish({ id: row.id, publishState })
        this.$emit('chagePages', this.pageData)
        this.$message({
          type: 'success',
          message: '操作成功!'
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '操作已取消'
        })
      })
    },
    // 删除
    onDelete (row) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        console.log(row.id)
        await remove({ id: row.id })
        this.$emit('chagePages', this.pageData)
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    add (row) {
      this.$confirm('此操作将该题目加入精选, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        console.log(row.id)
        await choiceAdd({ id: row.id, choiceState: 1 })
        this.$emit('chagePages', this.pageData)
        this.$message({
          type: 'success',
          message: '添加!'
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消添加'
        })
      })
    }
  },
  computed: {
    title () {
      return '数据总共' + this.counts + '条'
    }
  },
  watch: {
    tableData: {
      immediate: true,
      deep: true,
      handler (newVal, oldVal) {
        this.tableData = newVal
      }
    },
    counts: {
      immediate: true,
      deep: true,
      handler (newVal, oldVal) {
        this.counts = newVal
      }
    }
  },
  filters: {},
  components: { QuestionPreDailog }
}
</script>

<style scoped lang='scss'>
.pagination {
  margin-top: 20px;
}
.el-alert {
  margin-bottom: 15px;
}
::v-deep .el-link {
  font-size: 12px;
}
</style>
