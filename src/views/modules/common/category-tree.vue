<template>
  <div>
    <ul class="ztree"></ul>
  </div>
</template>
<script>
export default {
  data() {
    return {
      categoryTreeData: "",
      categoryTreeObj: "",
      treeNodes:[],
      settings:{
        data: {
          simpleData: {
            enable: true,
            idKey: "catId",
            pIdKey: "parentCid",
            rootPId: 0
          }
        },
        callback: {
          onClick: this.zTreeOnClick
        }
      }
      // cachedData: ""
    };
  },
  computed: {},
  mounted() {
    this.initCategoryList();
  },
  methods: {
    //初始化商品
    initCategoryList() {
      //将三级分类整个方法作为Promise方法，别人需要感知的时候，可以方便的等待方法运行完成
     return new Promise((resolve, reject) => {
        this.$http({
          url: this.$http.adornApiUrl("/pms/category/list/tree"),
          method: "get",
          params: this.$http.adornParams({
            level: 0
          })
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.categoryTreeData = data.data;
            //渲染树数据
            this.initZtree();
            resolve(data);
          } else {
            reject(data);
          }
        });
      });
    },
    //初始化ztree对象,ztree被点击的时候给父组件传递被选中的对象。
    initZtree() {
      // this.categoryTreeData.push({catId:0,name:"系统分类"});
      this.categoryTreeObj = $.fn.zTree.init(
        $("ul.ztree"),
        this.settings,
        this.categoryTreeData
      );
    },
    zTreeOnClick(event, treeId, treeNode) {
      //向父组件传递刚才选中的treeNode值，如果要传递多个值，就依次写更多变量即可
      this.$emit("treeNode", treeNode, this.categoryTreeObj);
    }
  }
};
</script>
<style scoped>
</style>
