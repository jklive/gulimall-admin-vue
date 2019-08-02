<template>
  <div class="wrapper">
    <el-dialog title="sku维护" :visible.sync="dialogVisible">
      <el-table :data="tableData" style="width: 100%">
        <el-table-column label="skuId" prop="skuId"></el-table-column>
        <el-table-column label="sku名字" prop="skuName"></el-table-column>
        <el-table-column label="sku标题" prop="skuTitle"></el-table-column>
        <el-table-column label="sku价格" prop="price"></el-table-column>
        <!-- <el-table-column label="库存信息">
          <el-table-column label="库存" prop="stock">
            <template slot-scope="scope">
              <span v-if="!scope.row.showStock">{{scope.row.stock}}</span>
              <el-input v-if="scope.row.showStock" v-model="scope.row.stock" placeholder="请输入内容"></el-input>
            </template>
          </el-table-column>
          <el-table-column label="所在仓库" prop="wareId">
            <template slot-scope="scope">
              <span v-if="!scope.row.showWare">{{scope.row.wareName}}</span>
              <el-select
                v-if="scope.row.showWare"
                v-model="scope.row.wareId"
                placeholder="请选择"
                @change="showWareName(scope.row)"
              >
                <el-option v-for="item in wares" :key="item.id" :label="item.name" :value="item.id"></el-option>
              </el-select>
            </template>
          </el-table-column>
        </el-table-column>-->
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
      <el-table :data="stockTableData">
        <el-table-column label="#" prop="id"></el-table-column>
        <el-table-column label="skuId" prop="skuId"></el-table-column>
        <el-table-column label="商品名" prop="skuName"></el-table-column>
        <el-table-column label="库存" prop="stock">
          <template slot-scope="scope">
            <span v-if="!scope.row.showStock">{{scope.row.stock}}</span>
            <el-input v-if="scope.row.showStock" v-model="scope.row.stock" placeholder="请输入内容"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="所在仓库" prop="wareId">
          <template slot-scope="scope">
            <span v-if="!scope.row.showWare">{{scope.row.wareName}}</span>
            <el-select
              v-if="scope.row.showWare"
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
            <el-button size="mini" type="text">修改</el-button>
            <el-button size="mini" type="text">保存</el-button>
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
      oldRow: {}
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
      console.log("数据...", row);
      this.stockDialogVisible = true;
      //获取当前sku对应的所有库存
      this.$http({
        url: this.$http.adornApiUrl("/wms/waresku/sku/" + row.skuId),
        method: "get"
      }).then(({ data }) => {
        this.stockTableData = data.data;
        // data.data
        for (var i = 0; i < this.stockTableData.length; i++) {
          this.stockTableData[i].showStock = false;
          this.stockTableData[i].showWare = false;
          this.stockTableData[i].wareName = this.getRealWareName(this.stockTableData[i].wareId);
        }
        console.log("库存===>", this.stockTableData);
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
      row.wareName = this.getRealWareName(row.wareId);
    },
    saveStockInfo(data) {
      console.log("点击保存了》。。");
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

          // for (let i = 0; i < dataList.length; i++) {
          //   //dataList[i] = "";//查出每个sku的库存记录
          //   //scope.row.showStock = true;scope.row.showWare = true
          //   dataList[i].showStock = false;
          //   dataList[i].showWare = false;
          //   this.$http({
          //     url: this.$http.adornApiUrl(
          //       "/wms/waresku/sku/" + dataList[i].skuId
          //     ),
          //     method: "get"
          //   }).then(({ data }) => {
          //     //获取每个的库存信息
          //     let skuStocks = data.data;
          //     console.log("skuStocks==>",skuStocks);
          //     for(let index = 0;index < skuStocks.length;index++){
          //         item.wareName = this.getRealWareName(skuStocks[index].wareId);
          //         item.wareId = skuStocks[index].wareId;
          //         item.stock = skuStocks[index].stock;
          //         item.wareStockId =  skuStocks[index].id;
          //         console.log("skuStocks==>",skuStocks);
          //         this.tableData.push(item);
          //     }
          //   });

          // }
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