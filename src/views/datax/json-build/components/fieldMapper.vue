<template>
  <div class="field-mapper-container">
    <el-row :gutter="20">
      <!-- 源端字段 -->
      <el-col :span="12">
        <el-card class="field-panel" shadow="hover">
          <div slot="header" class="field-panel-header">
            <span class="panel-title">源端字段</span>
            <div class="panel-actions">
              <el-checkbox
                v-model="fromCheckAll"
                :indeterminate="fromIsIndeterminate"
                @change="handleFromCheckAllChange"
              >全选</el-checkbox>
              <span class="selected-count">已选 {{ fromSelected.length }} 个字段</span>
            </div>
          </div>
          <el-table
            ref="fromTable"
            :data="sortedFromColumns"
            row-key="name"
            max-height="500"
            @selection-change="handleFromSelectionChange"
          >
            <el-table-column type="selection" width="50" />
            <el-table-column prop="name" label="字段名称" min-width="120" />
            <el-table-column prop="type" label="字段类型" min-width="100" />
            <el-table-column prop="comment" label="字段注释" min-width="120" show-overflow-tooltip />
          </el-table>
        </el-card>
      </el-col>
      <!-- 目标字段 -->
      <el-col :span="12">
        <el-card class="field-panel" shadow="hover">
          <div slot="header" class="field-panel-header">
            <span class="panel-title">目标字段</span>
            <div class="panel-actions">
              <el-checkbox
                v-model="toCheckAll"
                :indeterminate="toIsIndeterminate"
                @change="handleToCheckAllChange"
              >全选</el-checkbox>
              <span class="selected-count">已选 {{ toSelected.length }} 个字段</span>
            </div>
          </div>
          <el-table
            ref="toTable"
            :data="sortedToColumns"
            row-key="name"
            max-height="500"
            @selection-change="handleToSelectionChange"
          >
            <el-table-column type="selection" width="50" />
            <el-table-column prop="name" label="字段名称" min-width="120" />
            <el-table-column prop="type" label="字段类型" min-width="100" />
            <el-table-column prop="comment" label="字段注释" min-width="120" show-overflow-tooltip />
          </el-table>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  name: 'FieldMapper',
  data() {
    return {
      // 字段详细信息（包含 name、type、comment）
      fromColumnsDetail: [],
      toColumnsDetail: [],
      // 已选中的行数据
      fromSelected: [],
      toSelected: [],
      // 全选状态
      fromCheckAll: false,
      toCheckAll: false
    }
  },
  computed: {
    // 源端字段按名称排序
    sortedFromColumns() {
      return [...this.fromColumnsDetail].sort((a, b) => {
        if (!a.name) return 1
        if (!b.name) return -1
        return a.name.localeCompare(b.name)
      })
    },
    // 目标字段按名称排序
    sortedToColumns() {
      return [...this.toColumnsDetail].sort((a, b) => {
        if (!a.name) return 1
        if (!b.name) return -1
        return a.name.localeCompare(b.name)
      })
    },
    // 源端半选状态
    fromIsIndeterminate() {
      return this.fromSelected.length > 0 && this.fromSelected.length < this.fromColumnsDetail.length
    },
    // 目标端半选状态
    toIsIndeterminate() {
      return this.toSelected.length > 0 && this.toSelected.length < this.toColumnsDetail.length
    }
  },
  methods: {
    // 自动匹配：勾选左右两边名称相同的字段
    autoMatch() {
      this.$nextTick(() => {
        const fromTable = this.$refs.fromTable
        const toTable = this.$refs.toTable
        if (!fromTable || !toTable) return

        // 构建目标字段名称集合
        const toNameSet = new Set(this.toColumnsDetail.map(c => c.name))

        // 源端：勾选与目标端名称匹配的字段
        this.fromColumnsDetail.forEach(row => {
          if (toNameSet.has(row.name)) {
            fromTable.toggleRowSelection(row, true)
          }
        })

        // 构建源端字段名称集合（用于目标端匹配）
        const fromNameSet = new Set(this.fromColumnsDetail.map(c => c.name))

        // 目标端：勾选与源端名称匹配的字段
        this.toColumnsDetail.forEach(row => {
          if (fromNameSet.has(row.name)) {
            toTable.toggleRowSelection(row, true)
          }
        })

        this.updateCheckAllState()
      })
    },
    // 源端全选切换
    handleFromCheckAllChange(val) {
      const fromTable = this.$refs.fromTable
      if (!fromTable) return
      if (val) {
        this.fromColumnsDetail.forEach(row => {
          fromTable.toggleRowSelection(row, true)
        })
      } else {
        fromTable.clearSelection()
      }
    },
    // 目标端全选切换
    handleToCheckAllChange(val) {
      const toTable = this.$refs.toTable
      if (!toTable) return
      if (val) {
        this.toColumnsDetail.forEach(row => {
          toTable.toggleRowSelection(row, true)
        })
      } else {
        toTable.clearSelection()
      }
    },
    // 源端选择变化
    handleFromSelectionChange(selection) {
      this.fromSelected = selection
      this.fromCheckAll = selection.length === this.fromColumnsDetail.length
    },
    // 目标端选择变化
    handleToSelectionChange(selection) {
      this.toSelected = selection
      this.toCheckAll = selection.length === this.toColumnsDetail.length
    },
    // 更新全选状态
    updateCheckAllState() {
      this.fromCheckAll = this.fromSelected.length === this.fromColumnsDetail.length && this.fromColumnsDetail.length > 0
      this.toCheckAll = this.toSelected.length === this.toColumnsDetail.length && this.toColumnsDetail.length > 0
    },
    // 获取源端选中的字段名称列表
    getLColumns() {
      return this.fromSelected.map(row => row.name)
    },
    // 获取目标端选中的字段名称列表
    getRColumns() {
      return this.toSelected.map(row => row.name)
    }
  }
}
</script>

<style lang="scss" scoped>
.field-mapper-container {
  padding: 10px 0;

  .field-panel {
    .field-panel-header {
      display: flex;
      align-items: center;
      justify-content: space-between;

      .panel-title {
        font-weight: bold;
        font-size: 15px;
        color: #303133;
      }

      .panel-actions {
        display: flex;
        align-items: center;
        gap: 12px;

        .selected-count {
          color: #909399;
          font-size: 12px;
        }
      }
    }
  }
}
</style>
