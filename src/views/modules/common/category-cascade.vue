<!--  -->
<template>
  <el-cascader :props="props" size="medium" clearable v-model="id" @change="change"></el-cascader>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    let slef = this;
    //这里存放数据
    return {
      id: 1,
      props: {
        lazy: true,
        value: "catId",
        label: "name",
        lazyLoad(node, resolve) {
          const { level } = node;
          if (level == 0) {
            slef.rootCatagory().then(data => {
              resolve(data);
            });
          }
          //(level <= 2) &&
          if (level > 0) {
            console.log(node);
            if (level == 3) {
              resolve(null);
            } else {
              slef.childCatagory(node.value).then(data => {
                resolve(data);
              });
            }
          }
        }
      }
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    rootCatagory() {
      return new Promise((resolve, reject) => {
        this.$http({
          url: this.$http.adornApiUrl("/pms/category/list/tree"),
          method: "get",
          params: this.$http.adornParams({
            level: 1
          })
        }).then(({ data }) => {
          if (data && data.code === 0) {
            resolve(data.data);
          } else {
            reject(data.data);
          }
        });
      });
    },
    childCatagory(id) {
      return new Promise((resolve, reject) => {
        this.$http({
          url: this.$http.adornApiUrl("/pms/category/list/children/tree/" + id),
          method: "get",
          params: this.$http.adornParams({})
        }).then(({ data }) => {
          if (data && data.code === 0) {
            resolve(data.data);
          } else {
            reject(data.data);
          }
        });
      });
    },
    change(val){
        this.$emit("selectedValue",val[val.length-1])
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