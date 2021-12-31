<template>
  <div>
    <el-switch
      v-model="draggable"
      active-color="#13ce66"
      inactive-color="#ff4949"
      active-text="开启拖拽"
      inactive-text="关闭拖拽">
    </el-switch>
    <el-button type="success" size="mini" icon="el-icon-check" plain
    :disabled="!draggable" 
    @click="bachSave">
      批量保存
    </el-button>
    <el-button type="danger" size="mini" icon="el-icon-close" plain
    :disabled="disabled" 
    @click="bachDelete">
      批量删除
    </el-button>
    <el-tree 
      show-checkbox
      ref="MenusTree"
      node-key="catId"
      :data="menus" 
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedKey" 
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
            <el-button
              v-if="node.level <= 2"
              type="text"
              size="mini"
              @click="() => append(data)">
              Append
            </el-button>
            <el-button
              type="text"
              size="mini"
              @click="() => update(data)">
              Update
            </el-button>
            <el-button
              v-if="node.childNodes.length == 0"
              type="text"
              size="mini"
              @click="() => remove(node, data)">
              Delete
            </el-button>
          </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称:">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标:">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位:">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'VscodeProjectCategory',

  data () {
    return {
      menus: [],
      expandedKey: [],
      title: '',
      dialogType: '',
      dialogVisible: false,
      category: {
        catId: null,
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: ''

      },
      maxLevel: 0,
      updateNodes: [],
      draggable: false,
      pCid: [],
      disabled: true,
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },

  mounted () {
  },

  methods: {
    // 获取数据列表
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        console.log(data.data)
        this.menus = data.data
      })
    },
    // 打开添加弹窗
    append (data) {
      this.dialogType = 'append'
      this.title = '添加'
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
    },
    // 打开修改弹窗
    update (data) {
      this.dialogType = 'update'
      this.title = '修改'
      this.dialogVisible = true
      // 获取最新数据回显
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        console.log('回显的数据：', data)
        this.category.catId = data.data.catId
        this.category.name = data.data.name
        this.category.parentCid = data.data.catId
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
      })
    },
    // 判断提交数据方法
    submitData () {
      if (this.dialogType === 'append') {
        this.addCategory()
      }
      if (this.dialogType === 'update') {
        this.updateCategory()
      }
    },
    // 添加方法
    addCategory () {
      console.log('添加的数据：', this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.dialogVisible = false
        this.$message({
          type: data.msg,
          message: data.msg
        })
        // 刷新菜单
        this.getMenus()
        // 展开列表
        this.expandedKey = [this.category.parentCid]
        // 初始化数据
        this.initialization()
      })
    },
    // 修改方法
    updateCategory () {
      console.log('修改后的数据：', this.category)
      var {catId, name, icon, productUnit} = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, name, icon, productUnit}, false)
      }).then(({ data }) => {
        this.dialogVisible = false
        this.$message({
          type: data.msg,
          message: data.msg
        })
        // 刷新菜单
        this.getMenus()
        // 展开列表
        this.expandedKey = [this.category.parentCid]
        // 初始化数据
        this.initialization()
      })
    },
    // 删除方法
    remove (node, data) {
      console.log('remove', node, data)
      var ids = [data.catId]
      this.$confirm(`此操作将永久删除【${data.name}】分类`, ' 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            type: data.msg,
            message: data.msg
          })
          // 刷新菜单
          this.getMenus()
          // 展开列表
          this.expandedKey = [node.parent.data.catId]
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    // 批量删除方法
    bachDelete () {
      let catIds = []
      let CheckedNodes = this.$refs.MenusTree.getCheckedNodes()
      console.log('被选中的节点数据：', CheckedNodes)
      CheckedNodes.forEach(element => {
        catIds.push(element.catId)
      })
      this.$confirm('此操作将永久删除所选分类', ' 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(catIds, false)
        }).then(({data}) => {
          this.$message({
            type: data.msg,
            message: data.msg
          })
          // 刷新菜单
          this.getMenus()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    // 保存拖拽修改方法
    bachSave () {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({
          type: data.msg,
          message: data.msg
        })
        // 刷新菜单
        this.getMenus()
        // 展开列表
        this.expandedKey = this.pCid
        // 恢复初始值
        this.updateNodes = []
        this.pCid = []
        this.draggable = false
      })
    },
    // 判断是否可以拖拽此节点位置
    allowDrop (draggingNode, dropNode, type) {
      console.log('拖动节点数据：', draggingNode, dropNode, type)
      this.countNodeLevel(draggingNode)
      let deep = Math.abs(this.maxLevel - draggingNode.level + 1)
      console.log(deep, this.maxLevel)
      if (type === 'inner') {
        this.maxLevel = 0
        return (deep + dropNode.level) <= 3
      } else {
        this.maxLevel = 0
        return (deep + dropNode.parent.level) <= 3
      }
    },
    // 计算获取节点总层数
    countNodeLevel (node) {
      if (node.childNodes !== null && node.childNodes.length > 0) {
        // node.forEach(element => {
        //   if (element.children.catLevel > this.maxLevel) {
        //     this.maxLevel = element.children.catLevel
        //   }
        //   this.countNodeLevel(element.children)
        // })
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    // 拖拽成功是调用的方法
    handleDrop (draggingNode, dropNode, dropType, ev) {
      console.log('tree drop: ', draggingNode, dropNode, dropType)
      let pCid = 0
      let siblings = null
      // 获取当前节点父节点ID
      if (dropType === 'before' || dropType === 'after') {
        pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }
      this.pCid.push(pCid)
      // 获取当前节点排序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          // 如果遍历的是当前节点
          let catLevel = draggingNode.level
          if (siblings[i].level !== draggingNode.level) {
            // 当前层级发生变化
            catLevel = siblings[i].level
            // 修改子节点层级
            this.updateNodesLevel(siblings[i])
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel
          })
        } else {
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i
          })
        }
      }
      // 获取当前节点的层级
      console.log('需要更新的节点数据：', this.updateNodes)
    },
    // 递归修改子节点层级
    updateNodesLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level
          })
          this.updateNodesLevel(node.childNodes[i])
        }
      }
    },
    // 初始化数据方法
    initialization () {
      this.category = {
        catId: null,
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: ''
      }
    }
  },

  created () {
    this.getMenus()
  }

}
</script>

<style lang="scss" scoped>

</style>