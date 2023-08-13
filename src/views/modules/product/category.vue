<template>
  <div>
    <el-tree :data="data" :props="defaultProps" show-checkbox node-key="catId" :default-expanded-keys=expandKeys>
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button type="text" size="mini" v-if="data.catLevel <= 2" @click="() => append(data)">
            添加
          </el-button>
          <el-button type="text" size="mini" v-if="data.catLevel <= 2" @click="() => update(data)">
            更新
          </el-button>
          <el-button type="text" size="mini" v-if="data.childrens.length == 0" @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="categoryForm" label-position="top">
        <el-form-item label="类别名称" :label-width="formLabelWidth" >
          <el-input v-model="categoryForm.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" :label-width="formLabelWidth">
          <el-input v-model="categoryForm.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" :label-width="formLabelWidth">
          <el-input v-model="categoryForm.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addDialog">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  data() {
    return {
      data: [],
      defaultProps: {
        children: 'childrens',
        label: 'name'
      },
      expandKeys: [],
      dialogVisible: false,
      categoryForm: {
        name: '',
        parentCid: '',
        catLevel: '',
        showStatus: '',
        icon: '',
        productUnit: ''
      },
      formLabelWidth: '120px'
    };
  },
  methods: {
    append(data) {
      console.log(data)
      this.dialogVisible = true
      this.categoryForm.parentCid = data.catId
      this.categoryForm.catLevel = data.catLevel + 1
      this.categoryForm.showStatus = 1
      this.categoryForm.sort = 0
    },
    addDialog(){
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data:this.$http.adornData(this.categoryForm,false)
      }).then(({data}) => {
        if(data && data.code == 0) {
          this.$message({
            message: "数据添加成功",
            type:"success"
          });
          this.dialogVisible=false
          this.getCategory()
          this.expandKeys = [this.categoryForm.parentCid]
        }
      })
    },

    remove(node, data) {
      this.$confirm(`此操作将删除【{${data.name}}】节点, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let ids = [data.catId]
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids,false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
            })
            this.getCategory()
            this.expandKeys = [node.parent.data.catId]
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    },
    getCategory() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        console.log(data)
        this.data = data.data
      })
    },
  
  }, created() {
    this.getCategory()
  }
};


</script>

<style></style>