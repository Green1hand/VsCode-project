<template>
    <el-row :gutter="20">
        <el-col :span="6">
            <category @tree-node-click='treenodeclick'></category>
        </el-col>
        <el-col :span="18">
            <div class="mod-config">
                <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
                <el-form-item>
                    <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button @click="getDataList()">查询</el-button>
                    <el-button type="success" @click="getDataListAll()">查询全部</el-button>
                    <el-button v-if="isAuth('product:attrgroup:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
                    <el-button v-if="isAuth('product:attrgroup:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
                </el-form-item>
                </el-form>
                <el-table
                    :data="dataList"
                    border
                    v-model="dataList"
                    v-loading="dataListLoading"
                    @selection-change="selectionChangeHandle"
                    style="width: 100%;">
                    <el-table-column
                        type="selection"
                        header-align="center"
                        align="center"
                        width="50">
                    </el-table-column>
                    <el-table-column
                        prop="attrId"
                        header-align="center"
                        align="center"
                        label="ID">
                    </el-table-column>
                    <el-table-column
                        prop="attrName"
                        header-align="center"
                        align="center"
                        label="名称">
                    </el-table-column>
                    <el-table-column
                        prop="searchType"
                        header-align="center"
                        align="center"
                        label="可检索">
                        <template slot-scope="scope">
                            <i v-if="scope.row.searchType === 1" class="el-icon-success"></i>
                            <i v-if="scope.row.searchType === 0" class="el-icon-error"></i>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="valueType"
                        header-align="center"
                        align="center"
                        label="值类型">
                        <template slot-scope="scope">
                            <el-tag v-if="scope.row.valueType === 2">类型3</el-tag>
                            <el-tag v-if="scope.row.valueType === 1">类型2</el-tag>
                            <el-tag v-if="scope.row.valueType === 0">类型1</el-tag>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="icon"
                        header-align="center"
                        align="center"
                        label="图标">
                    </el-table-column>
                    <el-table-column
                        prop="valueSelect"
                        header-align="center"
                        align="center"
                        :show-overflow-tooltip="true"
                        label="可选值">
                    </el-table-column>
                    <el-table-column
                        prop="enable"
                        header-align="center"
                        align="center"
                        label="启用">
                        <template slot-scope="scope">
                            <i v-if="scope.row.enable === 1" class="el-icon-success"></i>
                            <i v-if="scope.row.enable === 0" class="el-icon-error"></i>
                        </template>
                    </el-table-column>
                    <el-table-column
                        prop="catelogName"
                        header-align="center"
                        align="center"
                        label="所属分类">
                    </el-table-column>
                    <el-table-column
                        prop="attrgroupName"
                        header-align="center"
                        align="center"
                        label="所属分组">
                    </el-table-column>
                    <el-table-column
                        fixed="right"
                        header-align="center"
                        align="center"
                        width="150"
                        label="操作">
                        <template slot-scope="scope">
                        <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.attrId)">修改</el-button>
                        <el-button type="text" size="small" @click="deleteHandle(scope.row.attrId)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <el-pagination
                    @size-change="sizeChangeHandle"
                    @current-change="currentChangeHandle"
                    :current-page="pageIndex"
                    :page-sizes="[10, 20, 50, 100]"
                    :page-size="pageSize"
                    :total="totalPage"
                    layout="total, sizes, prev, pager, next, jumper">
                </el-pagination>
                <!-- 弹窗, 新增 / 修改 -->
                <AddOrUpdateBaseattr v-if="addOrUpdateVisible" ref="addOrUpdateBaseattr" @refreshDataList="getDataList"></AddOrUpdateBaseattr>
            </div>
        </el-col>
    </el-row>
</template>

<script>
import category from '../common/category'
import AddOrUpdateBaseattr from './baseattr-add-or-update.vue'

export default {
  name: 'VscodeProjectAttrgroup',

  components: {category, AddOrUpdateBaseattr},

  data () {
    return {
      catId: 0,
      dataForm: {
        key: ''
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    }
  },
  activated () {
    this.getDataList()
  },
  methods: {
      // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl(`/product/attr/base/list/${this.catId}`),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'key': this.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          console.log('获取的数据：', data.page)
          this.totalPage = data.page.totalCount
          this.dataList = data.page.list
                    // this.getMenName(data.page.list)
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
        // 查询所有值
    getDataListAll () {
      this.catId = 0
      this.getDataList()
    },
        // 子组件事件
    treenodeclick (data, node, component) {
      console.log('获取子组件点击事件数据：', data, node, component)
      console.log('被点击节点ID：', data.catId)
      if (node.level === 3) {
        this.catId = data.catId
        this.getDataList()
      }
    },
        // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
        // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
        // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
        // 新增 / 修改
    addOrUpdateHandle (id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdateBaseattr.init(id)
      })
    },
        // 删除
    deleteHandle (id) {
      var ids = id ? [id] : this.dataListSelections.map(item => {
        return item.attrId
      })
      this.$confirm('此操作将永久删除这些属性分组！', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/attr/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
