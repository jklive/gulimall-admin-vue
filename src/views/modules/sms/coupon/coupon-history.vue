<!--  -->
<template>
  <div>
    <el-form :inline="true" :model="searchForm" class="demo-form-inline">
      <el-form-item label="id">
        <el-input v-model="searchForm.id" size="mini" placeholder="输入关键字搜索" /> 
      </el-form-item>
      <el-form-item>
        <el-button>查询</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="tableData" style="width: 100%;">
      <el-table-column prop="id" label="id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="couponId" label="优惠卷id" header-align="center" align="center"></el-table-column>
      <el-table-column label="领取的会员" header-align="center" align="center">
        <el-table-column prop="memberId" label="id" header-align="center" align="center"></el-table-column>
        <el-table-column prop="memberNickName" label="名字" header-align="center" align="center"></el-table-column>
      </el-table-column>
      <el-table-column label="使用的订单" header-align="center" align="center">
        <el-table-column prop="orderId" label="订单id" header-align="center" align="center"></el-table-column>
        <el-table-column prop="orderSn" label="订单号" header-align="center" align="center"></el-table-column>
      </el-table-column>
      <el-table-column prop="useTime" label="使用时间" header-align="center" align="center" width="180px"></el-table-column>
      <el-table-column prop="useType" label="使用的类型" header-align="center" align="center">
        <template slot-scope="scope">
          <!-- 0->未使用；1->已使用；2->已过期 -->
          <el-tag type="success" v-if="scope.row.useType == 0">未使用</el-tag>
          <el-tag v-if="scope.row.useType == 1">已使用</el-tag>
          <el-tag v-if="scope.row.useType == 2" type="info">已过期</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" label="领取时间" header-align="center" align="center" width="180px"></el-table-column>
      <el-table-column prop="getType" label="获取方式" header-align="center" align="center">
        <template slot-scope="scope">
          <!-- 0->未使用；1->已使用；2->已过期 -->
          <el-tag type="success" v-if="scope.row.getType == 0">后台赠送</el-tag>
          <el-tag v-if="scope.row.getType == 1">主动领取</el-tag>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="totalCount"
    ></el-pagination>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    //这里存放数据
    return {
      tableData: [],
      pageIndex: 1,
      pageSize: 10,
      totalCount: 0,
      dataListLoading: false,
      searchForm: {
        id: "",
        couponId: ""
      }
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    handleSizeChange(val) {
      this.pageSize = val;
      this.getDataList();
    },
    handleCurrentChange(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    getDataList() {
      this.$http({
        url: this.$http.adornApiUrl("/sms/couponhistory/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize
          //   key: this.dataForm.key
        })
      }).then(({ data }) => {
        this.tableData = data.data.list;
        this.pageIndex = data.data.currPage;
        this.pageSize = data.data.pageSize;
        this.totalCount = data.data.totalCount;
      });
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {},
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.getDataList();
  },
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