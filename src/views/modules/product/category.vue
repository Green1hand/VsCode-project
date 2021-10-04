<<template>
  <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
  </el-tree>
</template>

<script>
export default {
  name: 'VscodeProjectCategory',

  data () {
    return {
      menus: [],
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
    append (data) {
      console.log('append', data)
    },
    remove (node, data) {
      console.log('remove', node, data)
    },
    handleNodeClick (data) {
      console.log(data)
    }
  },

  created () {
    this.getMenus()
  }

}
</script>

<style lang="scss" scoped>

</style>