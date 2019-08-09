<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="商品id,商品名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList(0)">查本类</el-button>
        <el-button @click="getDataList(1)">查全站</el-button>
        <!-- utils里面index.js判断权限，将登录后的所有权限都保存在了sessionStorage中 v-if="isAuth('pms:spuinfo:save')"  v-if="isAuth('pms:spuinfo:delete')" -->
        <el-button type="primary" @click="addSpuStep()">添加spu</el-button>
        <el-dropdown type="primary" trigger="click" @command="handleCommand">
          <el-button type="warning">
            批量操作
            <i class="el-icon-arrow-down el-icon--right"></i>
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="deleteHandle" type="danger">批量删除</el-dropdown-item>
            <el-dropdown-item command="batchUp">批量上架</el-dropdown-item>
            <el-dropdown-item command="batchDwon">批量下架</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
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
      <el-table-column prop="publishStatus" header-align="center" align="center" label="上架状态">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.publishStatus"
            :active-value="1"
            :inactive-value="0"
            @change="updateSpuInfo(scope.row.id,$event)"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" header-align="center" align="center" label="创建时间"></el-table-column>
      <el-table-column prop="uodateTime" header-align="center" align="center" label="更新时间"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
  </div>
</template>

<script>
import AddOrUpdate from "./spuinfo-add-or-update";
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
  props: ["catId"],
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList();
  },
  methods: {
    addSpuStep() {
      this.$router.push({
        name: `spu-add-step`, // 只是把query改了，其他都没变
        query: {
          catId: this.catId
        }
      });
    },
    handleCommand(cmd) {
      console.log(this.dataListSelections);
      var dataIds = [];
      for (var i = 0; i < this.dataListSelections.length; i++) {
        dataIds.push(this.dataListSelections[i].id);
      }
      if (dataIds.length <= 0) {
        this.$message("请先选择需要操作的商品...");
        return true;
      }
      if ("deleteHandle" == cmd) {
        //批量删除
        this.deleteHandle();
      }
      if ("batchUp" == cmd) {
        //批量上架
        ///pms/spuinfo/batch/updateStatus
        this.$http({
          url: this.$http.adornApiUrl("/pms/spuinfo/batch/updateStatus"),
          method: "post",
          data: this.$http.adornData({ spuIds: dataIds, status: 1 }, false)
        }).then(({ data }) => {
          this.$message("上架成功....");
        });
      }
      if ("batchDown" == cmd) {
        //批量下架
        this.$http({
          url: this.$http.adornApiUrl("/pms/spuinfo/batch/updateStatus"),
          method: "post",
          data: this.$http.adornData({ spuIds: dataIds, status: 0 }, false)
        }).then(({ data }) => {
          this.$message("下架成功....");
        });
      }
    },
    batchUpdateSpuInfo(val) {
      //this.updateSpuInfo()
      console.log(this.dataListSelections);
    },
    updateSpuInfo(id, val) {
      this.$http({
        url: this.$http.adornApiUrl("/pms/spuinfo/updateStatus/" + id),
        method: "get",
        params: this.$http.adornParams({ status: val })
      }).then(({ data }) => {
        if (data.code == 0) {
          this.$message("上架成功");
        }
      });
    },
    // 获取数据列表
    getDataList(all) {
      this.all = all;
      console.log("要去查询的值", this.catId);
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornApiUrl("/pms/spuinfo/simple/search"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          catId: this.all == 0 ? this.catId : 0
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
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
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
          url: this.$http.adornUrl("/pms/spuinfo/delete"),
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
