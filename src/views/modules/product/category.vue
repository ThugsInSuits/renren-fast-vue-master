<template>
  <div>
    <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽">
    </el-switch>
    <el-button type="primary" v-if="draggable" @click="updateDropData">批量更新</el-button>
    <el-button type="danger" @click="batchDeleteData">批量删除</el-button>
    <el-tree  ref="tree" :data="data" :props="defaultProps" show-checkbox node-key="catId" :default-expanded-keys=expandKeys
      :draggable="draggable" :allow-drop="allowDrop" @node-drop="handleDrop">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button type="text" size="mini" v-if="data.catLevel <= 2" @click="() => append(data)">
            添加
          </el-button>
          <el-button type="text" size="mini" @click="() => update(data)">
            更新
          </el-button>
          <el-button type="text" size="mini" v-if="data.childrens.length == 0" @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="dialogType ? `新增` : `修改`" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="categoryForm" label-position="top">
        <el-form-item label="类别名称" :label-width="formLabelWidth">
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
        <el-button type="primary" @click="submit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  data() {
    return {
      draggable:false,
      dialogType: false,// true 新增，false修改
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
      formLabelWidth: '120px',
      maxLevel: 0,
      updateChilds: [],
      pCids: []
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

      this.categoryForm.catId = null;
      this.categoryForm.icon = null;
      this.categoryForm.productUnit = null;

      this.dialogType = true
    },
    addDialog() {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.categoryForm, false)
      }).then(({ data }) => {
        if (data && data.code == 0) {
          this.$message({
            message: "数据添加成功",
            type: "success"
          });
          this.dialogVisible = false
          this.getCategory()
          this.expandKeys = [this.categoryForm.parentCid]
        }
      })
    },
    update(data) {
      // 获取最新的内容，防止其他用户更改数据后，展示错误
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'post',
        data: this.$http.adornData(this.categoryForm, false)
      }).then(({ data }) => {
        this.categoryForm.name = data.category.name;
        this.categoryForm.productUnit = data.category.productUnit
        this.categoryForm.icon = data.category.icon
        this.categoryForm.catLevel = data.category.catLevel
        // 填充更新数据的id
        this.categoryForm.catId = data.category.catId;
        this.categoryForm.showStatus = data.category.showStatus
        this.categoryForm.sort = 0
      })
      this.dialogVisible = true
      this.dialogType = false
    },
    updateDialog() {
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(this.categoryForm)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "数据修改成功",
            type: "success"
          });
          this.dialogVisible = false
          this.getCategory()
          this.expandKeys = [this.categoryForm.catId]
        }
      })
    },

    submit() {
      if (this.dialogType) {
        this.addDialog();
      } else {
        this.updateDialog();
      }
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
          data: this.$http.adornData(ids, false)
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
    handleDrop(draggingNode, dropNode, type) {
      console.log(draggingNode, dropNode, type)
      // 1.拖拽节点的父节点需要更改
      let parentCid = null;
      // 拖拽节点的子节点
      let subChild = null;
      if (type === "inner") {
        parentCid = dropNode.data.catId;
        subChild = dropNode.childNodes;
      } else {
        parentCid = dropNode.data.parentCid == undefined ? 0 : dropNode.data.parentCid;
        subChild = dropNode.parent.childNodes
      }

      // 2.拖拽后节点所在的新的兄弟节点间排序问题
      for (let i = 0; i < subChild.length; i++) {
        // 循环子节点的时候，如果说子节点是要移动的那个节点，那么该节点下的所有节点都需要变动
        // 如果不是则直接入数组
        if (subChild[i].data.catId == draggingNode.data.catId) {
          let oriLevel = draggingNode.data.catLevel
          if (subChild[i].level != oriLevel) {
            // 层级发生移动，需要修改该节点层级以及子节点的层级
            oriLevel = subChild[i].level;
            this.updateChildLevel(subChild[i])
          }
          this.updateChilds.push({
            catId: subChild[i].data.catId,
            sort: i,
            catLevel: oriLevel,
            parentCid: parentCid
          })
        } else {
          this.updateChilds.push({ catId: subChild[i].data.catId, sort: i })
        }
      }
      // 3.拖拽后的节点及其节点的层级需要改变
      // console.log(this.updateChilds)
      // this.updateDropData()
      this.pCids.push(parentCid);
    },
    updateChildLevel(subChild) {
      if (subChild.childNodes != null && subChild.childNodes.length > 0) {
        for (let i = 0; i < subChild.childNodes.length; i++) {
          let childNode = subChild.childNodes[i]
          this.updateChilds.push({
            catId: childNode.data.catId,
            catLevel: childNode.level
          })
          this.updateChildLevel(childNode)
        }
      }
    },
    updateDropData() {
      this.$http({
        url: this.$http.adornUrl("/product/category/batchUpdate"),
        method: "post",
        data: this.$http.adornData(this.updateChilds, false)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "数据拖拽成功",
            type: "success"
          });
          this.getCategory();
          this.expandKeys = this.pCids;
          this.maxLevel = 0;
          this.updateChilds = []
        } else {
          this.$message.error(data.msg);
        }
      })
    },
    // 判断拖拽节点是否可以拖拽
    // 'prev'、'inner' 和 'next'，分别表示放置在目标节点前、插入至目标节点和放置在目标节点后
    allowDrop(draggingNode, dropNode, type) {
      this.countLevel(draggingNode);
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1
      console.log(deep, this.maxLevel)
      if (type == 'inner') {
        return deep + dropNode.level <= 3
      }
      return deep + dropNode.parent.level <= 3
    },
    countLevel(draggingNode) {
      this.maxLevel = draggingNode.data.catLevel;
      if (draggingNode.childNodes != null && draggingNode.childNodes.length > 0) {
        for (let i = 0; i < draggingNode.childNodes.length; i++) {
          if (draggingNode.childNodes[i].level > this.maxLevel) {
            this.maxLevel = draggingNode.childNodes[i].level
          }
          this.countLevel(draggingNode.childNodes[i])
        }
      }
    },
    batchDeleteData(){
      // 批量删除选中节点
      let checkedNodes = [];
      let checkNodes = this.$refs.tree.getCheckedNodes(false,false)
      console.log("aa",checkNodes)
      for(let i=0;i< checkNodes.length;i++){
        checkedNodes.push(checkNodes[i].catId);
      }
      this.$http({
        url: this.$http.adornUrl("/product/category/delete"),
        method: "post",
        data: this.$http.adornData(checkedNodes, false)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "批量删除成功",
            type: "success"
          });
          this.getCategory();
        } else {
          this.$message.error(data.msg);
        }
      })

    }

  }, created() {
    this.getCategory()
  }
};


</script>

<style></style>