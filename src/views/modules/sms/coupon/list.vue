<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('sms:coupon:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button
          v-if="isAuth('sms:coupon:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
        >批量删除</el-button>
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
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-card class="box-card">
            <el-form label-width="200px" label-position="left">
              <el-form-item label="优惠券有效期">
                <span>
                  <el-tag>{{scope.row.startTime}}</el-tag> - 
                  <el-tag type="danger">{{scope.row.endTime}}</el-tag>
                </span>
              </el-form-item>

              <el-form-item label="备注">
                <span>{{scope.row.note}}</span>
              </el-form-item>
              <el-form-item label="发行数量">
                <span>{{scope.row.publishCount}}</span>
              </el-form-item>
              <el-form-item label="已使用数量">
                <span>{{scope.row.useCount}}</span>
              </el-form-item>
              <el-form-item label="领取数量">
                <span>{{scope.row.receiveCount}}</span>
              </el-form-item>
              <el-form-item label="可领取日期">
                <span>
                  <el-tag>{{scope.row.enableStartTime}}</el-tag> - 
                  <el-tag type="danger">{{scope.row.enableEndTime}}</el-tag>
                </span>
              </el-form-item>
              <el-form-item label="可以领取的会员等级">
                <span>
                  <el-tag v-if="scope.row.memberLevel==0">不限等级</el-tag>
                  <el-tag v-else>{{scope.row.memberLevel}}</el-tag>
                </span>
              </el-form-item>
            </el-form>
          </el-card>
        </template>
      </el-table-column>
      <el-table-column prop="id" header-align="center" align="center" label="id"></el-table-column>
      <el-table-column prop="couponName" header-align="center" align="center" label="优惠卷名字"></el-table-column>
      <el-table-column prop="code" header-align="center" align="center" label="优惠码"></el-table-column>
      <el-table-column prop="couponType" header-align="center" align="center" label="优惠卷类型">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.couponType==0">全场赠券</el-tag>
          <el-tag v-if="scope.row.couponType==1">会员赠券</el-tag>
          <el-tag v-if="scope.row.couponType==2">购物赠券</el-tag>
          <el-tag v-if="scope.row.couponType==3">注册赠券</el-tag>
        </template>
      </el-table-column>
       <el-table-column prop="useType" header-align="center" align="center" label="使用类型">
         <template slot-scope="scope">
          <el-tag v-if="scope.row.useType==0">全场通用</el-tag>
          <el-tag v-if="scope.row.useType==1">指定分类</el-tag>
          <el-tag v-if="scope.row.useType==2">指定商品</el-tag>
         </template>
       </el-table-column>
      <el-table-column prop="num" header-align="center" align="center" label="数量"></el-table-column>
      <el-table-column prop="amount" header-align="center" align="center" label="金额"></el-table-column>
      <el-table-column prop="minPoint" header-align="center" align="center" label="使用门槛"></el-table-column>
      <el-table-column prop="perLimit" header-align="center" align="center" label="每人限领张数"></el-table-column>

      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button v-if="scope.row.useType==1" size="small" type="success" @click="couponCategoryHandle(scope.row.id)">维护分类</el-button>
          <el-button v-if="scope.row.useType==2" size="small" type="primary" @click="couponSpuHandle(scope.row.id)">维护商品</el-button>
          <div>
            <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
            <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
            <el-button type="text" size="small">发布</el-button>
          </div>
          <el-button  size="small" type="warning" @click="couponHistoryHandle(scope.row.id)">领取记录</el-button>
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
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <!-- 弹窗，分类维护 -->
    <coupon-category v-if="couponCategoryVisible" ref="couponCategory"></coupon-category>
    <!-- 弹窗，spu维护 -->
    <coupon-spu v-if="couponSpuVisible" ref="couponSpu"></coupon-spu>
  </div>
</template>

<script>
import AddOrUpdate from "./coupon-add-or-update";
import CouponCategory from "./coupon-category";
import CouponSpu from "./coupon-spu";
import { fail } from 'assert';
export default {
  data() {
    return {
      dataForm: {
        key: ""
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      couponCategoryVisible: false,
      couponSpuVisible: false
    };
  },
  components: {
    AddOrUpdate,CouponCategory,CouponSpu
  },
  activated() {
    this.getDataList();
  },
  methods: {
    couponHistoryHandle(couponId){
      // this.$router.replace("/sms-seckill/list");
      this.$router.push({ path: '/sms-coupon/coupon-history', query: { couponId: couponId }});
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornApiUrl("/sms/coupon/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
      }).then(({ data }) => {
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
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    //couponCategoryHandle
    couponCategoryHandle(id){
      this.couponCategoryVisible = true;//会引入dialog
      //等待dialog的dom准备完成
      this.$nextTick(() => {
        this.$refs.couponCategory.init(id);
      });
    },
    //优惠券对应的商品维护
    couponSpuHandle(id){
        this.couponSpuVisible = true;
        //由于引入的是一个组件，必须等待组件实例对象完成，才能调用方法。所以需要nextTick
        this.$nextTick(() => {
           this.$refs.couponSpu.init(id);
        });
    },


    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornApiUrl("/sms/coupon/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              }
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    }
  }
};
</script>
