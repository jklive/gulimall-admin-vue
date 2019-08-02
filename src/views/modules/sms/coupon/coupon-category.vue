<template>
  <div>
    <el-dialog title="优惠券可用分类维护" :visible.sync="dialogTableVisible">
      <el-button type="primary" size="small" @click="addCategoryHandle()">新增关联分类</el-button>
      <el-button type="danger" size="small" @click="deleteCategoryHandle">删除选中关联</el-button>
      <el-table :data="categoryData" @selection-change="handleSelectionChange">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column property="categoryId" label="分类id"></el-table-column>
        <el-table-column property="categoryName" label="分类名"></el-table-column>
      </el-table>
    </el-dialog>
    <coupon-category-add
      v-if="dialogCategoryTableVisible"
      ref="couponCategoryAdd"
      @saveSuccess="init(couponId)"
    ></coupon-category-add>
  </div>
</template>

<script>
import CouponCategoryAdd from "./coupon-category-add";
export default {
  components: { CouponCategoryAdd },
  props: {},
  data() {
    return {
      categoryData: [],
      dialogTableVisible: false,
      dialogCategoryTableVisible: false,
      couponId: "",
      multiselects: []
    };
  },
  watch: {},
  computed: {},
  methods: {
    deleteCategoryHandle() {
      //获取选中的数据
      console.log("将要删除的数据；", this.multiselects);
      var ids = [];
      for (var i = 0; i < this.multiselects.length; i++) {
        ids.push(this.multiselects[i].id);
      }
      console.log("将要删除的数据ids；", ids);
      this.$http({
        url: this.$http.adornApiUrl("/sms/couponspucategoryrelation/delete"),
        method: "post",
        data: this.$http.adornData(ids, false)
      }).then(data => {
          this.$message({
                message: "分类关系删除成功",
                type: "success",
                duration: 1500
          });
          //刷新表格
          this.init(this.couponId);
      });
    },
    handleSelectionChange(val) {
      this.multiselects = val;
    },
    init(id) {
      this.dialogTableVisible = true;
      this.couponId = id;
      this.$nextTick(() => {
        //dialog实例化对象完成，发送请求获取表格数据
        this.$http({
          url: this.$http.adornApiUrl(
            `/sms/couponspucategoryrelation/category/list/${this.couponId}`
          ),
          method: "get",
          params: this.$http.adornParams({
            limit: 200000
          })
        }).then(({ data }) => {
          this.categoryData = data.data.list;
          console.log("获取到的数据", this.categoryData);
        });
      });
    },
    addCategoryHandle() {
      //添加三级分类框打开
      this.dialogCategoryTableVisible = true;
      this.$nextTick(() => {
        this.$refs.couponCategoryAdd.init(this.couponId);
      });
    }
  },
  created() {},
  mounted() {}
};
</script>
<style lang="scss" scoped>
</style>