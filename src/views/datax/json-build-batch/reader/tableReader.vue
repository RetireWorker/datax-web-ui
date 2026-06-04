<template>
  <div class="app-container">
    <el-form label-position="right" label-width="120px" :model="readerForm" :rules="rules">
      <el-form-item label="数据库源：" prop="datasourceId">
        <el-select v-model="readerForm.datasourceId" filterable style="width: 300px" @change="rDsChange">
          <el-option
            v-for="item in rDsList"
            :key="item.id"
            :label="item.datasourceName"
            :value="item.id"
          />
        </el-select>
      </el-form-item>
      <el-form-item v-show="dataSource==='postgresql' || dataSource==='oracle' ||dataSource==='sqlserver'" label="Schema：">
        <el-select v-model="readerForm.tableSchema" filterable style="width: 300px" @change="schemaChange">
          <el-option
            v-for="item in schemaList"
            :key="item"
            :label="item"
            :value="item"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="切分字段：">
        <el-input v-model="readerForm.splitPk" placeholder="切分主键" style="width: 13%" />
      </el-form-item>
      <el-form-item label="数据库表名：">
        <el-input v-model="tableFilter" placeholder="输入关键词过滤表名" clearable size="small" style="width: 200px; margin-bottom: 8px;" />
        <el-checkbox
          v-model="readerForm.checkAll"
          :indeterminate="readerForm.isIndeterminate"
          @change="rHandleCheckAllChange"
        >全选
        </el-checkbox>
        <span style="color: #909399; font-size: 12px; margin-left: 8px;">已选 {{ readerForm.tables.length }} 张表</span>
        <div style="margin: 15px 0;" />
        <el-checkbox-group v-model="readerForm.tables" @change="rHandleCheckedChange">
          <el-checkbox v-for="c in rTbList" :key="c" :label="c">{{ c }}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { list as jdbcDsList } from '@/api/datax-jdbcDatasource';
import * as dsQueryApi from '@/api/metadata-query';
import Bus from '../busReader';

export default {
  name: 'TableReader',
  data() {
    return {
      jdbcDsQuery: {
        current: 1,
        size: 200,
        ascs: 'datasource_name'
      },
      rDsList: [],
      rTbList: [],
      schemaList: [],
      loading: false,
      active: 1,
      customFields: '',
      customType: '',
      customValue: '',
      dataSource: '',
      tableFilter: '',
      readerForm: {
        datasourceId: undefined,
        tables: [],
        checkAll: false,
        isIndeterminate: true,
        splitPk: '',
        tableSchema: ''
      },
      rules: {
        datasourceId: [{ required: true, message: 'this is required', trigger: 'change' }],
        tableName: [{ required: true, message: 'this is required', trigger: 'change' }]
      }
    }
  },
  computed: {
    filteredTbList() {
      const keyword = this.tableFilter.trim().toLowerCase()
      if (!keyword) return this.rTbList
      return this.rTbList.filter(t => t.toLowerCase().includes(keyword))
    }
  },
  watch: {
    'readerForm.datasourceId': function(oldVal, newVal) {
      if (this.dataSource === 'postgresql' || this.dataSource === 'oracle' || this.dataSource === 'sqlserver') {
        this.getSchema()
      } else {
        this.getTables('reader')
      }
    }
  },
  created() {
    this.getJdbcDs()
  },
  methods: {
    // 获取可用数据源
    getJdbcDs() {
      this.loading = true
      jdbcDsList(this.jdbcDsQuery).then(response => {
        const { records } = response
        this.rDsList = records
        this.loading = false
      })
    },
    // 获取表名
    getTables(type) {
      if (type === 'reader') {
        let obj = {}
        if (this.dataSource === 'postgresql' || this.dataSource === 'oracle' || this.dataSource === 'sqlserver') {
          obj = {
            datasourceId: this.readerForm.datasourceId,
            tableSchema: this.readerForm.tableSchema
          }
        } else {
          obj = {
            datasourceId: this.readerForm.datasourceId
          }
        }
        // 组装
        dsQueryApi.getTables(obj).then(response => {
          if (response) {
            this.rTbList = response
              .filter(t => !t.toLowerCase().startsWith('sys'))
              .sort((a, b) => a.localeCompare(b))
          }
        })
      }
    },
    getSchema() {
      const obj = {
        datasourceId: this.readerForm.datasourceId
      }
      dsQueryApi.getTableSchema(obj).then(response => {
        this.schemaList = response
      })
    },
    // schema 切换
    schemaChange(e) {
      this.readerForm.tableSchema = e
      // 获取可用表
      this.getTables('reader')
    },
    // reader 数据源切换
    rDsChange(e) {
      // 清空
      this.readerForm.tableName = ''
      this.readerForm.datasourceId = e
      this.rDsList.find((item) => {
        if (item.id === e) {
          this.dataSource = item.datasource
        }
      })
      Bus.dataSourceId = e
      this.$emit('selectDataSource', this.dataSource)
      // 获取可用表
      this.getTables('reader')
    },
    rHandleCheckAllChange(val) {
      this.readerForm.tables = val ? this.rTbList : []
      this.readerForm.isIndeterminate = false
    },
    rHandleCheckedChange(value) {
      const checkedCount = value.length
      this.readerForm.checkAll = checkedCount === this.rTbList.length
      this.readerForm.isIndeterminate = checkedCount > 0 && checkedCount < this.rTbList.length
    },
    getData() {
      if (Bus.dataSourceId) {
        this.readerForm.datasourceId = Bus.dataSourceId
      }
      return this.readerForm
    }
  }
}
</script>
