<template>
  <div class="box">
    <!-- 头部 -->
    <div class="header">
      <el-row>
        <el-form ref="formRef" :model="form" label-width="80px">
          <el-col :span="6">
            <el-form-item label="标签名称" prop="tagName">
              <el-input placeholder="请输入内容" v-model="form.tagName">
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="城市" prop="city">
              <el-select
                v-model="form.city"
                placeholder="请选择"
                @change="handleProvince"
              >
                <el-option
                  v-for="item in citySelect.city"
                  :key="item"
                  :label="item"
                  :value="item"
                >
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="地区" prop="area">
              <el-select v-model="form.area" placeholder="请选择">
                <el-option
                  v-for="item in citySelect.area"
                  :key="item"
                  :label="item"
                  :value="item"
                >
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="企业简称" prop="abbreviation">
              <el-input placeholder="请输入内容" v-model="form.abbreviation">
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="状态" prop="state">
              <el-select v-model="form.state" placeholder="请选择">
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
          <el-col :span="6" style="margin-left: 14px">
            <el-form-item class="btn">
              <el-button @click="resetForm('formRef')">清除</el-button>
              <el-button type="primary" @click="onSearch">搜索</el-button>
            </el-form-item>
          </el-col>
        </el-form>
        <el-button
          class="add"
          type="success"
          icon="el-icon-edit"
          @click="openAdd('formRef')"
          >新增用户</el-button
        >
      </el-row>
    </div>
    <!-- 警告条 -->
    <el-alert type="info" show-icon :closable="false">
      数据一共 {{ total }} 条
    </el-alert>
    <!-- 表格 -->
    <el-table :data="tableData" style="width: 100%">
      <el-table-column type="index" label="序号"> </el-table-column>
      <el-table-column prop="number" label="企业编号"> </el-table-column>
      <el-table-column prop="shortName" label="企业简称"> </el-table-column>
      <el-table-column prop="tags" label="标签"> </el-table-column>
      <el-table-column prop="creatorID" label="创建者"> </el-table-column>
      <el-table-column prop="addDate" label="创建日期">
        <template slot-scope="scope">
          {{ scope.row.addDate | dateformat }}
        </template>
      </el-table-column>
      <el-table-column prop="remarks" label="备注"> </el-table-column>
      <el-table-column prop="state" label="状态">
        <template slot-scope="scope">
          {{ scope.row.state === 0 ? "禁用" : "启用" }}
        </template>
      </el-table-column>
      <el-table-column prop="" label="操作" width="160px">
        <template slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            circle
            @click="openEdit(scope.row)"
          ></el-button>
          <el-tooltip
            effect="dark"
            :content="scope.row.state === 0 ? '禁用' : '启用'"
            placement="top"
          >
            <el-button
              :type="scope.row.state === 0 ? 'warning' : 'success'"
              :icon="scope.row.state === 0 ? 'el-icon-close' : 'el-icon-check'"
              circle
              @click="setState(scope.row)"
            ></el-button>
          </el-tooltip>
          <el-button
            type="danger"
            icon="el-icon-delete"
            circle
            @click="del(scope.row.id)"
          ></el-button>
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
      width="50%"
      :before-close="handleClose"
    >
      <el-form
        ref="saveFormRef"
        :model="saveForm"
        :rules="saveFormRules"
        label-width="80px"
      >
        <el-form-item label="企业名称" prop="shortName">
          <el-input
            placeholder="请输入内容"
            v-model="saveForm.shortName"
          ></el-input>
        </el-form-item>
        <el-form-item label="所属公司" prop="company">
          <el-input
            placeholder="请输入内容"
            v-model="saveForm.company"
          ></el-input>
          <div>https://www.tianyancha.com （在此可查询所属公司全称及简称）</div>
        </el-form-item>
        <el-form-item label="城市" prop="city">
          <el-select
            v-model="saveForm.city"
            placeholder="请选择"
            @change="handleProvince"
          >
            <el-option
              v-for="item in selectCity.saveCity"
              :key="item"
              :label="item"
              :value="item"
            >
            </el-option>
          </el-select>
          <el-select v-model="saveForm.province" placeholder="请选择">
            <el-option
              v-for="item in selectCity.saveArea"
              :key="item"
              :label="item"
              :value="item"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="方向（企业标签）" prop="tags">
          <el-input placeholder="请输入内容" v-model="saveForm.tags"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="remarks">
          <el-input
            placeholder="请输入内容"
            v-model="saveForm.remarks"
            type="textarea"
          ></el-input>
        </el-form-item>
        <el-form-item label="是否名企">
          <el-switch
            v-model="isFamous"
            active-color="#13ce66"
            inactive-color="#ff4949"
          >
          </el-switch>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button
          @click="
            resetForm('saveFormRef');
            dialogVisible = false;
          "
          >取 消</el-button
        >
        <el-button type="primary" @click="saveAdd">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { list, add, update, remove, disabled } from '@/api/hmmm/companys'
import { provinces, citys } from '@/api/hmmm/citys.js'
export default {
  created () {
    this.getList()
  },
  data () {
    return {
      form: {
        tagName: '',
        city: '',
        area: '',
        abbreviation: '',
        state: null
      },
      citySelect: {
        city: [],
        area: []
      },
      options: [{
        value: 1,
        label: '启用'
      }, {
        value: 0,
        label: '禁用'
      }],
      tableData: [],
      page: 1,
      limit: 10,
      total: null,
      dialogVisible: false,
      saveForm: {
        shortName: '',
        company: '',
        province: '',
        city: '',
        remarks: '',
        tags: ''
      },
      saveFormRules: {
        company: [
          { required: true, message: '所属公司不能为空', trigger: 'blur' }
        ],
        shortName: [
          { required: true, message: '企业简称不能为空', trigger: 'blur' }
        ],
        remarks: [
          { required: true, message: '备注不能为空', trigger: 'blur' }
        ],
        tags: [
          { required: true, message: '请输入标签', trigger: 'blur' }
        ],
        city: [
          { required: true, message: '请选择城市', trigger: 'change' }
        ],
        province: [
          { required: true }
        ]
      },
      selectCity: {
        saveCity: [],
        saveArea: []
      },
      isFamous: false,
      id: null
    }
  },
  methods: {
    // 获取企业列表
    async getList () {
      const res = await list({ page: this.page, pagesize: this.limit, tags: this.form.tagName, province: this.form.area, city: this.form.city, shortName: this.form.abbreviation, state: this.form.state })
      this.tableData = res.data.items
      this.page = Number(res.data.page)
      this.total = res.data.counts
      this.limit = Number(res.data.pagesize)
      this.getProvince()
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
    // 获取城市列表
    getProvince () {
      this.citySelect.city = provinces()
      this.selectCity.saveCity = provinces()
    },
    // 根据选中的城市来获取其下级地区
    handleProvince (e) {
      this.form.area = ''
      this.citySelect.area = citys(e)
      this.form.area = this.citySelect.area[0]
      this.saveForm.province = ''
      this.selectCity.saveArea = citys(e)
      this.saveForm.province = this.selectCity.saveArea[0]
    },
    // 重置
    resetForm (formName) {
      this.$refs[formName].resetFields()
    },
    // 关闭对话框
    handleClose (done) {
      done()
      this.$nextTick(() => {
        this.$refs.saveFormRef.resetFields()
        this.saveForm.province = ''
      })
    },
    // 搜索
    onSearch () {
      this.getList()
    },
    // 打开对话框
    openAdd (formName) {
      this.dialogVisible = true
      this.resetForm(formName)
      this.$nextTick(() => {
        this.saveForm.province = ''
      })
    },
    openEdit (data) {
      this.dialogVisible = true
      this.saveForm.shortName = data.shortName
      this.saveForm.company = data.company
      this.saveForm.province = data.province
      this.saveForm.city = data.city
      this.saveForm.remarks = data.remarks
      this.saveForm.tags = data.tags
      this.isFamous = data.isFamous
      this.id = data.id
    },
    // 新增、修改
    saveAdd () {
      const data = {
        id: this.id,
        isFamous: this.isFamous,
        shortName: this.saveForm.shortName,
        company: this.saveForm.company,
        province: this.saveForm.province,
        city: this.saveForm.city,
        tags: this.saveForm.tags,
        remarks: this.saveForm.remarks
      }
      console.log(this.id)
      this.$refs.saveFormRef.validate(async (valid) => {
        if (valid) {
          if (this.id) {
            await update(data)
            this.$message.success('成功修改用户')
          } else {
            await add(data)
            this.$message.success('成功新增用户')
          }
          this.$refs.saveFormRef.resetFields()
          this.getList()
          this.dialogVisible = false
        } else {
          return false
        }
      })
    },
    // 删除
    async del (id) {
      await remove({ id })
      this.$message.success('成功删除用户')
      this.getList()
    },
    // 设置状态
    setState (data) {
      this.$confirm('此操作将修改此用户的状态, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        data.state = data.state === 0 ? 1 : 0
        await disabled({ id: data.id, state: data.state })
        this.$message.success('修改成功!')
        this.getList()
      }).catch(() => {
        this.$message.info('已取消修改')
      })
    }
  },
  computed: {
    title () {
      return this.id ? '修改用户' : '新增用户'
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
  margin: 20px;
  // 头部
  .header {
    .el-row .el-form {
      :deep(.el-input--suffix .el-input__inner) {
        padding-right: unset;
      }
      .btn {
        :deep(.el-form-item__content) {
          margin-left: 26px !important;
          width: 200px;
        }
      }
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
  // 分页
  .block {
    margin-top: 20px;
    text-align: right;
  }
  // 对话框
  .el-dialog {
    .el-form {
      .el-form-item {
        .el-select {
          width: 120px;
          margin-right: 10px;
        }
      }
    }
  }
}
</style>
