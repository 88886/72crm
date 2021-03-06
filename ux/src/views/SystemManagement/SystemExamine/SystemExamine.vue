<template>
  <div class="se-container">
    <div class="se-header">审批流程管理</div>
    <div class="se-body">
      <div class="se-table-header">
        <el-button
          class="se-table-header-button"
          type="primary"
          @click="addExamine">添加审批流程</el-button>
      </div>
      <el-table
        v-loading="loading"
        id="examine-table"
        :data="list"
        :height="tableHeight"
        class="main-table"
        highlight-current-row
        style="width: 100%"
        @row-click="handleRowClick">
        <el-table-column
          v-for="(item, index) in fieldList"
          :key="index"
          :formatter="fieldFormatter"
          :prop="item.prop"
          :width="item.width"
          :label="item.label"
          show-overflow-tooltip>
          <template
            slot="header"
            slot-scope="scope">
            <div class="table-head-name">{{ scope.column.label }}</div>
          </template>
        </el-table-column>
        <el-table-column/>
        <el-table-column
          fixed="right"
          label="操作"
          width="250">
          <template slot-scope="scope">
            <el-button
              type="text"
              size="small"
              @click="handleClick('edit', scope)">编辑</el-button>
            <el-button
              type="text"
              size="small"
              @click="handleClick('delete', scope)">删除</el-button>
            <el-button
              type="text"
              size="small"
              @click="handleClick('change', scope)">{{ scope.row['status'] === 0 ? '启用' : '停用' }}</el-button>
            <el-button
              type="text"
              size="small"
              @click="handleClick('copy', scope)">复制并新建</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="p-contianer">
        <el-pagination
          :current-page="currentPage"
          :page-sizes="pageSizes"
          :page-size.sync="pageSize"
          :total="total"
          class="p-bar"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"/>
      </div>
    </div>
    <create-system-examine
      v-if="showHandleView"
      :handle="createHandleInfo"
      @save="getList"
      @hiden-view="showHandleView=false"/>
    <system-examine-detail
      v-if="showDetail"
      :data="detailData"
      @refresh="getList"
      @hide-view="showDetail=false"/>
  </div>
</template>

<script>
import CreateSystemExamine from './components/CreateSystemExamine'
import SystemExamineDetail from './components/systemExamineDetail'
import {
  examineFlowIndex,
  examineFlowDelete,
  examineFlowEnables
} from '@/api/systemManagement/examineflow'

export default {
  /** 系统管理 的 审核管理 */
  name: 'SystemExamine',
  components: {
    CreateSystemExamine,
    SystemExamineDetail
  },
  mixins: [],
  data() {
    return {
      loading: false, // 加载动画
      tableHeight: document.documentElement.clientHeight - 240, // 表的高度
      list: [],
      fieldList: [
        {
          prop: 'name',
          label: '审批流名称',
          width: 150
        },
        {
          prop: 'config',
          label: '流程类型',
          width: 150
        },
        {
          prop: 'types',
          label: '关联对象',
          width: 100
        },
        {
          prop: 'user_ids',
          label: '适用范围',
          width: 150
        },
        {
          prop: 'realname',
          label: '最后修改人',
          width: 150
        },
        {
          prop: 'update_time',
          label: '最后修改时间',
          width: 150
        },
        {
          prop: 'status',
          label: '状态',
          width: 100
        }
      ],
      currentPage: 1,
      pageSize: 10,
      pageSizes: [10, 20, 30, 40],
      total: 0,
      /** 展示操作框 */
      showHandleView: false,
      // 创建的相关信息
      createHandleInfo: { action: 'save', type: 'examineflow', id: '' },
      // 详情展示
      showDetail: false,
      detailData: {}
    }
  },
  computed: {},
  mounted() {
    var self = this
    /** 控制table的高度 */
    window.onresize = function() {
      self.tableHeight = document.documentElement.clientHeight - 240
    }

    this.getList()
  },
  methods: {
    /** 获取列表数据 */
    getList() {
      this.loading = true
      examineFlowIndex({
        page: this.currentPage,
        limit: this.pageSize
      })
        .then(res => {
          this.list = res.data.list

          this.total = res.data.dataCount

          this.loading = false
        })
        .catch(() => {
          this.loading = false
        })
    },
    /** 格式化字段 */
    fieldFormatter(row, column) {
      // 如果需要格式化
      if (column.property === 'config') {
        if (row[column.property] === 1) {
          return '固定审批流'
        } else if (row[column.property] === 0) {
          return '授权审批人'
        } else {
          return ''
        }
      } else if (column.property === 'types') {
        return { crm_contract: '合同', crm_receivables: '回款' }[
          row[column.property]
        ]
      } else if (column.property === 'user_ids') {
        const structures = row['structure_ids_info'] || []
        let strName = structures
          .map(item => {
            return item.name
          })
          .join('、')

        if (strName) {
          strName += '、'
        }

        const users = row['user_ids_info'] || []
        const userName = users
          .map(item => {
            return item.realname
          })
          .join('、')

        const name = strName + userName
        return name || '全公司'
      } else if (column.property === 'status') {
        if (row[column.property] === 0) {
          return '停用'
        }
        return '启用'
      }
      return row[column.property]
    },
    /**
     *  添加审批流
     */
    addExamine() {
      this.createHandleInfo = { action: 'save', type: 'examineflow', id: '' }
      this.showHandleView = true
    },
    /** 列表操作 */
    // 当某一行被点击时会触发该事件
    handleRowClick(row, column, event) {
      if (column.property) {
        this.detailData = row
        this.showDetail = true
      }
    },
    // 更改每页展示数量
    handleSizeChange(val) {
      this.pageSize = val
      this.getList()
    },
    // 更改当前页数
    handleCurrentChange(val) {
      this.currentPage = val
      this.getList()
    },
    handleClick(type, scope) {
      if (type === 'edit') {
        this.createHandleInfo.action = 'update'
        this.createHandleInfo.id = scope.row.flow_id
        this.createHandleInfo.data = scope.row
        this.showHandleView = true
      } else if (type === 'delete') {
        // 启用停用
        this.$confirm('您确定要删除该审批流?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            examineFlowDelete({
              id: scope.row['flow_id']
            })
              .then(res => {
                this.list.splice(scope.$index, 1)
                this.getList()
                this.$message({
                  type: 'success',
                  message: res.data
                })
              })
              .catch(() => {})
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            })
          })
      } else if (type === 'change') {
        // 启用停用
        this.$confirm(
          '您确定要' +
            (scope.row['status'] === 0 ? '启用' : '停用') +
            '该审批流?',
          '提示',
          {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }
        )
          .then(() => {
            examineFlowEnables({
              id: scope.row['flow_id'],
              status: scope.row['status'] === 0 ? 1 : 0
            })
              .then(res => {
                scope.row['status'] = scope.row['status'] === 0 ? 1 : 0
                this.$message({
                  type: 'success',
                  message: res.data
                })
              })
              .catch(() => {})
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            })
          })
      } else if (type === 'copy') {
        this.createHandleInfo.action = 'save'
        this.createHandleInfo.id = ''
        this.createHandleInfo.data = scope.row
        this.showHandleView = true
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.se-header {
  height: 60px;
  position: relative;
  z-index: 100;
  padding: 0 20px;
  font-size: 18px;
  line-height: 60px;
}

.se-body {
  background-color: white;
  border: 1px solid #e6e6e6;
  border-radius: 2px;
}

.se-table-header {
  height: 50px;
  background-color: white;
  position: relative;
  .se-table-header-button {
    float: right;
    margin-right: 20px;
    margin-top: 10px;
  }
}

.p-contianer {
  position: relative;
  background-color: white;
  height: 44px;
  .p-bar {
    float: right;
    margin: 5px 100px 0 0;
    font-size: 14px !important;
  }
}

@import '../styles/table.scss';
</style>
