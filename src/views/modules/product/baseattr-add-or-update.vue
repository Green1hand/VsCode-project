<template>
  <el-dialog
    :title="!dataForm.attrGroupId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    @closed="dialogClose">
      <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="名称" prop="attrName">
        <el-input v-model="dataForm.attrName" placeholder="名称"></el-input>
      </el-form-item>
      <el-form-item label="可检索" prop="searchType">
        <el-switch
          v-model="dataForm.searchType  "
          :active-value="1"
          :inactive-value="0"
          active-color="#13ce66"
          inactive-color="#ff4949">
        </el-switch>
      </el-form-item>
      <el-form-item label="值类型" prop="valueType">
        <el-select v-model="dataForm.valueType" placeholder="选择类型" clearable>
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="图标"></el-input>
      </el-form-item>
      <el-form-item label="可选值" prop="valueSelect">
        <el-input v-model="dataForm.valueSelect" placeholder="可选值"></el-input>
      </el-form-item>
      <el-form-item label="启用" prop="enable">
        <el-switch
          v-model="dataForm.enable"
          :active-value="1"
          :inactive-value="0"
          active-color="#13ce66"
          inactive-color="#ff4949">
        </el-switch>
      </el-form-item>
      <el-form-item label="所属分类" prop="catelogId">
      <!-- <el-input v-model="dataForm.catelogId" placeholder="所属分类id"></el-input> -->
      <el-cascader
        style="width: 400px"
        v-model="dataForm.catelogPath"
        :options="categorys"
        :props="props"
        placeholder="试试搜索"
        filterable
        clearable>
      </el-cascader>
      </el-form-item>
      <el-form-item label="所属分组" prop="attrgroupId">
        <el-select v-model="dataForm.attrgroupId" placeholder="选择类型" clearable>
          <el-option
            v-for="item in attrgroup"
            :key="item.attrGroupId"
            :label="item.attrGroupName"
            :value="item.attrGroupId">
          </el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        props: {
          value: 'catId',
          label: 'name',
          children: 'children'
        },
        categorys: [],
        dataForm: {
          attrId: 0,
          attrName: '',
          searchType: 0,
          valueType: '',
          icon: '',
          valueSelect: '',
          enable: 0,
          catelogId: 0,
          catelogPath: [],
          attrgroupId: '',
          attrgroupName: ''
        },
        options: [{
          value: 0,
          label: '类型1'
        }, {
          value: 1,
          label: '类型2'
        }, {
          value: 2,
          label: '类型3'
        }],
        attrgroup: [],
        dataRule: {
          attrGroupName: [
            { required: true, message: '组名不能为空', trigger: 'blur' }
          ],
          sort: [
            { required: true, message: '排序不能为空', trigger: 'blur' }
          ],
          descript: [
            { required: true, message: '描述不能为空', trigger: 'blur' }
          ],
          icon: [
            { required: true, message: '组图标不能为空', trigger: 'blur' }
          ],
          catelogId: [
            { required: true, message: '所属分类id不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      getCategorys () {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get'
        }).then(({data}) => {
          console.log(data.data)
          this.categorys = data.data
        })
      },
      init (id) {
        this.dataForm.attrId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.attrId) {
            this.$http({
              url: this.$http.adornUrl(`/product/attr/info/${this.dataForm.attrId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              console.log('获取的规格数据:', data)
              if (data && data.code === 0) {
                this.dataForm.attrId = data.attr.attrId
                this.dataForm.attrName = data.attr.attrName
                this.dataForm.searchType = data.attr.searchType
                this.dataForm.valueType = data.attr.valueType
                this.dataForm.icon = data.attr.icon
                this.dataForm.valueSelect = data.attr.valueSelect
                this.dataForm.enable = data.attr.enable
                this.dataForm.catelogId = data.attr.catelogId
                this.dataForm.catelogPath = data.attr.catelogPath
                this.dataForm.attrgroupId = data.attr.attrgroupId
                this.dataForm.attrgroupName = data.attr.attrgroupName
              }
            })
            this.initAttrgroup()
          }
        })
      },
      initAttrgroup () {
        this.$http({
          url: this.$http.adornUrl(`/product/attrgroup/list`),
          method: 'get',
          params: this.$http.adornParams()
        }).then((data) => {
          console.log('获取的分组数据：', data.data)
          data.data.attrgroup.forEach((item) => {
            this.attrgroup.push({
              value: item.attrGroupId,
              label: item.attrGroupName
            })
          })
          this.attrgroup = data.data.attrgroup
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            console.log('修改后的数据', this.dataForm)
            this.$http({
              url: this.$http.adornUrl(`/product/attr/${!this.dataForm.attrId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'attrId': this.dataForm.attrId || undefined,
                'attrName': this.dataForm.attrName,
                'searchType': this.dataForm.searchType,
                'valueType': this.dataForm.valueType,
                'icon': this.dataForm.icon,
                'valueSelect': this.dataForm.valueSelect,
                'enable': this.dataForm.enable,
                'catelogPath': this.dataForm.catelogPath,
                'attrgroupId': this.dataForm.attrgroupId,
                'attrgroupName': this.dataForm.attrgroupName,
                'catelogId': this.dataForm.catelogPath[this.dataForm.catelogPath.length - 1]
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      dialogClose () {
        this.dataForm.catelogPath = []
      }
    },
    created () {
      this.getCategorys()
    }
  }
</script>
