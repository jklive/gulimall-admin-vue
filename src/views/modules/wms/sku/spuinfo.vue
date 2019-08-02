<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="商品id,商品名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList(0)">查本类</el-button>
        <el-button @click="getDataList(1)">查全站</el-button>
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
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="skuUpdateHandle(scope.row.id)">库存维护</el-button>
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
    <skustock ref="skustock"></skustock>
  </div>
</template>

<script>
import Skustock from "./skustock"
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
      all: 0
    };
  },
  props:["catId"],
  components: {
    Skustock
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    getDataList(all) {
      this.all = all;
      console.log("要去查询的值",this.catId);
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornApiUrl("/pms/spuinfo/simple/search"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          catId: this.all == 0?this.catId:0
        })
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
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList(this.all);
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList(this.all);
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    skuUpdateHandle(id){
      this.$refs.skustock.init(id);
    }
  }
};
</script>
