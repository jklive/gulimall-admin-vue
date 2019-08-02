<template>
  <div>
    <el-row>
      <el-col :span="4">
        <div class="left-tree">
          <!--ref很重要，方便父组件触发子组件的一些方法  -->
          <CategoryTree @treeNode="acceptTreeNode" ref="categoryTreeComponent" />
        </div>
      </el-col>
      <el-col :span="20">
          <SpuInfoTable :cat-id="categoryInfo.catId" ref="spuInfoTable"/>
      </el-col>
    </el-row>
  </div>
</template>
<script>

import SpuInfoTable from "../spu/spuinfo";
import CategoryTree from "../../common/category-tree";
export default {
  components: { CategoryTree,SpuInfoTable },
  data() {
    return {
      currentTreeNode: {}, //原始的treeNode
      categoryInfo: {
        catId: "",
        isParent: "",
        level: "",
        parentCid: "",
        productCount: "",
        showStatus: "1",
        name: "",
        icon: "",
        productUnit: ""
      } //解构出来的category信息
    };
  },
  methods: {
    //接受子组件传递来的值，并保存
    acceptTreeNode(data) {
      this.currentTreeNode = data;
      this.categoryInfo = {
        catId: data.catId,
        isParent: data.isParent,
        level: data.level,
        parentCid: data.parentCid,
        productCount: data.productCount,
        showStatus: data.showStatus+"",
        name: data.name,
        icon: data.icon,
        productUnit: data.productUnit
      };
      console.log("现在的值",this.categoryInfo.catId);
      //自动改变查询，一定要用nextTick，否则prop属性还没传递下去，实例还不能使用
      this.$nextTick(()=>{
        this.$refs.spuInfoTable.getDataList(0);
      });
      
    }
  }
};
</script>
<style scoped>
</style>
