<template>
  <el-card>
    <!-- 头部区域 -->
    <div class="header">
      <div class="left">
        <div class="search">
          <span class="search-title">标签名称</span>
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
        >新增标签</el-button
      >
    </div>
    <!-- 提示区 -->
    <el-alert class="tip" :title="`数据一共${counts}条`" type="info" show-icon>
    </el-alert>
    <!-- 表格区 -->
    <el-table class="table" :data="tableData" border style="width: 100%">
      <el-table-column type="index" label="序号" align="center" width="80">
      </el-table-column>
      <el-table-column prop="subjectName" label="所属科目" align="center">
      </el-table-column>
      <el-table-column prop="tagName" label="标签名称" align="center">
      </el-table-column>
      <el-table-column prop="username" label="创建者" align="center">
      </el-table-column>
      <el-table-column prop="addDate" label="创建日期" align="center">
        <template slot-scope="scope">
          {{ scope.row.addDate | formatDate }}
        </template>
      </el-table-column>
      <!-- <el-table-column prop="totals" label="面试题数量" align="center">
      </el-table-column> -->
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
    <!-- 修改目录弹出层组件 -->
    <EditDialog
      title="修改标签"
      valueTitle="标签名称"
      :dialogVisible.sync="isEditShow"
      :value="currentValue"
      @confirmClick="editConfirm"
    ></EditDialog>
    <!-- 新增目录弹出层 -->
    <EditDialog
      title="新增标签"
      valueTitle="标签名称"
      :dialogVisible.sync="isAddShow"
      :value="newValue"
      @confirmClick="addConfirm"
    ></EditDialog>
  </el-card>
</template>

<script>
import { list, changeState, remove, update, add as addSome } from '@/api/hmmm/tags'
import EditDialog from '@/components/EditDialog'
export default {
  created () {
    this.getList()
  },
  data () {
    return {
      isEditShow: false,
      isAddShow: false,
      currentValue: {
        currentName: '',
        currentSubject: '',
        id: null,
        subjectID: null
      },
      newValue: {
        currentName: '',
        currentSubject: '',
        id: null,
        subjectID: null
      },
      stateValue: null,
      nameValue: null,
      options: [],
      tableData: [],
      counts: 0,
      page: 1,
      pages: 3,
      pagesize: 10
    }
  },
  methods: {
    async getList () {
      const { data: res } = await list()
      // console.log(res)
      this.counts = res.counts
      this.page = res.page
      this.pagesize = res.pagesize
      this.pages = res.pages
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
      this.currentValue.currentName = row.tagName
      this.currentValue.currentSubject = row.subjectName
      this.currentValue.id = row.id
      this.currentValue.subjectID = row.subjectID
      this.isEditShow = true
    },
    async editConfirm (value) {
      try {
        await update({
          id: this.currentValue.id,
          tagName: value.currentName,
          subjectID: this.currentValue.subjectID
        })
        this.$message.success('修改成功')
        this.getList()
      } catch (error) {
        this.$message.error('修改失败')
      }
    },
    delClick (id) {
      this.$confirm('此操作将永久删除该目录, 是否继续?', '提示', {
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
      const { data: res } = await list({
        tagName: this.nameValue,
        state: this.stateValue
      })
      // console.log(res)
      this.tableData = res.items
      this.nameValue = null
      this.stateValue = null
    },
    addSome () {
      this.isAddShow = true
    },
    async addConfirm () {
      try {
        await addSome({
          tagName: this.newValue.currentName,
          subjectID: this.newValue.subjectID
        })
        this.newValue = {
          currentName: '',
          currentSubject: '',
          id: null,
          subjectID: null
        }
        this.getList()
        this.$message.success('添加成功')
      } catch (error) {
        this.$message.error('添加失败')
      }
    }
  },
  computed: {},
  watch: {},
  filters: {},
  components: { EditDialog }
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
}
</style>
