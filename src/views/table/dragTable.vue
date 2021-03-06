<template>
  <div class="app-container">
    <!-- Note that row-key is necessary to get a correct row order. -->
    <el-table-draggable>
      <el-table v-loading="listLoading" ref="dragTable" :data="list" border fit highlight-current-row style="width: 100%">
        <el-table-column align="center" label="ID" width="65">
          <template slot-scope="scope">
            <span>{{ scope.row.id }}</span>
          </template>
        </el-table-column>

        <el-table-column width="180px" align="center" label="Date">
          <template slot-scope="scope">
            <span>{{ scope.row.timestamp | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
          </template>
        </el-table-column>

        <el-table-column min-width="300px" label="Title">
          <template slot-scope="scope">
            <span>{{ scope.row.title }}</span>
          </template>
        </el-table-column>

        <el-table-column width="110px" align="center" label="Author">
          <template slot-scope="scope">
            <span>{{ scope.row.author }}</span>
          </template>
        </el-table-column>

        <el-table-column width="100px" label="Importance">
          <template slot-scope="scope">
            <svg-icon v-for="n in +scope.row.importance" :key="n" icon-class="star" class="icon-star" />
          </template>
        </el-table-column>

        <el-table-column align="center" label="Readings" width="95">
          <template slot-scope="scope">
            <span>{{ scope.row.pageviews }}</span>
          </template>
        </el-table-column>

        <el-table-column class-name="status-col" label="Status" width="110">
          <template slot-scope="{row}">
            <el-tag :type="row.status | statusFilter">
              {{ row.status }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column align="center" label="Drag" width="80">
          <template slot-scope="{}">
            <svg-icon class="drag-handler" icon-class="drag" />
          </template>
        </el-table-column>
      </el-table>
    </el-table-draggable>
    <div class="show-d">
      <el-tag>The default order :</el-tag> {{ oldList }}
    </div>
    <div class="show-d">
      <el-tag>The after dragging order :</el-tag> {{ newList }}
    </div>
  </div>
</template>

<script>
import { fetchList } from '@/api/article'
import Sortable from 'sortablejs'
// element ui 由2.7.0 升级到时 2.10.1 时dragtable row-key 有问题不能使用
// 安装 npm -i element-ui-el-table-draggable 实现拖拽 但是顺序需要处理 意义不大？
import ElTableDraggable from 'element-ui-el-table-draggable'

export default {
  name: 'DragTable',
  components: {
    ElTableDraggable
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10
      },
      sortable: null,
      oldList: [],
      newList: []
    }
  },
  created() {
    this.getList()
  },
  methods: {
    async getList() {
      this.listLoading = true
      const { data } = await fetchList(this.listQuery)
      console.info(data)
      this.list = data.items
      this.total = data.total
      this.listLoading = false
      this.oldList = this.list.map(v => v.id)
      this.newList = this.oldList.slice()
      this.$nextTick(() => {
        this.setSort()
      })
    },
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll('.el-table__body-wrapper > table > tbody')[0]
      this.sortable = Sortable.create(el, {
        ghostClass: 'sortable-ghost', // Class name for the drop placeholder,
        setData: function(dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData('Text', '')
        },
        onEnd: evt => {
          const targetRow = this.list.splice(evt.oldIndex, 1)[0]
          this.list.splice(evt.newIndex, 0, targetRow)

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0]
          this.newList.splice(evt.newIndex, 0, tempIndex)
        }
      })
    }
  }
}
</script>

<style>
.sortable-ghost {
  opacity: 0.8;
  color: #fff !important;
  background: #42b983 !important;
}
</style>

<style scoped>
.icon-star {
  margin-right: 2px;
}
.drag-handler {
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.show-d {
  margin-top: 15px;
}
</style>
