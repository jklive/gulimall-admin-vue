<!--  -->
<template>
  <div>
    <el-row>
      <el-table :data="tableData" height="350">
        <el-table-column label="基本属性">
          <el-table-column type="selection" align="center"></el-table-column>
          <el-table-column type="index" label="序号"></el-table-column>
          <el-table-column prop="attrId" label="属性ID"></el-table-column>
          <el-table-column prop="attrName" label="属性名"></el-table-column>
          <el-table-column prop="searchType" label="是否检索">
            <template slot-scope="scope">
              <el-switch
                v-model="scope.row.searchType"
                active-color="#13ce66"
                inactive-color="#ff4949"
                :active-value="1"
                :inactive-value="0"
                disabled
              ></el-switch>
            </template>
          </el-table-column>
          <el-table-column prop="valueType" label="值类型">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.valueType==0" type="primary">选择单值</el-tag>
              <el-tag v-if="scope.row.valueType==1" type="success">选择多值</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="icon" label="属性图标">
            <template slot-scope="scope">
              <el-image style="width: 24px; height: 24px" :src="scope.row.icon" fit="contain"></el-image>
            </template>
          </el-table-column>
          <el-table-column prop="valueSelect" label="属性值"></el-table-column>
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
          <el-table-column prop="showDesc" label="快速展示">
            <template slot-scope="scope">
              <el-switch
                v-model="scope.row.showDesc"
                active-color="#13ce66"
                inactive-color="#ff4949"
                :active-value="1"
                :inactive-value="0"
                disabled
              ></el-switch>
            </template>
          </el-table-column>
          <el-table-column header-align="center" align="center" width="150" label="操作">
            <template slot-scope="scope">
              <el-button type="text" size="small" @click="addAttributeHandle(scope.row.attrId)">修改</el-button>
              <el-button
                type="text"
                size="small"
                @click="deleteAttributeHandle(scope.row.attrId,scope.row.attrName)"
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
      pageIndex: 1,
      pageSize: 10,
      totalCount: 0,
      tableData: [],
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
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.getDataList();
    },
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    getDataList() {
      console.log("===刷新列表了....");
      this.$http({
        url: this.$http.adornApiUrl("/pms/attr/base/" + this.catId),
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
    deleteAttributeHandle(id, name) {
      this.$confirm(
        "确认删除分组【" +
          name +
          "】,这个组关联的属性将自动归为【其他】组，是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      )
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
    addAttributeHandle(id) {
      this.attrAddUpdateShow = true;
      //1、查出属性，打开模态框进行回显；
      this.$http({
        url: this.$http.adornApiUrl("/pms/attr/info/" + id),
        method: "get"
      }).then(({ data }) => {
        console.log("基础数据....",data)
        this.$nextTick(() => {
          if (data && data.code === 0) {
            this.$refs.attrAddUpdate.showDetails(data.data);
          } else {
          }
        });
      });
      //1、查出当前属性id的详细信息
      //2、回显数据
    },
    saveSuccess() {
      this.getDataList();
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