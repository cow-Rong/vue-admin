<template>
  <div class="table-inline-edit">
    <Hints>
      <template slot="hintName">Table表格组件(行内编辑)</template>
      <template slot="hintInfo">
        <p>element-Table：使用elementUI的Table组件，可用于展示多条结构类似的数据，并对其进行行内编辑操作</p>
        <p>
          地址：访问
          <el-link
            type="success"
            href="https://element.eleme.cn/2.13/TableClassic.vue#/zh-CN/component/table"
            target="_blank"
          >element-Table</el-link>
        </p>
      </template>
    </Hints>
    <!-- <el-card shadow="always"> -->
    <div ref="container" class="container" @scroll="handleScroll">
      <el-table
        v-loading="listLoading"
        :data="showDatas"
        tooltip-effect="dark"
        size="medium"
        :height="totalHeight+100"
        :style="setStyles"
      >
        <el-table-column prop="id" label="编号" align="center" width="120" />
        <el-table-column prop="name" label="姓名" align="center" />
        <el-table-column label="性别" align="center">
          <template slot-scope="{row}">{{ row.sex }}</template>
        </el-table-column>
        <el-table-column label="爱好" align="center" min-width="300px">
          <template slot-scope="{row}">
            <template v-if="row.editable">
              <el-input v-model="row.hobby" style="width: 300px;" />
              <el-button class="cancel-btn" type="warning" @click="cancelEdit(row)">取消</el-button>
            </template>
            <span v-else>{{ row.hobby }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="education" label="学历" align="center" />
        <el-table-column align="center" label="操作" width="200">
          <template slot-scope="{row}">
            <el-button v-if="row.editable" type="success" size="small" @click="confirmEdit(row)">保存</el-button>
            <el-button v-else type="primary" size="small" @click="row.editable = !row.editable">编辑</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- </el-card> -->
    </div>
  </div>
</template>

<script>
import { getTableList } from '@/api'
import Hints from '@/components/Hints'

export default {
  name: 'TableInlineEdit',
  components: { Hints },
  data() {
    return {
      listLoading: true,
      tableList: [],
      listQuery: {
        currentPage: 1,
        pageSize: 10
      },
      // 虚拟滚动需要的参数
      isScrolling: false,
      scrollTop: 0,
      scrollBottom: 160,
      availableHeight: 160,

      flatterData: [],

      rowHeight: 52,
      rowNum: 0,
      totalHeight: 0,

      startIndex: 0,
      endIndex: 0,

      showDatas: []
    }
  },
  computed: {
    setStyles() {
      return {
        '--paddingTop': this.startIndex * this.rowHeight + 'px',
        '--tableHeight': this.totalHeight + 100
      }
    }
  },
  created() {},
  mounted() {
    this.availableHeight = this.$refs.container.offsetHeight
    debugger
    this.scrollTop = this.$refs.container.scrollTop || 0
    this.scrollBottom = this.scrollTop + this.availableHeight
    this.fetchData()
  },
  methods: {
    // 获取数据列表
    fetchData() {
      this.listLoading = true
      // 获取数据列表接口
      getTableList(this.listQuery)
        .then(res => {
          const data = res.data
          if (data.code === 0) {
            const list = data.data.list.map(item => {
              this.$set(item, 'editable', false)
              item.originalHobby = item.hobby
              return item
            })
            this.tableList = Array(30).fill(list[0])
            this.rowNum = this.tableList.length
            this.totalHeight = this.rowHeight * this.rowNum
            this.getshowDatas()
            this.listLoading = false
          }
        })
        .catch(() => {
          this.listLoading = false
        })
    },
    cancelEdit(row) {
      row.hobby = row.originalHobby
      row.editable = false
    },
    confirmEdit(row) {
      row.editable = false
      row.originalHobby = row.hobby
      // 此处添加 后端保存数据接口
    },
    handleScroll(e) {
      if (this.isScrolling) {
        return
      }
      this.isScrolling = true
      requestAnimationFrame(() => {
        this.isScrolling = false
        this.scrollTop = e.target.scrollTop
        this.scrollBottom = this.scrollTop + this.availableHeight
        this.getshowDatas()
      })
    },
    getshowDatas() {
      debugger
      if (this.tableList.length === 0) return

      this.startIndex = Math.max(
        0,
        Math.floor(this.scrollTop / this.rowHeight) - 20
      )
      this.endIndex = Math.min(
        this.rowNum,
        Math.ceil(this.scrollBottom / this.rowHeight) + 20
      )

      const showflatterData = []
      let index = this.startIndex
      while (index < this.endIndex) {
        showflatterData.push(this.tableList[index])
        index++
      }

      setTimeout(() => {
        this.showDatas = showflatterData
      }, 600)
      // this.showDatas = showflatterData;
    }
  }
}
</script>

<style lang="less">
.table-inline-edit {
  height: 100%;
  .el-table thead {
    font-weight: 600;
    th {
      padding: 16px 0 15px !important;
      background-color: #f2f3f7;
    }
  }
}

.container {
  overflow-y: auto;
  height: calc(100% - 96px);
}
.el-table--fit {
  padding-top: var(--paddingTop);
  height: var(--tableHeight);
}
</style>
