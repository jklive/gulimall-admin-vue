<!--  -->
<template>
  <div class>
    <el-row>
      <el-col :span="6">
        <category-tree @treeNode="acceptTreeNode"></category-tree>
      </el-col>
      <el-col :span="18">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
          <el-form-item>
            <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" v-if="currentNode" @click="searchCategory">查本类</el-button>
            <el-button type="success" @click="searchAll">查全站</el-button>
            <!-- utils里面index.js判断权限，将登录后的所有权限都保存在了sessionStorage中 v-if="isAuth('pms:spuinfo:save')"  v-if="isAuth('pms:spuinfo:delete')" -->
            <el-button
              type="warning"
              style="float:right;"
              @click="discountDialog(dataListSelections)"
            >批量设置打折</el-button>
          </el-form-item>
        </el-form>
        <el-table
          :data="dataList"
          border
          v-loading="dataListLoading"
          @selection-change="selectionChangeHandle"
          style="width: 100%;"
        >
          <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
          <el-table-column prop="id" header-align="center" align="center" label="商品id"></el-table-column>
          <el-table-column prop="spuName" header-align="center" align="center" label="商品名称"></el-table-column>
          <el-table-column prop="spuDescription" header-align="center" align="center" label="商品描述"></el-table-column>
          <el-table-column prop="catalogId" header-align="center" align="center" label="所属分类"></el-table-column>
          <el-table-column prop="brandId" header-align="center" align="center" label="品牌"></el-table-column>
          <el-table-column prop="publishStatus" header-align="center" align="center" label="上架状态"></el-table-column>
          <el-table-column prop="createTime" header-align="center" align="center" label="创建时间"></el-table-column>
          <el-table-column prop="uodateTime" header-align="center" align="center" label="更新时间"></el-table-column>
          <el-table-column header-align="center" align="left" label="优惠操作">
            <template slot-scope="scope">
              <el-button
                type="primary"
                icon="el-icon-edit"
                circle
                @click="discountDialog([scope.row])"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="sizeChangeHandle"
          @current-change="currentChangeHandle"
          :current-page="pageIndex"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="pageSize"
          :total="totalPage"
          layout="total, sizes, prev, pager, next, jumper"
        ></el-pagination>
      </el-col>
    </el-row>
    <discount-dialog v-if="discountDialogShow" ref="discountDialog"></discount-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import CategoryTree from "../../common/category-tree";
import DiscountDialog from "./discount-dialog";
export default {
  //import引入的组件需要注入到对象中才能使用
  components: { CategoryTree, DiscountDialog },
  data() {
    //这里存放数据
    return {
      currentNode: "",
      dataForm: {
        key: ""
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      searchType: 1, //0 查本类  1 查全站
      catId: null,
      discountDialogShow: false
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    search(params) {
      this.$http({
        url: this.$http.adornApiUrl("/pms/spuinfo/simple/search"),
        method: "get",
        params: this.$http.adornParams(params)
      }).then(({ data }) => {
        console.log(data);
        if (data && data.code === 0) {
          this.dataList = data.data.list;
          this.totalPage = data.data.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    searchCategory() {
      this.catId = this.currentNode.catId;
      this.getDataList();
    },
    searchAll() {
      this.catId = null;
      this.getDataList();
    },
    acceptTreeNode(node, obj) {
      if (node.catLevel == 3) {
        this.currentNode = node;
        //查询
        this.catId = node.catId;
        this.getDataList();
      }
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      var params;
      if (this.catId) {
        params = {
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          catId: this.catId
        };
      } else {
        params = {
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        };
      }

      this.search(params);
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    discountDialog(ids) {
      if (ids.length != 0) {
        this.discountDialogShow = true;
        this.$nextTick(() => {
          this.$refs.discountDialog.init(ids);
        });
      }else{
          this.$message("请先选择需要设置打折的商品")
      }
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
  activated() {
    this.getDataList();
  } //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style lang='scss' scoped>
//@import url(); 引入公共css类
</style>