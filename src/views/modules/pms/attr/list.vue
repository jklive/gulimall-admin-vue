<!--  -->
<template>
  <div class="mod-config">
    <el-row>
      <el-col :span="4">
        <div class="left-tree">
          <!--ref很重要，方便父组件触发子组件的一些方法  -->
          <CategoryTree @treeNode="acceptTreeNode" ref="categoryTreeComponent" />
        </div>
      </el-col>
      <el-col :span="20">
        <el-row>
          <el-form class="el-form el-form--inline">
            <el-form-item>
              <el-button type="primary" size="mini" @click="addAttributeHandle(categoryId)">添加属性</el-button>
              <el-button type="primary" size="mini" @click="attrGroupHandle(categoryId)">添加属性分组</el-button>
            </el-form-item>
          </el-form>
        </el-row>
        <div>
          <el-collapse>
            <el-collapse-item title="属性分组" name="1">
              <attr-group :cat-id="categoryId" ref="attrGroup"></attr-group>
            </el-collapse-item>
            <el-collapse-item title="基本属性" name="2">
              <base-attr :cat-id="categoryId" ref="baseAttrTable"></base-attr>
            </el-collapse-item>
            <el-collapse-item title="销售属性" name="3">
              <sale-attr :cat-id="categoryId" ref="saveAttrTable"></sale-attr>
            </el-collapse-item>
          </el-collapse>
        </div>
      </el-col>
    </el-row>
    <attribute-add-or-updata
      :cat-id="categoryId"
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @saveSuccess="attrsSaveSuccess"
    ></attribute-add-or-updata>
    <attr-group-add-update
      :cat-id="categoryId"
      v-if="attrGroupVisible"
      ref="attrGroupAddUpdate"
      @saveSuccess="groupSaveSuccess"
    ></attr-group-add-update>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import CategoryTree from "../../common/category-tree";
import AttributeAddOrUpdata from "./attribute-add-update";
import AttrGroup from "./attr-group";
import BaseAttr from "./base-attr";
import SaleAttr from "./sale-attr";
import AttrGroupAddUpdate from "./attr-group-add-update";
export default {
  //import引入的组件需要注入到对象中才能使用
  components: {
    CategoryTree,
    AttributeAddOrUpdata,
    AttrGroup,
    BaseAttr,
    SaleAttr,
    AttrGroupAddUpdate
  },
  data() {
    //这里存放数据
    return {
      treeObj: {},
      currentTreeNode: {}, //原始的treeNode
      categoryId: "", //当前商品的ID
      addOrUpdateVisible: false,
      attrGroupVisible: false
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    // 接受ztree树
    acceptTreeNode(data, treeObj) {
      this.currentTreeNode = data;
      this.treeObj = treeObj;
      if (this.currentTreeNode.catLevel !== 3) {
        return false;
      }
      this.categoryId = this.currentTreeNode.catId;
      this.$http({
        url: this.$http.adornApiUrl("/pms/attr/base/" + this.categoryId),
        method: "get"
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.baseTableData = data.data.list;
          this.basetotalCount = data.data.totalCount;
        } else {
        }
      });
    },
    // 添加属性
    addAttributeHandle(id) {
      if (!this.categoryId) {
        this.$alert("请先选择三级分类");
      } else {
        this.addOrUpdateVisible = true;
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(this.categoryId);
        });
      }
    },
    // 维护属性分组
    attrGroupHandle(catId) {
      if (!this.categoryId) {
        this.$alert("请先选择三级分类");
      } else {
        this.attrGroupVisible = true;
        this.$nextTick(() => {
          this.$refs.attrGroupAddUpdate.init(catId);
        });
      }
    },
    attrsSaveSuccess(type) {
      console.log("attrsSaveSuccess...", type);
      if (!type && type == 0) {
        this.$refs.saveAttrTable.getDataList();
      } else if (type == 1) {
        this.$refs.baseAttrTable.getDataList();
      }
    },
    groupSaveSuccess() {
      this.$refs.attrGroup.initAttrGroup(this.categoryId);
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {},
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {} //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style lang='scss' scoped>
</style>