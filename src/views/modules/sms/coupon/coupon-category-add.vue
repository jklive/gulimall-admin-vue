<template>
  <el-dialog title="选择三级分类" :visible.sync="dialogTableVisible">
    <el-row>
      <el-col :span="10">
        <category-tree @treeNode="acceptTreeNode"></category-tree>
      </el-col>
      <el-col :span="14">
        <el-card shadow="always" v-if="cardShow">
          <h2>已选择的分类</h2>
          <hr />
          <el-tag
            style="margin-left:10px;margin-bottom:10px;"
            v-for="node in selectedTreeNodes"
            :key="node.catId"
            effect="dark"
            closable
            @close="unselectCategory(node.catId)"
          >{{node.name}}</el-tag>
          <div v-if="btnShow">
            <hr />
            <el-button>取消</el-button>
            <el-button type="primary" @click="saveCouponCategory">确定</el-button>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </el-dialog>
</template>

<script>
import CategoryTree from "../../common/category-tree";
import { fail } from "assert";
export default {
  components: { CategoryTree },
  props: {},
  data() {
    return {
      dialogTableVisible: false,
      couponId: "",
      selectedTreeNodes: [],
      btnShow: false,
      cardShow: false
    };
  },
  watch: {},
  computed: {},
  methods: {
    init(id) {
      this.dialogTableVisible = true;
      this.couponId = id;
      this.cardShow = true;
    },
    acceptTreeNode(node, obj) {
      console.log("选中的节点，", node, obj);
      if (node.catLevel === 3) {
        //如果是三级分类，并且之前没加过
        var flag = false;
        for (var i = 0; i < this.selectedTreeNodes.length; i++) {
          if (this.selectedTreeNodes[i].catId == node.catId) {
            flag = true;
            break;
          }
        }
        if (!flag) {
          this.selectedTreeNodes.push(node);
          this.btnShow = true;
        }
      }
    },
    unselectCategory(id) {
      console.log("node", id);
      console.log(this.selectedTreeNodes);
      for (var i = 0; i < this.selectedTreeNodes.length; i++) {
        if (this.selectedTreeNodes[i].catId == id) {
          this.selectedTreeNodes.splice(i, 1);
        }
      }
    },
    saveCouponCategory() {
      //  this.dialogTableVisible = false;
      //  this.cardShow = false;
      //  this.btnShow = false;
      //  this.selectedTreeNodes = [];
      // 发送请求，保存数据；
      var postDatas = new Array();
      for (var i = 0; i < this.selectedTreeNodes.length; i++) {
        var item = {
          categoryId: this.selectedTreeNodes[i].catId,
          categoryName: this.selectedTreeNodes[i].name,
          couponId: this.couponId
        };
        postDatas.push(item);
      }
      this.$http({
        url: this.$http.adornApiUrl("/sms/couponspucategoryrelation/save"),
        method: "post",
        data: this.$http.adornData(postDatas, false)
      }).then(data => {
          this.$message({
              message: "保存成功",
              type: "success",
              duration: 1500
          }); 
          //关闭对话框，刷新数据
          this.dialogTableVisible = false;
          this.cardShow = false;
          this.btnShow = false;
          this.selectedTreeNodes = [];
          //向父及组件发送事件；
          this.$emit("saveSuccess",data);
      });
    }
  },
  created() {},
  mounted() {}
};
</script>
<style lang="scss" scoped>
</style>