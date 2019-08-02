<!--  -->
<template>
  <div>
    <el-row>
      <el-table :data="tableData" height="350">
        <el-table-column label="销售属性">
          <el-table-column type="selection" align="center"></el-table-column>
          <el-table-column type="index" label="序号"></el-table-column>
          <el-table-column prop="attrId" label="属性ID"></el-table-column>
          <el-table-column prop="attrName" label="属性名"></el-table-column>
          <!-- 都是多值，sku要选择不同组合值 -->
          <!-- <el-table-column prop="valueType" label="值类型"></el-table-column> -->
          <el-table-column prop="icon" label="属性图标">
            <template slot-scope="scope">
              <el-image style="width: 24px; height: 24px" :src="scope.row.icon" fit="contain"></el-image>
            </template>
          </el-table-column>
          <el-table-column prop="valueSelect" label="属性值"></el-table-column>
          <!-- <el-table-column prop="attrType" label="属性类型"></el-table-column> -->
          <!-- <el-table-column prop="catelogId" label="所属分类"></el-table-column> -->
          <el-table-column prop="enable" label="启用状态">
            <template slot-scope="scope">
              <el-switch
                v-model="scope.row.enable"
                active-color="#13ce66"
                inactive-color="#ff4949"
                :active-value="1"
                :inactive-value="0"
                disabled
              ></el-switch>
            </template>
          </el-table-column>
          <!-- <el-table-column prop="showDesc" label="快速展示"></el-table-column> -->
          <el-table-column header-align="center" align="center" width="150" label="操作">
            <template slot-scope="scope">
              <el-button type="text" size="small" @click="addSaleAttrHandle(scope.row.attrId)">修改</el-button>
              <el-button
                type="text"
                size="small"
                @click="deleteSaleAttrHandle(scope.row.attrId,scope.row.attrName)"
              >删除</el-button>
            </template>
          </el-table-column>
        </el-table-column>
      </el-table>
    </el-row>
    <el-row>
      <div class="block">
        <el-pagination
          @size-change="sizeChangeHandle"
          @current-change="currentChangeHandle"
          :current-page="pageIndex"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="totalCount"
        ></el-pagination>
      </div>
    </el-row>
    <attribute-add-update :cat-id="catId" v-if="attrAddUpdateShow" ref="attrAddUpdate" @saveSuccess="saveSuccess"></attribute-add-update>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import AttributeAddUpdate from "./attribute-add-update";
export default {
  //import引入的组件需要注入到对象中才能使用
  props: ["catId"],
  components: { AttributeAddUpdate },
  data() {
    //这里存放数据
    return {
      tableData: [],
      totalCount: 0,
      pageIndex: 1,
      pageSize: 10,
      attrAddUpdateShow: false
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {
    catId(val) {
      this.getDataList();
    }
  },
  //方法集合
  methods: {
    saveSuccess(){
      this.getDataList();
    },
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.getDataList();
    },
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    getDataList() {
      console.log("sale-attr    getDataList")
      //
      this.$http({
        url: this.$http.adornApiUrl("/pms/attr/sale/" + this.catId),
        method: "get",
        params: this.$http.adornParams({
          limit: this.pageSize,
          page: this.pageIndex
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.tableData = data.data.list;
          this.totalCount = data.data.totalCount;
          this.pageIndex = data.data.currPage;
        } else {
        }
      });
    },
    deleteSaleAttrHandle(id, name) {
      this.$confirm("确认删除属性【" + name + "】，是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornApiUrl("/pms/attr/delete"),
            method: "post",
            data: this.$http.adornData([id], false, "json")
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({ type: "success", message: "删除成功..." });
              this.getDataList();
            } else {
            }
          });
        })
        .catch(() => {
          this.$message({ type: "info", message: "已取消" });
        });
    },
    addSaleAttrHandle(id) {
      console.log("1", id);
      this.attrAddUpdateShow = true;

      //1、查出属性，打开模态框进行回显；
      this.$http({
        url: this.$http.adornApiUrl("/pms/attr/info/" + id),
        method: "get"
      }).then(({data}) => {
          if (data && data.code === 0) {
            // data.data;
            this.$nextTick(()=>{
              //还要查询这个属性属于哪一组   
                this.$refs.attrAddUpdate.showDetails(data.data);
            })
            
          } else {
          }
      }).catch((err) => {
          console.log("说道数据不对...",err);
        });

      //1、查出当前属性id的详细信息
      //2、回显数据
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