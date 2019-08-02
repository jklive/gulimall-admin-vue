<!--  -->
<template>
  <el-dialog title="优惠券关联商品选择" :visible.sync="dialogVisible">
    <el-row>
      <el-col :span="8">
        <category-tree @treeNode="acceptTreeNode"></category-tree>
      </el-col>
      <el-col :span="16">
        <el-form :inline="true" :model="searchForm">
          <el-form-item>
            <el-input v-model="searchForm.key" placeholder="商品id，商品名"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button v-if="categoryId" type="success" @click="searchCategory">查本类</el-button>
            <el-button type="primary" @click="searchAll">查全部</el-button>
          </el-form-item>
        </el-form>
        <el-table
          v-loading="dataListLoading"
          :data="dataList"
          style="width: 100%"
          @selection-change="handleSelectionChange"
        >
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column prop="id" label="商品id"></el-table-column>
          <el-table-column prop="spuName" label="商品名字"></el-table-column>
          <el-table-column prop="spuDescription" label="描述"></el-table-column>
          <el-table-column prop="publishStatus" label="发布状态">
            <template slot-scope="scope">
              <el-tag type="warning" v-if="scope.row.publishStatus==0">未发布</el-tag>
              <el-tag type="success" v-if="scope.row.publishStatus==1">已发布</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="catalogId" label="所属分类">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{categoryNameShow(scope.row.catalogId) }}</span>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="sizeChangeHandle"
          @current-change="currentChangeHandle"
          :current-page="pageIndex"
          :page-sizes="[5, 10, 20, 50]"
          :page-size="pageSize"
          :total="totalPage"
          layout="total, sizes, prev, pager, next, jumper"
        ></el-pagination>
      </el-col>
    </el-row>

    <span slot="footer" class="dialog-footer">
      <el-button @click="cancelCouponSpu">取 消</el-button>
      <el-button type="primary" @click="saveCouponSpu">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import CategoryTree from "../../common/category-tree";

export default {
  //import引入的组件需要注入到对象中才能使用
  components: { CategoryTree },
  data() {
    //这里存放数据
    return {
      dialogVisible: false,
      searchForm: {
        key: ""
      },
      dataList: [],
      categoryId: "",
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      multipleSelection: [],
      couponId: ''
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    cancelCouponSpu() {},
    //
    saveCouponSpu() {
      //console.log(this.multipleSelection)
      for (var i = 0; i < this.multipleSelection.length; i++) {
        var data = {
          couponId: this.couponId,
          spuId: this.multipleSelection[i].id,
          spuName: this.multipleSelection[i].spuName
        };
        this.$http({
          url: this.$http.adornApiUrl("/sms/couponspurelation/save"),
          method: "post",
          data: this.$http.adornData(data, true, "json")
        }).then(({ data }) => {
        });
      };
      this.dialogVisible = false;
      this.$message({
          message: '保存优惠券与商品关系成功',
          type: 'success',
          duration: 1500
      });
      this.$emit("saveSuccess");
    },
    //async
    categoryNameShow(id) {
      // await this.$http({
      //   url: this.$http.adornApiUrl("/pms/category/info/" + id),
      //   method: "get",
      //   params: this.$http.adornParams({})
      // }).then(({ data }) => {
      //   console.log("11111");

      // });
      return id + "：未渲染";
    },
    init(id) {
      this.dialogVisible = true;
      this.couponId = id;
      this.getDataList();
    },
    acceptTreeNode(node, obj) {
      if (node.catLevel == 3) {
        this.categoryId = node.catId;
        this.getDataList();
      }
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      var params = {
        page: this.pageIndex,
        limit: this.pageSize,
        catId: this.categoryId
      };
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
    //查本分类符合条件的
    searchCategory() {
      ///pms/spuinfo/simple/search?catId=1&key=1
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    //查全部
    searchAll() {
      var params = {
        page: this.pageIndex,
        limit: this.pageSize,
        key: this.searchForm.key
      };
      this.search(params);
    },
    searchCategory() {
      var params = {
        page: this.pageIndex,
        limit: this.pageSize,
        key: this.searchForm.key,
        catId: this.categoryId
      };
      this.search(params);
    },
    search(params) {
      this.$http({
        url: this.$http.adornApiUrl("/pms/spuinfo/simple/search"),
        method: "get",
        params: this.$http.adornParams(params)
      }).then(({ data }) => {
        if (data && data.code == 0) {
          this.dataList = data.data.list;
          this.totalPage = data.data.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
        console.log("获取到的数据", this.dataList);
      });
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
//@import url(); 引入公共css类
</style>