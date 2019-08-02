<!--  -->
<template>
  <div class>
    <el-dialog :title="'【'+attrGroupName+'】属性组关联'" :visible.sync="dialogVisible">
      <el-table :data="tableData" style="width: 100%">
        <el-table-column prop="attrId" label="属性id" width="150"></el-table-column>
        <el-table-column label="属性信息" width="400">
          <template slot-scope="scope">{{getAttrInfo(scope.row.attrId)}}</template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button size="mini" type="danger" @click="removeRelation(scope.row.attrId)">删除关联</el-button>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
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
      dialogVisible: false,
      tableData: [],
      attrGroupId: 0,
      attrGroupName: "",
      attrs: []
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    init(id) {
      this.attrGroupId = id;
      //传入分组id
      this.dialogVisible = true;
      //获取当前分组和子属性详情
      this.$http({
        url: this.$http.adornApiUrl(
          "/pms/attrgroup/info/withattrs/" + this.attrGroupId
        ),
        method: "get"
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.attrGroupName = data.data.attrGroupName;
          this.tableData = data.data.relations;
          this.attrs = data.data.attrEntities;
        } else {
        }
      });
    },
    getAttrInfo(id) {
      for (var i = 0; i < this.attrs.length; i++) {
        if (this.attrs[i].attrId == id) {
          return this.attrs[i].attrName + " ：" + this.attrs[i].valueSelect;
        }
      }
      return "此属性已经失去关联，请及时删除";
    },
    removeRelation(id) {
      var post = { attrId: id, attrGroupId: this.attrGroupId };
      this.$http({
          url: this.$http.adornApiUrl("/pms/attrattrgrouprelation/delete/attr"),
          method: "post",
          data: this.$http.adornData([post], false, "json")
        }).then(({ data }) => {
          if (data && data.code === 0) {
              this.$message({
                  type: "success",
                  message: "删除成功"
              });
              this.init(post.attrGroupId);
          } else {
          }
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