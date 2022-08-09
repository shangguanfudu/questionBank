<template>
  <div class="box">
    <!-- 头部 -->
    <div class="header">
      <el-row>
        <el-col :span="18">
          <el-form ref="form" :model="form" label-width="80px">
            <el-form-item label="学科名称">
              <el-input
                placeholder="请输入内容"
                v-model="form.value"
                clearable
              ></el-input>
              <el-button type="primary" @click="onSearch">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="6">
          <el-button
            class="add"
            type="success"
            icon="el-icon-edit"
            @click="addSubject"
            >新增学科</el-button
          >
        </el-col>
      </el-row>
    </div>
    <!-- 警告条 -->
    <el-alert type="info" show-icon :closable="false">
      数据一共 {{ total }} 条
    </el-alert>
    <!-- 表格 -->
    <el-table :data="tableData" style="width: 100%">
      <el-table-column type="index" label="序号" width="60px">
      </el-table-column>
      <el-table-column prop="subjectName" label="学科名称"> </el-table-column>
      <el-table-column prop="username" label="创建者"> </el-table-column>
      <el-table-column prop="addDate" label="创建日期" width="160px">
        <template slot-scope="scope">
          {{ scope.row.addDate | dateformat }}
        </template>
      </el-table-column>
      <el-table-column prop="isFrontDisplay" label="前台是否显示">
        <template slot-scope="scope">
          {{ scope.row.isFrontDisplay === 1 ? "是" : "否" }}
        </template>
      </el-table-column>
      <el-table-column prop="twoLevelDirectory" label="二级目录">
      </el-table-column>
      <el-table-column prop="tags" label="标签"> </el-table-column>
      <el-table-column prop="totals" label="题目数量"> </el-table-column>
      <el-table-column prop="address" label="操作" width="260px">
        <template slot-scope="scope">
          <el-link type="primary" @click="$router.push('directorys')"
            >学科分类</el-link
          >
          <el-link type="primary" @click="$router.push('tags')"
            >学科标签</el-link
          >
          <el-link type="primary" @click="dietSubject(scope.row)">修改</el-link>
          <el-link type="primary" @click="delSubject(scope.row.id)"
            >删除</el-link
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <div class="block">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="page"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="limit"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </div>

    <!-- 新增、修改 对话框 -->
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
    >
      <el-form
        ref="saveFormRef"
        :model="saveForm"
        :rules="saveFormRules"
        label-width="80px"
      >
        <el-form-item label="学科名称" prop="saveVal">
          <el-input
            placeholder="请输入内容"
            v-model="saveForm.saveVal"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="是否显示">
          <el-switch
            v-model="isFrontDisplay"
            active-color="#13ce66"
            inactive-color="#ff4949"
          >
          </el-switch>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveSubject">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { list, add, remove, update } from '@/api/hmmm/subjects'
export default {
  created () {
    this.getList()
  },
  data () {
    return {
      form: {
        value: ''
      },
      tableData: [],
      page: 1,
      limit: 10,
      total: null,
      dialogVisible: false,
      saveForm: {
        saveVal: ''
      },
      saveFormRules: {
        saveVal: [
          { required: true, message: '请输入学科名称', trigger: 'blur' }
        ]
      },
      isFrontDisplay: true,
      id: null
    }
  },
  methods: {
    // 获取学科列表
    async getList () {
      const res = await list({ page: this.page, pagesize: this.limit, subjectName: this.form.value })
      this.tableData = res.data.items
      this.page = Number(res.data.page)
      this.total = res.data.counts
      this.limit = Number(res.data.pagesize)
    },
    // 搜索
    onSearch () {
      this.getList()
    },
    // 分页
    handleSizeChange (val) {
      this.limit = val
      this.getList()
    },
    handleCurrentChange (val) {
      this.page = val
      this.getList()
    },
    // 关闭对话框
    handleClose (done) {
      done()
    },
    addSubject () {
      this.dialogVisible = true
      this.saveForm.saveVal = ''
      this.isFrontDisplay = false
      this.id = null
    },
    dietSubject (data) {
      data.isFrontDisplay === 1 ? this.isFrontDisplay = true : this.isFrontDisplay = false
      this.id = data.id
      this.saveForm.saveVal = data.subjectName
      this.dialogVisible = true
    },
    // 新增、修改
    async saveSubject () {
      // 如果有 id 是修改，否则新增
      if (this.id) {
        await update({ subjectName: this.saveForm.saveVal, id: this.id, isFrontDisplay: this.isFrontDisplay })
        this.$message.success('成功修改学科')
      } else {
        await add({ subjectName: this.saveForm.saveVal, isFrontDisplay: this.isFrontDisplay })
        this.$message.success('成功新增学科')
      }
      this.id = null
      this.dialogVisible = false
      this.$refs.saveFormRef.resetFields()
      this.getList()
    },
    // 删除
    delSubject (id) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        await remove({ id })
        this.$message.success('删除成功!')
        this.getList()
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    }
  },
  computed: {
    title () {
      return this.id ? '修改学科' : '新增学科'
    }
  },
  watch: {},
  filters: {},
  components: {}
}
</script>

<style scoped lang='less'>
.box {
  background-color: #fff;
  padding: 20px;
  // 头部
  .header {
    :deep(.el-input--medium) {
      width: 200px;
      margin-right: 10px;
    }
    .add {
      float: right;
    }
  }
  // 警告条
  .el-alert {
    margin-bottom: 20px;
    :deep(.el-alert__icon.is-big) {
      font-size: 16px;
      width: 16px;
    }
  }
  // 文字按钮
  .el-table {
    .el-link {
      margin: 0 5px;
    }
  }
  // 分页
  .block {
    margin-top: 20px;
    text-align: right;
  }
}
</style>
