<template>
  <el-select
    v-model="value"
    filterable
    remote
    reserve-keyword
    placeholder="请输入关键词"
    :remote-method="remoteMethod"
    :loading="loading"
    @change = "change"
  >
    <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"></el-option>
  </el-select>
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
      value: null,
      options: [],
      loading: false,
      states: []
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    remoteMethod(query) {
      if (query !== "") {
        this.loading = true;
        this.$http({
          url: this.$http.adornApiUrl("/pms/brand/list"),
          method: "get",
          params: this.$http.adornParams({
            key: query
          })
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.options = data.data.list.map(item => {
              console.log("map item", item);
              return { value: item.brandId, label: item.name };
            });
            this.loading = false;
          } else {
          }
        });
      } else {
        this.options = []
      }
    },
    change(val){
        this.$emit("selectedVal",val);
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {},
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.$http({
      url: this.$http.adornApiUrl("/pms/brand/list"),
      method: "get",
      params: this.$http.adornParams({})
    }).then(({ data }) => {
      if (data && data.code === 0) {
        this.states = data.data.list;
        this.options = this.states.map(item => {
          console.log("map item", item);
          return { value: item.brandId, label: item.name };
        });
      } else {
      }
    });
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