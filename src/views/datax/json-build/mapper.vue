<template>
  <div class="app-container">
    <FieldMapper ref="mapper" />
  </div>
</template>
<script>
import FieldMapper from './components/fieldMapper';
export default {
  name: 'Mapper',
  components: { FieldMapper },
  methods: {
    sendColumnsDetail(fromColumnsDetail, toColumnsDetail, fromColumnList, toColumnList) {
      this.$refs.mapper.fromColumnsDetail = fromColumnsDetail
      this.$refs.mapper.toColumnsDetail = toColumnsDetail
      this.$refs.mapper.fromColumnList = fromColumnList
      this.$refs.mapper.toColumnList = toColumnList
      // 触发自动匹配
      this.$refs.mapper.autoMatch()
    },
    getLColumns() {
      return this.$refs.mapper.getLColumns()
    },
    getRColumns() {
      return this.$refs.mapper.getRColumns()
    }
  }
}
</script>
