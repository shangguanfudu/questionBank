<template>
  <el-card>
    <!-- 头部区域 -->
    <div class="header">
      <div class="left">
        <div class="search">
          <span class="search-title">关键字</span>
          <el-input v-model="nameValue" style="width: 200px"></el-input>
        </div>
        <div class="search">
          <span class="search-title">状态</span>
          <el-select v-model="stateValue" placeholder="请选择">
            <el-option label="启用" :value="1"> </el-option>
            <el-option label="禁用" :value="0"> </el-option>
          </el-select>
        </div>
        <div class="btns">
          <el-button @click="clearInput">清除</el-button>
          <el-button type="primary" @click="searchSome">搜索</el-button>
        </div>
      </div>
      <el-button class="add" type="success" @click="addSome" icon="el-icon-edit"
        >新增技巧</el-button
      >
    </div>
    <!-- 提示区 -->
    <el-alert class="tip" :title="`数据一共${counts}条`" type="info" show-icon>
    </el-alert>
    <!-- 表格区 -->
    <el-table class="table" :data="tableData" border style="width: 100%">
      <el-table-column type="index" label="序号" align="center" width="80">
      </el-table-column>
      <el-table-column prop="title" label="文章标题" align="center">
        <template slot-scope="scope">
          <span class="text">{{ scope.row.title }}</span>
          <i class="el-icon-film" v-if="scope.row.videoURl !== null"> </i>
        </template>
      </el-table-column>
      <el-table-column prop="visits" label="阅读数" align="center">
      </el-table-column>
      <el-table-column prop="username" label="录入人" align="center">
      </el-table-column>
      <el-table-column prop="createTime" label="录入时间" align="center">
        <template slot-scope="scope">
          {{ scope.row.createTime | formatDate }}
        </template>
      </el-table-column>
      <el-table-column label="状态" align="center">
        <template slot-scope="scope">
          {{ scope.row.state === 1 ? "已启用" : "已禁用" }}
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="changeState(scope.row)">
            {{ scope.row.state === 1 ? "禁用" : "启用" }}
          </el-button>
          <el-button
            @click="viewClick(scope.row)"
            type="text"
            size="small"
            :disabled="scope.row.state === 1"
            :class="scope.row.state === 1 ? 'disabled' : ''"
          >
            预览
          </el-button>
          <el-button
            @click="editClick(scope.row)"
            type="text"
            size="small"
            :disabled="scope.row.state === 1"
            :class="scope.row.state === 1 ? 'disabled' : ''"
          >
            修改
          </el-button>
          <el-button
            :class="scope.row.state === 1 ? 'disabled' : ''"
            :disabled="scope.row.state === 1"
            type="text"
            size="small"
            @click="delClick(scope.row.id)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <div class="page">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="page"
        :page-sizes="[10, 15, 20, 25]"
        :page-size="pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="counts"
      >
      </el-pagination>
    </div>
    <!-- 预览弹出层 -->
    <el-dialog
      class="preview"
      title="预览文章"
      :visible.sync="isPreviewShow"
      width="50%"
    >
      <h2>{{ currentValue.title }}</h2>
      <div class="info">
        <span>{{ currentValue.createTime | formatDate }}</span
        ><span>{{ currentValue.username }}</span
        ><span class="el-icon-view"></span
        ><span>{{ currentValue.visits }}</span>
      </div>
      <div class="content" v-html="currentValue.articleBody"></div>
    </el-dialog>
    <!-- 修改文章弹出层组件 -->
    <!-- 新增文章弹出层 -->
    <el-dialog
      :title="this.form.title === '' ? '新增文章' : '修改文章'"
      :visible.sync="isAddShow"
      @close="onClose"
    >
      <el-form ref="form" :model="form" :rules="rules">
        <el-form-item label="文章标题:" prop="title" label-width="86px">
          <el-input v-model="form.title" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label-width="86px" label="文章内容:" prop="content">
          <el-input v-model="form.articleBody" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label-width="86px" label="视频地址:" prop="videoURL">
          <el-input v-model="form.videoURL" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="isAddShow = false">取 消</el-button>
        <el-button type="primary" @click="onConfirm()">确 定</el-button>
      </div>
    </el-dialog>
  </el-card>
</template>

<script>
import { list, changeState, remove, update, add as addSome } from '@/api/hmmm/articles'
export default {
  created () {
    this.getList()
  },
  data () {
    return {
      form: {
        title: '',
        articleBody: '',
        videoURL: 'http://v.itheima.com/zycbrm2.mp4'
      },
      rules: {
        title: [
          { required: true, message: '请输入标题', trigger: 'blur' }
        ],
        articleBody: [
          { required: true, message: '请输入内容', trigger: 'blur' }
        ]
      },
      isEditShow: false,
      isAddShow: false,
      isPreviewShow: false,
      stateValue: null,
      nameValue: null,
      options: [],
      tableData: [],
      counts: 0,
      page: 1,
      pages: 3,
      pagesize: 10,
      currentValue: {}
    }
  },
  methods: {
    async getList () {
      const { data: res } = await list({
        page: this.page,
        pagesize: this.pagesize,
        keyword: this.nameValue,
        state: this.stateValue
      })
      // console.log(res)
      this.pages = res.pages
      this.counts = res.counts
      this.tableData = res.items
    },
    async changeState (row) {
      await changeState({
        id: row.id,
        state: row.state
      })
      row.state = row.state === 1 ? 0 : 1
      this.$message.success('改变状态成功')
    },
    editClick (row) {
      this.isAddShow = true
      this.form = row
    },
    delClick (id) {
      this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        remove({ id })
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        this.getList()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    clearInput () {
      this.nameValue = null
      this.stateValue = null
    },
    async searchSome () {
      this.getList()
      this.nameValue = null
      this.stateValue = null
    },
    addSome () {
      this.isAddShow = true
    },
    onClose () {
      this.$refs.form.resetFields()
      this.form = {
        title: '',
        articleBody: '',
        videoURL: 'http://v.itheima.com/zycbrm2.mp4'
      }
    },
    async onConfirm () {
      if (!this.form.id) {
        try {
          await addSome({ ...this.form })
          this.getList()
          this.$message.success('添加成功')
          this.isAddShow = false
        } catch (error) {
          this.$message.error('添加失败')
        }
      } else {
        try {
          await update({ ...this.form })
          this.$message.success('修改成功')
          this.getList()
          this.isAddShow = false
        } catch (error) {
          this.$message.error('修改失败')
        }
      }
    },

    handleSizeChange (val) {
      // console.log(`每页 ${val} 条`)
      this.pagesize = val
      this.getList()
    },
    handleCurrentChange (val) {
      // console.log(`当前页: ${val}`)
      this.page = val
      this.getList()
    },
    viewClick (row) {
      this.isPreviewShow = true
      this.currentValue = row
    }
  },
  computed: {
  },
  watch: {},
  filters: {},
  components: {}
}
</script>

<style scoped lang='less'>
.header {
  display: flex;
  justify-content: space-between;
  .left {
    display: flex;
    .search {
      margin-right: 20px;
      .search-title {
        color: #666;
        font-weight: 700;
        margin-right: 15px;
      }
    }
    .btns {
      margin-left: 10px;
    }
  }
  // .add {

  // }
}
.tip {
  margin: 15px 0;
}
.table {
  text-align: center;
  .el-button {
    color: #66b1ff;
    font-size: 16px;
  }
  .disabled {
    color: #999;
  }
  .text {
    font-size: 16px;
    margin-right: 5px;
  }
  .el-icon-film {
    font-size: 16px;
    color: #66b1ff;
    cursor: pointer;
  }
}
.page {
  display: flex;
  justify-content: flex-end;
  margin-top: 15px;
}
.preview {
  .info {
    margin-bottom: 15px;
    margin-left: 10px;
    span {
      margin-right: 5px;
    }
  }
  .content {
    background: #f5f5f5;
    padding: 10px;
  }
}
</style>
