<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="id">
      </el-table-column>
      <el-table-column
        prop="orderId"
        header-align="center"
        align="center"
        label="订单id">
      </el-table-column>
      <el-table-column
        prop="orderSn"
        header-align="center"
        align="center"
        label="订单号">
      </el-table-column>
      <el-table-column
        prop="consignee"
        header-align="center"
        align="center"
        label="收货人">
      </el-table-column>
      <el-table-column
        prop="consigneeTel"
        header-align="center"
        align="center"
        label="收货人电话">
      </el-table-column>
      <el-table-column
        prop="deliveryAddress"
        header-align="center"
        align="center"
        label="配送地址">
      </el-table-column>
      <el-table-column
        prop="orderComment"
        header-align="center"
        align="center"
        label="订单备注">
      </el-table-column>
      <el-table-column
        prop="paymentWay"
        header-align="center"
        align="center"
        label="付款方式">
        <!-- 【 1:在线付款 2:货到付款】 -->
      </el-table-column>
      <el-table-column
        prop="taskStatus"
        header-align="center"
        align="center"
        label="任务状态">
      </el-table-column>
      <el-table-column
        prop="orderBody"
        header-align="center"
        align="center"
        label="订单描述">
      </el-table-column>
      <el-table-column
        prop="trackingNo"
        header-align="center"
        align="center"
        label="物流单号">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="wareId"
        header-align="center"
        align="center"
        label="仓库id">
      </el-table-column>
      <el-table-column
        prop="taskComment"
        header-align="center"
        align="center"
        label="工作单备注">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">查看详情</el-button>
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
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornApiUrl('/wms/wareordertask/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.data.list
            this.totalPage = data.data.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
    }
  }
</script>
