<template>
  <el-dialog title="优惠券关联商品维护" :visible.sync="dialogVisible">
    <div>
      <!-- 引入组件 -->
      <el-button type="primary" @click="addCouponSpuHandle">新增关联商品</el-button>
      <el-button type="danger" @click="deleteCouponSpuHandle">删除关联</el-button>
      <coupon-spu-relation ref="couponSpuRelation" @selectedRow="selectedRow"></coupon-spu-relation>
      <coupon-spu-list
        v-if="couponSpuListVisible"
        ref="couponSpuList"
        @saveSuccess="couponSpuListSaveSuccess"
      ></coupon-spu-list>
    </div>
    <span slot="footer" class="dialog-footer"></span>
  </el-dialog>
</template>

<script>
import CouponSpuRelation from "./coupon-spu-relation";
import CouponSpuList from "./coupon-spu-list";
export default {
  components: {
    CouponSpuRelation,
    CouponSpuList
  },
  props: {},
  data() {
    return {
      dialogVisible: false,
      couponId: "",
      tableData: [],
      searchSpus: [],
      state: "",
      couponSpuListVisible: false,
      selectedRows: []
    };
  },
  watch: {},
  computed: {},
  methods: {
    init(id) {
      this.couponId = id;
      this.dialogVisible = true;
      this.$nextTick(() => {
        this.$refs.couponSpuRelation.init(this.couponId);
      });
    },
    deleteCouponSpuHandle() {
      var deletes = [];
      for (var i = 0; i < this.selectedRows.length; i++) {
        deletes.push(this.selectedRows[i].id);
      }
      this.$http({
        url: this.$http.adornApiUrl("/sms/couponspurelation/delete"),
        method: "post",
        data: this.$http.adornData(deletes, false, "json")
      }).then(({ data }) => {
        this.$message({
          message: "删除完成",
          type: "success",
          duration: 1500
        });
        this.$refs.couponSpuRelation.init(this.couponId);
      });
    },
    selectedRow(data) {
      this.selectedRows = data;
    },
    addCouponSpuHandle() {
      this.couponSpuListVisible = true;
      this.$nextTick(() => {
        //组件初始化完成
        this.$refs.couponSpuList.init(this.couponId);
      });
    },
    couponSpuListSaveSuccess() {
      this.$refs.couponSpuRelation.getDataList();
    }
  },
  created() {},
  mounted() {}
};
</script>
<style lang="scss" scoped>
</style>