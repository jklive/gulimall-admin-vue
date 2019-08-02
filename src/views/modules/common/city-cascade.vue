<template>
  <div class="wrapper">
      <!-- 省市区三级联动选择 -->
      <el-cascader :props="props"></el-cascader>
  </div>
</template>

<script>
export default {
  components:{},
  props:{},
  data(){
    return {
        props: {
          lazy: true,
          lazyLoad (node, resolve) {
              
            const { level } = node;
            console.log("省市区等级",level)
            resolve(nodes);
          }
        }
    }
  },
  watch:{},
  computed:{},
  methods:{},
  created(){},
  mounted(){}
}
</script>
<style lang="scss" scoped>
.wrapper{}
</style>