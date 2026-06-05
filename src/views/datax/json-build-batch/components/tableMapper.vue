<template>
  <div class="app-container">
    <el-form label-position="left" label-width="80px" :model="readerForm">
      <el-form-item label="源端表">
        <el-checkbox
          v-model="readerForm.lcheckAll"
          :indeterminate="readerForm.isIndeterminate"
          @change="lHandleCheckAllChange"
        >全选</el-checkbox>
        <span style="color: #909399; font-size: 12px; margin-left: 8px;">已选 {{ readerForm.ltables.length }} 张表</span>
        <div style="margin: 15px 0;" />
        <el-checkbox-group v-model="readerForm.ltables" @change="lHandleCheckedChange">
          <el-checkbox v-for="c in sortedFromTablesList" :key="c" :label="c">{{ c }}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="目标表">
        <el-checkbox
          v-model="readerForm.rcheckAll"
          :indeterminate="readerForm.isIndeterminate"
          @change="rHandleCheckAllChange"
        >全选</el-checkbox>
        <span style="color: #909399; font-size: 12px; margin-left: 8px;">已选 {{ readerForm.rtables.length }} 张表</span>
        <div style="margin: 20px 0;" />
        <el-checkbox-group v-model="readerForm.rtables" @change="rHandleCheckedChange">
          <el-checkbox v-for="c in sortedToTablesList" :key="c" :label="c">{{ c }}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: 'TableMapper',
  data() {
    return {
      mapperJson: {},
      fromTablesList: [],
      toTablesList: [],
      readerForm: {
        ltables: [],
        rtables: [],
        lcheckAll: false,
        rcheckAll: false,
        isIndeterminate: true
      }
    }
  },
  computed: {
    sortedFromTablesList() {
      return [...this.fromTablesList].sort((a, b) => a.localeCompare(b))
    },
    sortedToTablesList() {
      return [...this.toTablesList].sort((a, b) => a.localeCompare(b))
    }
  },
  mounted() {
  },
  methods: {
    lHandleCheckAllChange(val) {
      this.readerForm.ltables = val ? this.fromTablesList : []
      this.readerForm.isIndeterminate = false
    },
    rHandleCheckAllChange(val) {
      this.readerForm.rtables = val ? this.toTablesList : []
      this.readerForm.isIndeterminate = false
    },
    lHandleCheckedChange(value) {
      const checkedCount = value.length
      this.readerForm.checkAll = checkedCount === this.fromTablesList.length
      this.readerForm.isIndeterminate = checkedCount > 0 && checkedCount < this.fromTablesList.length
    },
    rHandleCheckedChange(value) {
      const checkedCount = value.length
      this.readerForm.checkAll = checkedCount === this.toTablesList.length
      this.readerForm.isIndeterminate = checkedCount > 0 && checkedCount < this.toTablesList.length
    },
    getLTables() {
      return this.readerForm.ltables
    },
    getRTables() {
      return this.readerForm.rtables
    },
    // 按表名自动将目标表与源表选中项匹配（两端都可能带 Schema 前缀）
    autoMatchTargetTables() {
      if (!this.fromTablesList.length || !this.toTablesList.length) return
      const stripSchema = (t) => {
        const parts = t.split('.')
        const name = parts.length > 1 ? parts[parts.length - 1] : t
        return name.replace(/^"|"$/g, '')
      }
      // 默认全选源端表
      if (!this.readerForm.ltables.length) {
        this.readerForm.ltables = [...this.fromTablesList]
      }
      // 按表名匹配目标端表
      const sourceNames = this.readerForm.ltables.map(stripSchema)
      this.readerForm.rtables = this.toTablesList.filter(t => sourceNames.includes(stripSchema(t)))
    }
  }
}
</script>
