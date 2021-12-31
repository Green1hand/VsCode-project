<template>
    <div>
        <el-dialog 
            title="关联分类"
            ref="brandcategory"
            center
            width = 500px
            :close-on-click-modal="false"
            :visible.sync="visible">
            <el-popover
                placement="right"
                width="250"
                v-model="hidden"
                trigger="manual">
                <div style="height: 300px;; overflow-y:scroll">
                    <el-tree
                        :data="menus" 
                        :props="defaultProps" 
                        :highlight-current="true"
                        :accordion="true"
                        @node-click="handleNodeClick">        
                    </el-tree>
                </div>    
                <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
                <el-button type="primary" slot="reference" @click="(getMenus())">{{!hidden? "新增关联":"取消新增"}}</el-button>
            </el-popover>
            <el-table
                :data="tableData"
                style="width: 100%; min-height: 300px;">
                <el-table-column
                    type="index"
                    width="50">
                </el-table-column>
                <el-table-column
                    prop="brandName"
                    label="品牌名"
                    width="145">
                </el-table-column>
                <el-table-column
                    prop="catelogName"
                    label="分类名"
                    width="145">
                </el-table-column>
                <el-table-column label="操作" width="100">
                    <template slot-scope="scope">
                        <el-button
                        size="mini"
                        type="danger"
                        @click="handleDelete(scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: 'VscodeProjectBrandCategroy',

    data() {
        return {
            visible: false,
            hidden: false,
            brandId: 0,
            tableData: [],
            CategoryBrandRelationEntity: {},
            menus: [],
            defaultProps: {
                children: 'children',
                label: 'name'
            }
        }
    },

    mounted() {
        
    },

    methods: {
        // 获取关联分类
        BrandCategory(brandId) {
            this.visible = true
            console.log("父组件传递的数据：", brandId)
            this.brandId = brandId
            this.$http({
                url: this.$http.adornUrl('/product/categorybrandrelation/catelog/list'),
                method: 'get',
                params: this.$http.adornParams({brandId})
            }).then(({data}) => {
                console.log("获取的关联分类数据：", data)
                this.tableData=data.data
            })
        },
        // 获取分类信息
        getMenus () {
            console.log("当前显示状态：", this.hidden)
            this.hidden = !this.hidden
            if (this.hidden) {
                this.$http({
                    url: this.$http.adornUrl('/product/category/list/tree'),
                    method: 'get'
                }).then(({data}) => {
                    console.log("获取的分类数据：", data.data)
                    this.menus = data.data
                })
            }
        },
        // 删除
        handleDelete (row) {
            console.log("选中的行数据：", row)
            var ids = [row.id]
            this.$confirm(`此操作将取消关联【${row.catelogName}】分类`, ' 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl('/product/categorybrandrelation/delete'),
                    method: 'post',
                    data: this.$http.adornData(ids, false)
                }).then(({data}) => {
                    this.$message({
                        type: data.msg,
                        message: data.msg
                    })
                    // 刷新数据
                    this.BrandCategory(this.brandId)
                })
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                })
            })
        },
        // 添加
        dataFormSubmit () {
            console.log("需要添加的数据id：", this.CategoryBrandRelationEntity)
            this.$http({
                url: this.$http.adornUrl('/product/categorybrandrelation/save'),
                method: 'post',
                data: this.$http.adornData(this.CategoryBrandRelationEntity, false)
            }).then(({ data }) => {
                this.$message({
                    type: data.msg,
                    message: data.msg
                })
                this.CategoryBrandRelationEntity = {}
                 // 刷新数据
                this.BrandCategory(this.brandId)
                // 刷新状态
                this.hidden = !this.hidden
            })
        },
        handleNodeClick(data) {
            console.log("选中的节点数据：", data)
            this.CategoryBrandRelationEntity = {
                brandId: this.brandId,
                catelogId: data.catId
            }        
        }
    },
};
</script>

<style lang="scss" scoped>
    // .el-popover{
    //     height: 400px;
    //     overflow: auto;
    // }
</style>