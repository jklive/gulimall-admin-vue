<template>
  <div class>
    <div>
      <el-row>
        <el-table :data="attrGroupTableData" height="350">
          <el-table-column label="属性分组">
            <el-table-column type="selection" align="center"></el-table-column>
            <el-table-column type="index" label="序号"></el-table-column>
            <el-table-column prop="attrGroupId" label="分组ID"></el-table-column>
            <el-table-column prop="attrGroupName" label="分组名"></el-table-column>
            <el-table-column prop="icon" label="组图标">
              <template slot-scope="scope">
                <el-image style="width: 64px; height: 24px" :src="scope.row.icon" fit="contain"></el-image>
              </template>
            </el-table-column>
            <el-table-column prop="descript" label="描述"></el-table-column>
            <el-table-column header-align="center" align="center" width="250" label="操作">
              <template slot-scope="scope">
                <el-button type="text" size="small" @click="attrGroupRelationHandle(scope.row.attrGroupId)">维护关联</el-button>
                <el-button type="text" size="small" @click="attrGroupAddHandle(scope.row.attrGroupId)">修改</el-button>
                <el-button
                  type="text"
                  size="small"
                  @click="attrGroupDeleteHandle(scope.row.attrGroupId,scope.row.attrGroupName)"
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
    </div>
    <attr-group-add-update v-if="attrGroupDialogShow" ref="attrGroupAddUpdate" @saveSuccess="saveSuccess"></attr-group-add-update>
    <attr-group-relation v-if="attrGroupRelationDialogShow" ref="attrGroupRelation"></attr-group-relation>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import AttrGroupAddUpdate from "./attr-group-add-update";
import AttrGroupRelation from "./attr-group-relation";
export default {
  //import引入的组件需要注入到对象中才能使用
  components: { AttrGroupAddUpdate,AttrGroupRelation },
  props: ["catId"],
  data() {
    //这里存放数据
    return {
      attrGroupTableData: [],
      pageIndex: 1,
      pageSize: 10,
      totalCount: 0,
      attrGroupDialogShow: false,
      attrGroupRelationDialogShow: false
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {
    catId(val) {
      this.initAttrGroup(val);
    }
  },
  //方法集合
  methods: {
    saveSuccess(){
      this.initAttrGroup(this.catId);
    },
    // 初始化屬性分組
    initAttrGroup(catId) {
      this.$nextTick(() => {
        this.$http({
          url: this.$http.adornApiUrl("/pms/attrgroup/list/category/" + catId),
          method: "get",
          params: this.$http.adornParams({
            limit: this.pageSize,
            page: this.pageIndex
          })
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.attrGroupTableData = data.data.list;
            this.totalCount = data.data.totalCount;
            this.pageIndex = data.data.currPage;
          }
        });
      });
    },
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.initAttrGroup(this.catId);
    },
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.initAttrGroup(this.catId);
    },
    attrGroupDeleteHandle(val, name) {
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
            url: this.$http.adornApiUrl("/pms/attrgroup/delete"),
            method: "post",
            data: this.$http.adornData([val], false, "json")
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({ type: "success", message: "删除成功..." });
              this.initAttrGroup(this.catId);
            } else {
            }
          });
        })
        .catch(() => {
          this.$message({ type: "info", message: "已取消" });
        });
    },
    attrGroupAddHandle(id) {
      //按照属性分组id查出属性分组信息，进行回显并修改
      this.$http({
        url: this.$http.adornApiUrl("/pms/attrgroup/info/" + id),
        method: "get"
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.attrGroupDialogShow = true;
          this.$nextTick(() => {
            this.$refs.attrGroupAddUpdate.showDetails(data.data);
          });
        } else {
        }
      });
    },
    attrGroupRelationHandle(id){
        this.attrGroupRelationDialogShow = true;
        this.$nextTick(()=>{
          this.$refs.attrGroupRelation.init(id);
        });
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