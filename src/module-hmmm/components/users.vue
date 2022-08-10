<template>
  <el-card class="card">
    <!-- 搜索信息 -->
    <el-row type="flex" justify="space-between" align="middle">
      <el-col>
        <el-form>
          <el-form-item>
            <el-input
              v-model="search"
              style="width: 200px"
              placeholder="根据用户名搜索"
            />
            <el-button
              size="small"
              style="margin-left: 15px"
              @click="clearSearch"
              >清空</el-button
            >
            <el-button @click="searchUsername" type="primary" size="small"
              >搜索</el-button
            >
          </el-form-item>
        </el-form>
      </el-col>
      <el-col>
        <el-row type="flex" justify="end">
          <el-button type="success" size="small" @click="showDialogVisible">
            <i class="el-icon-edit"></i>
            <span>新増用户</span>
          </el-button>
        </el-row>
      </el-col>
    </el-row>
    <!-- 提示信息 -->
    <el-alert type="info" show-icon :closable="false" class="alert">
      <template slot="title">
        <span>共1条记录</span>
      </template>
    </el-alert>
    <!-- 表格信息 -->
    <el-table :data="list">
      <el-table-column label="序号" align="center" type="index" />
      <el-table-column label="邮箱" align="center" prop="email" />
      <el-table-column label="联系电话" prop="phone" />
      <el-table-column label="用户" align="center" prop="username" />
      <el-table-column
        label="权限组名称"
        align="center"
        prop="permission_group_title"
      />
      <el-table-column label="角色" align="center" prop="role" />
      <el-table-column label="操作" fixed="right">
        <template>
          <el-button
            type="primary"
            icon="el-icon-edit"
            plain
            circle
            @click="updateEdit"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页信息 -->
    <el-row type="flex" justify="end">
      <el-pagination
        @current-change="handleCurrentChange"
        :current-page="currentPage4"
        :page-sizes="[10, 20, 30, 50]"
        :page-size="page.size"
        layout="prev, pager, next,sizes,  jumper"
        background
        :total="page.total"
      >
      </el-pagination>
    </el-row>
    <!-- 弹窗模块 -->
    <el-dialog :title="title" :visible.sync="dialogVisible" @close="btnCancel">
      <!-- 表单 -->
      <el-form
        ref="addList"
        label-width="300px"
        :model="formData"
        :rules="rules"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="formData.username" style="width: 50%" />
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="formData.email" style="width: 50%" />
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="formData.password" style="width: 50%" />
        </el-form-item>
        <el-form-item label="角色">
          <el-input v-model="formData.role" style="width: 50%" />
        </el-form-item>
        <el-form-item label="权限组名称">
          <el-select
            v-model="formData.permission_group_id"
            style="width: 50%"
            placeholder="请选择"
          >
            <el-option
              v-for="(item, index) in peoples"
              :key="index"
              :label="item.title"
              :value="item.id"
            />
          </el-select>
          <!-- :value="item.id"是传给服务器的值 -->
        </el-form-item>
        <el-form-item label="联系电话">
          <el-input v-model="formData.phone" style="width: 50%" />
        </el-form-item>
        <el-form-item label="介绍">
          <el-input
            type="textarea"
            :rows="2"
            style="width: 50%"
            placeholder="please input"
            v-model="formData.textarea"
          >
          </el-input>
        </el-form-item>
      </el-form>
      <!-- footer插槽 -->
      <template v-slot:footer>
        <el-row type="flex" justify="center">
          <el-col :span="4">
            <el-button size="medium" @click="dialogVisible = false"
              >取消</el-button
            >
            <el-button type="primary" size="medium" @click="btnOK"
              >确定</el-button
            >
          </el-col>
        </el-row>
      </template>
    </el-dialog>
  </el-card>
  <!--  -->
</template>

<script>
import { list, add, update } from '@/api/base/users'
import { simple } from '@/api/base/permissions'
export default {
  created () {
    this.getUserList()
  },
  data () {
    return {
      list: [], // 接用户数据的
      isEdit: false, // 判断是否是新增还是编辑
      currentPage4: 1,
      search: '', // 搜索
      peoples: [], // 接收获取的权限简单列表的数据
      textarea: '',
      page: {
        page: 1, // 当前页码
        size: 10,
        total: 0 // 总数
      },
      dialogVisible: false, // 关闭弹窗
      formData: {
        username: '', // 用户名
        email: '', // 邮箱
        password: '', // 密码
        role: '', // 角色
        permission_group_id: '', // 权限
        phone: '', // 联系方式
        textarea: ''// 介绍
      },
      rules: {
        username: { required: true, message: '用户名不能为空', trigger: 'blur' },
        email: { required: true, message: '邮箱不能为空', trigger: 'blur' },
        password: { required: true, message: '密码不能为空', trigger: 'blur' }
      }
    }
  },
  methods: {
    handleCurrentChange (val) {
      this.page.page = val
      this.getUserList()
    },
    // 获取表格列表
    async getUserList () {
      try {
        const res = await list(this.page)
        console.log(res.data)
        this.list = res.data.list
      } catch (error) {
        console.log(error)
      }
    },
    // 获取权限组简单列表
    // async getPermissionSimple () {
    //   try {
    //     const res = await simple()
    //     console.log(res.data)
    //     this.peoples = res.data
    // console.log(this.peoples)
    //   } catch (error) {
    //     console.log(error)
    //   }
    // },
    // 获取权限组简单列表
    async showDialogVisible () {
      const res = await simple()
      console.log(res.data)
      this.peoples = res.data
      this.dialogVisible = true
      this.isEdit = false
    },
    // 点击确定时 校验整个表单
    async btnOK () {
      if (this.isEdit) {
        // 编辑用户
        try {
          await update(this.list[0]) // 编辑接口
          this.getUserList()// 重新更新数据到表格
        } catch (error) {
          console.log(error)
        }
      } else {
        // 创建用户
        try {
          await this.$refs.addList.validate()
          // 调用创建用户
          await add(this.formData) // 创建接口
        } catch (error) {
          console.log(error)
        }
        this.dialogVisible = false
      }
    },
    btnCancel () {
      // 重置原来的数据
      this.formData = {
        username: '', // 用户名
        email: '', // 邮箱
        password: '', // 密码
        role: '', // 角色
        permission: '', // 权限
        phone: '', // 联系方式
        textarea: ''// 介绍
      }
      this.$refs.addList.resetFields() // 重置校验结果
      this.dialogVisible = false
    },
    // 编辑用户
    updateEdit () {
      // 重新赋值
      this.formData = this.list[0]
      this.dialogVisible = true
      this.isEdit = true
    },
    // 清空搜索
    async clearSearch () {
      this.search = ''
      await list(this.page)
    },
    // 搜索用户名
    async searchUsername () {
      await list({
        page: 1,
        size: 10,
        username: this.search
      })
    }
  },
  computed: {
    title () {
      return this.isEdit ? '编辑用户' : '创建用户'
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
