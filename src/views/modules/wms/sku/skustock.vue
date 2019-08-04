<template>
  <div class="wrapper">
    <el-dialog title="sku维护" :visible.sync="dialogVisible">
      <el-table :data="tableData" style="width: 100%">
        <el-table-column label="skuId" prop="skuId"></el-table-column>
        <el-table-column label="sku名字" prop="skuName"></el-table-column>
        <el-table-column label="sku标题" prop="skuTitle"></el-table-column>
        <el-table-column label="sku价格" prop="price"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button @click="editSku(scope.row)" type="primary" size="mini">库存维护</el-button>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="库存维护" :visible.sync="stockDialogVisible">
      <el-button size="mini" type="primary" @click="addSkuStockData">新增库存信息</el-button>
      <el-table :data="stockTableData">
        <el-table-column label="#" prop="id" width="50"></el-table-column>
        <el-table-column label="skuId" prop="skuId" width="70"></el-table-column>
        <el-table-column label="商品名" prop="skuName"></el-table-column>
        <el-table-column label="库存" prop="stock">
          <template slot-scope="scope">
            <!-- <span v-if="!scope.row.showStock">{{scope.row.stock}}</span> -->
            <el-input v-model="scope.row.stock" placeholder="请输入内容"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="所在仓库" prop="wareId" width="150">
          <template slot-scope="scope">
            <!-- <span v-if="!scope.row.showWare">{{scope.row.wareName}}</span> -->
            <el-select
              v-model="scope.row.wareId"
              placeholder="请选择"
              @change="showWareName(scope.row)"
            >
              <el-option v-for="item in wares" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </template>
        </el-table-column>
        <el-table-column label="锁定" prop="stockLocked"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- <el-button size="mini" type="text" @click="editSkuStock(scope.row)">修改</el-button> -->
            <el-button size="mini" type="text" @click="saveStockInfo(scope.row)">保存</el-button>
            <el-button size="mini" type="text" @click="deleteStockInfo(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      dialogVisible: false,
      stockDialogVisible: false,
      spuId: 0,
      tableData: [],
      stockTableData: [],
      wares: [],
      oldRow: {},
      currentSku: {}
    };
  },
  watch: {},
  computed: {},
  methods: {
    init(id) {
      this.spuId = id;
      this.dialogVisible = true;
      //查出当前spu的所有sku以及库存信息
      this.getSkusBySpuId(this.spuId);
    },
    addSkuStockData() {
      var sku = {
        skuId: this.currentSku.skuId,
        skuName: this.currentSku.skuName,
        stock: 0,
        wareId: null,
        stockLocked: 0
      };
      this.stockTableData.push(sku);
    },
    editSkuStock(row) {
      row.showStock = true;
      row.showWare = true;
      console.log("editSkuStock====", row);
    },
    deleteStockInfo(row) {
      this.$confirm("确定库存信息?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "error"
      }).then(() => {
        //发送请求
        this.$http({
          url: this.$http.adornApiUrl("/wms/waresku/delete"),
          method: "post",
          data: this.$http.adornData([row.id], false)
        }).then(({ data }) => {
          this.$message({
            type: "success",
            message: "库存删除存成功"
          });
          this.editSku(row);
        });
      });
    },
    cancelStockInfo(row) {
      //this.getSkusBySpuId(this.spuId);
      row.showStock = false;
      row.showWare = false;
      var orow = JSON.parse(this.oldRow);
      row.stock = orow.stock;
      row.wareId = orow.wareId;
      row.wareName = orow.wareName;
    },
    editSku(row) {
      this.currentSku.skuId = row.skuId;
      this.currentSku.skuName = row.skuName;
      console.log("数据...", row);
      this.stockDialogVisible = true;
      //获取当前sku对应的所有库存
      this.$http({
        url: this.$http.adornApiUrl("/wms/waresku/sku/" + row.skuId),
        method: "get"
      }).then(({ data }) => {
        this.stockTableData = data.data;
        for (var i = 0; i < this.stockTableData.length; i++) {
          this.stockTableData[i].showStock = true;
          this.stockTableData[i].showWare = true;
          this.stockTableData[i].wareName = this.getRealWareName(
            this.stockTableData[i].wareId
          );
        }
      });
    },
    getRealWareName(id) {
      for (var i = 0; i < this.wares.length; i++) {
        if (this.wares[i].id == id) {
          return this.wares[i].name;
        }
      }
    },
    showWareName(row) {
      console.log("showWareName...", row);
      row.wareName = this.getRealWareName(row.wareId);
    },
    saveStockInfo(row) {
      this.$confirm("是否保存库存信息?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        //发送请求
        this.$http({
          url: this.$http.adornApiUrl("/wms/waresku/save"),
          method: "post",
          data: this.$http.adornData(row, false)
        }).then(({ data }) => {
          this.$message({
            type: "success",
            message: "库存保存成功"
          });
          this.editSku(row);
        });
      });
    },
    //
    objectSpanMethod({ row, column, rowIndex, columnIndex }) {},
    getSkusBySpuId(id) {
      this.tableData = [];
      return new Promise((resolve, reject) => {
        this.$http({
          url: this.$http.adornApiUrl("/pms/skuinfo/list/spu/" + id),
          method: "get"
        }).then(({ data }) => {
          this.tableData = data.data;
        });
      });
    },
    getWares() {
      this.$http({
        url: this.$http.adornApiUrl("/wms/wareinfo/list"),
        method: "get",
        params: this.$http.adornParams({
          limit: 1000
        })
      }).then(({ data }) => {
        this.wares = data.data.list;
      });
    }
  },
  created() {},
  mounted() {
    this.getWares();
  }
};
</script>
<style lang="scss" scoped>
.wrapper {
}
</style>