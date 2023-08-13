<template>
  <div>
    <el-tree :data="data" :props="defaultProps"
      show-checkbox
      node-key="catId"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            v-if="data.catLevel <= 2"
            @click="() => append(data)">
            添加
          </el-button>
          <el-button
            type="text"
            size="mini"
            v-if="data.childrens.length == 0"
            @click="() => shanch(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
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
        }
      };
    },
    methods: {
      append(data) {
        console.log("add")
      },

      remove(node, data) {
       console.log("remove")
      },

      getCategory() {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          console.log(data)
          this.data = data.data
        })
      }
    },created(){
      this.getCategory()
    }
  };


</script>

<style>

</style>