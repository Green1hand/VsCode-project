<template>
    <el-tree 
        ref="MenusTree"
        node-key="catId"
        :data="menus" 
        :props="defaultProps"
        @node-click="nodeclick">
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
        };
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
        nodeclick (data, node, component) {
            console.log('子节被点击了：', data, node, component)
            this.$emit('tree-node-click', data, node, component);
        }
    },

    created () {
        this.getMenus()
    }
}
</script>

<style lang="scss" scoped>

</style>