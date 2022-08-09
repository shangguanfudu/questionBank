<template>
  <el-dialog
    width="30%"
    :title="title"
    :visible.sync="dialogVisible"
    @close="$emit('update:dialogVisible', false)"
  >
    <el-form>
      <el-form-item label="所属科目">
        <el-select v-model="value.subjectID">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item :label="valueTitle" label-width="68px">
        <el-input v-model="value.currentName" autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="$emit('update:dialogVisible', false)">取 消</el-button>
      <el-button type="primary" @click="confirmClick">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { simple as getSubList } from '@/api/hmmm/subjects'
export default {
  props: {
    title: {
      type: String,
      required: true
    },
    valueTitle: {
      type: String,
      required: true
    },
    dialogVisible: {
      type: Boolean,
      required: true,
      default: false
    },
    value: {
      type: Object,
      required: true
    }
  },
  async created () {
    try {
      const { data: res } = await getSubList()
      // console.log(res)
      this.options = res
    } catch (error) {
      console.log(error)
    }
  },
  data () {
    return {
      options: []
    }
  },
  methods: {
    confirmClick () {
      this.$emit('confirmClick', this.value)
      this.$emit('update:dialogVisible', false)
    }
  },
  computed: {},
  watch: {},
  filters: {},
  components: {}
}
</script>

<style scoped lang='less'>
</style>
