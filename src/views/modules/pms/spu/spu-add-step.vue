<!--  -->
<template>
  <div>
    <el-steps :active="spuStep" finish-status="success" simple>
      <el-step title="录入spu基本信息" icon="el-icon-info"></el-step>
      <el-step title="录入spu属性信息" icon="el-icon-edit"></el-step>
      <el-step title="录入sku相关信息" icon="el-icon-share"></el-step>
    </el-steps>
    <div v-if="spuStep==0" style="width:450px;margin-left:30%;padding:20px">
      <!-- -->
      <el-form ref="spuInfoform" :model="spuInfo" label-width="80px" :rules="spuInfoRules">
        <el-form-item label="商品名称" prop="spuName">
          <el-input v-model="spuInfo.spuName"></el-input>
        </el-form-item>
        <el-form-item label="商品描述" prop="spuDescription">
          <el-input v-model="spuInfo.spuDescription"></el-input>
        </el-form-item>
        <el-form-item label="选择分类" prop="catalogId">
          <!-- <el-cascader :props="props" size="medium" clearable v-model=""></el-cascader> -->
          <category-cascade @selectedValue="selectedValue"></category-cascade>
        </el-form-item>
        <el-form-item label="品牌" prop="brandId">
          <brand-select @selectedVal="brandSelectedVal"></brand-select>
        </el-form-item>
        <el-form-item label="上架" prop="publishStatus">
          <el-switch
            v-model="spuInfo.publishStatus"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
          ></el-switch>
        </el-form-item>
        <el-form-item label="商品介绍图片集" prop="images">
          <br />
          <!-- <el-input v-model="spuInfo.images"></el-input> -->
          <multi-upload :value="spuInfo.images" @input="emitInput"></multi-upload>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" style="margin-top: 12px;" @click="next">{{btnText}}</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div v-if="spuStep==1" style="width:300px;margin-left:30%;padding:20px">
      <el-form ref="spuInfoform" :model="spuInfo" label-width="80px">
        <!-- 当前分类下所有属性分组遍历 -->
        <el-form-item
          :label="attrGroup.attrGroupName"
          v-for="(attrGroup,i)  in attrGroups"
          :key="i"
        >
          <!-- 获取每个分组的所有属性，遍历的是外面的，选中的是form里面绑定值得 -->
          <!-- 属性有很多值 -->

          <el-form-item
            v-for="(attr,index) in attrGroup.attrEntities"
            :key="index"
            :label="attr.attrName"
          >
            <el-checkbox-group 
              size="small"
              v-model="finalValues[i][index].valueSelected"
              @change="handleCheckedAttrChange"
            >
              <el-checkbox
                border
                v-for="(attrItem,num) in attr.valueSelect"
                :key="index+'-'+num+''+i"
                :label="attrItem"
              >{{attrItem}}</el-checkbox>
              <!-- -->
            </el-checkbox-group>
<!-- 
            <el-radio-group
              v-model="finalValues[i][index].valueSelected"
              v-if="attr.valueType == 0"
            >
              <el-radio
                v-for="(attrItem,num) in attr.valueSelect"
                :key="index+'-'+num+''+i"
                :label="attrItem"
              >{{attrItem}}</el-radio>
            </el-radio-group> -->
          </el-form-item>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" size="mini" style="margin-top: 12px;" @click="prev">上一步</el-button>
          <el-button type="primary" size="mini" style="margin-top: 12px;" @click="next">{{btnText}}</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div v-if="spuStep==2" style="width:650px;margin-left:30%;padding:20px">
      <!-- 1:颜色  ： 21黑，22白，23粉
      2：内存：   81:8g,82:10g,88:21g-->
      <!-- [{attrId:1,value:[白,黑]}，,{attrId:2,value:[8,10]}] -->
      <el-form ref="saleGroupForm" label-width="80px" :v-model="saleGroupForm">
        <el-form-item v-for="(sale,index) in saleGroups" :key="index" :label="sale.attrName">
          <el-checkbox-group v-model="saleGroupValue[index].valueSelect" :key="index">
            <el-checkbox
              v-for="(saleItem,i) in sale.valueSelect"
              :label="saleItem"
              :key="i"
            >{{saleItem}}</el-checkbox>
          </el-checkbox-group>
        </el-form-item>
      </el-form>
      <!--[{25:18,30:5g,},{}]  -->
      <el-table :data="tableFinalData">
        <el-table-column
          v-for="(sale,index) in saleDataTable"
          :key="index"
          :label="sale.attrName"
          :prop="'attr_'+sale.attrId"
        ></el-table-column>
        <el-table-column label="价格" v-if="saleDataTable.length>0" prop="price">
          <template slot-scope="scope">
            <el-input v-model="scope.row.price" placeholder="价格"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="设置详情" v-if="saleDataTable.length>0" type="expand">
          <template slot-scope="scope">
            <!-- 设置基本信息 -->
            <el-card class="box-card" shadow="hover">
              <div slot="header" class="clearfix">
                <span>sku基本信息</span>
              </div>
              <el-form label-position="left" class="demo-table-expand">
                <el-form-item>
                  <el-input v-model="scope.row.skuName">
                    <template slot="prepend">名字</template>
                  </el-input>
                </el-form-item>
                <el-form-item>
                  <el-input v-model="scope.row.skuDesc">
                    <template slot="prepend">描述</template>
                  </el-input>
                </el-form-item>
                <el-form-item>
                  <el-input v-model="scope.row.skuTitle">
                    <template slot="prepend">标题</template>
                  </el-input>
                </el-form-item>
                <el-form-item>
                  <el-input v-model="scope.row.skuSubtitle">
                    <template slot="prepend">副标题</template>
                  </el-input>
                </el-form-item>
                <el-form-item>
                  <el-input v-model="scope.row.weight">
                    <template slot="prepend">重量</template>
                  </el-input>
                </el-form-item>
              </el-form>
            </el-card>
            <!-- 设置积分 -->
            <el-card class="box-card" shadow="hover">
              <div slot="header" class="clearfix">
                <span>设置积分</span>
              </div>
              <el-form label-position="left" class="demo-table-expand">
                <el-form-item>
                  <el-input v-model="scope.row.growBounds" placeholder="成长积分">
                    <template slot="prepend">赠送</template>
                    <template slot="append">成长积分</template>
                  </el-input>
                </el-form-item>
                <el-form-item>
                  <el-input v-model="scope.row.buyBounds" placeholder="购物积分">
                    <template slot="prepend">赠送</template>
                    <template slot="append">购物积分</template>
                  </el-input>
                </el-form-item>
                <el-form-item label="生效情况">
                  <br />
                  <el-form-item label="无优惠，成长积分是否生效">
                    <el-switch v-model="scope.row.work[0]" :active-value="1" :inactive-value="0"></el-switch>
                  </el-form-item>
                  <el-form-item label="无优惠，购物积分是否生效">
                    <el-switch v-model="scope.row.work[1]" :active-value="1" :inactive-value="0"></el-switch>
                  </el-form-item>
                  <el-form-item label="有优惠，成长积分是否生效">
                    <el-switch v-model="scope.row.work[2]" :active-value="1" :inactive-value="0"></el-switch>
                  </el-form-item>
                  <el-form-item label="有优惠，购物积分是否生效">
                    <el-switch v-model="scope.row.work[3]" :active-value="1" :inactive-value="0"></el-switch>
                  </el-form-item>
                </el-form-item>
              </el-form>
            </el-card>

            <!-- 设置打折 -->
            <el-card class="box-card" shadow="hover">
              <div slot="header" class="clearfix">
                <span>设置打折</span>
              </div>
              <el-form label-position="left" class="demo-table-expand">
                <el-form-item>
                  <el-input v-model="scope.row.fullCount">
                    <template slot="prepend">满</template>
                    <template slot="append">件</template>
                  </el-input>
                </el-form-item>
                <el-form-item>
                  <el-input v-model="scope.row.discount">
                    <template slot="prepend">打</template>
                    <template slot="append">% 折</template>
                  </el-input>
                </el-form-item>
                <el-form-item label="可否与其他优惠叠加">
                  <el-switch
                    v-model="scope.row.ladderAddOther"
                    :active-value="1"
                    :inactive-value="0"
                  ></el-switch>
                </el-form-item>
              </el-form>
            </el-card>

            <!-- 设置满减 -->
            <el-card class="box-card" shadow="hover">
              <div slot="header" class="clearfix">
                <span>设置满减</span>
              </div>
              <el-form label-position="left" class="demo-table-expand">
                <el-form-item>
                  <el-input v-model="scope.row.fullPrice" placeholder>
                    <template slot="prepend">满</template>
                    <template slot="append">元</template>
                  </el-input>
                </el-form-item>
                <el-form-item>
                  <el-input v-model="scope.row.reducePrice">
                    <template slot="prepend">减</template>
                    <template slot="append">元</template>
                  </el-input>
                </el-form-item>
                <el-form-item label="可否与其他优惠叠加">
                  <el-switch v-model="scope.row.fullAddOther" :active-value="1" :inactive-value="0"></el-switch>
                </el-form-item>
              </el-form>
            </el-card>

            <!-- 上传图片 -->
            <el-card class="box-card" shadow="hover">
              <div slot="header" class="clearfix">
                <span>上传图片</span>
                <!-- vue，想要回调的时候额外用到自己的数据，和人家原生默认的，只需要 $event就能引用到 -->
                <multi-upload :value="scope.row.images" @input="skuUploadImage(scope.row,$event)"></multi-upload>
              </div>
            </el-card>
          </template>
        </el-table-column>
      </el-table>

      <el-button type="primary" size="mini" style="margin-top: 12px;" @click="prev">上一步</el-button>
      <el-button type="primary" size="mini" style="margin-top: 12px;" @click="next">{{btnText}}</el-button>
    </div>
    <div v-if="spuStep==3" style="width:450px;margin-left:30%;padding:20px">
      在这里写第四屏的信息
      <el-button type="primary" size="mini" style="margin-top: 12px;" @click="prev">上一步</el-button>
      <el-button type="primary" size="mini" style="margin-top: 12px;" @click="next">{{btnText}}</el-button>
    </div>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import CategoryCascade from "../../common/category-cascade";
import BrandSelect from "../../common/brand-select";
import MultiUpload from "@/components/upload/multiUpload";
export default {
  //import引入的组件需要注入到对象中才能使用
  components: { CategoryCascade, BrandSelect, MultiUpload },
  data() {
    let slef = this;
    //这里存放数据
    return {
      spuStep: 0,
      btnText: "下一步",
      finalValues: [[]],
      saleGroups: [],
      saleGroupValue: [],
      spuInfo: {
        spuName: "",
        spuDescription: "",
        catalogId: 0,
        brandId: 0,
        publishStatus: 0,
        images: [],
        attrGroups: [],
        saleAttrs: []
      },
      saleGroupForm: {},
      attrGroups: [],
      saleDataTable: [],
      tableFinalData: [],
      spuInfoRules: {
        spuName: [
          { required: true, message: "请添加商品名字", trigger: "blur" }
        ],
        spuDescription: [
          { required: true, message: "请添加商品描述", trigger: "blur" }
        ],
        catalogId: [
          { required: true, message: "请选择商品分类", trigger: "change" }
        ],
        brandId: [
          { required: true, message: "请选择商品品牌", trigger: "change" }
        ],
        images: [
          {
            type: "array",
            required: true,
            message: "请至少上传一张图片",
            trigger: "change"
          }
        ]
      },
      categorys: []
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {
    saleGroupValue: {
      handler(newValue, oldValue) {
        console.log("new==>" + newValue + "====>" + oldValue);
        this.setTableData(newValue);
      },
      deep: true
    }
  },
  //方法集合
  methods: {
    //计算笛卡尔积
    calcDescartes(array) {
      if (array.length < 2) return array[0] || [];
      return [].reduce.call(array, function(col, set) {
        var res = [];
        col.forEach(function(c) {
          set.forEach(function(s) {
            var t = [].concat(Array.isArray(c) ? c : [c]);
            t.push(s);
            res.push(t);
          });
        });
        return res;
      });
    },
    // 选中结果排列组合 生成的最终table值saleDataTable
    setTableData(saleList) {
      this.saleDataTable = [];
      this.tableFinalData = [];
      for (var i = 0; i < saleList.length; i++) {
        if (saleList[i].valueSelect.length > 0) {
          this.saleDataTable.push(saleList[i]);
        }
      }
      console.log(this.saleDataTable, "~~~~~~~~~~~");
      var decar = [];
      var attrId = [];
      for (var i = 0; i < this.saleDataTable.length; i++) {
        decar.push(this.saleDataTable[i].valueSelect);
        attrId.push(this.saleDataTable[i].attrId);
      }
      //遍历进行交叉想成
      var decarResult = this.calcDescartes(decar);
      console.log(decar, attrId, "计算后的", this.calcDescartes(decar));
      //封装计算后的笛卡尔积和其他结果
      /**
       * [
       * 0: (3) ["6g", "黑色", "256"]
       * 1: (3) ["6g", "黑色", "128g"]
       * 2: (3) ["6g", "黑色", "64g"]
       * 3: (3) ["6g", "白色", "256"]
       * ]
       */
      for (var i = 0; i < decarResult.length; i++) {
        var tableItem = {};
        for (var j = 0; j < decarResult[i].length; j++) {
          tableItem["attr_" + attrId[j]] = decarResult[i][j];
          tableItem.price = 0;
          tableItem.stock = 0;
          tableItem.growBounds = tableItem.price;
          tableItem.buyBounds = tableItem.price;
          tableItem.work = [0, 0, 0, 0];
          tableItem.fullCount = 1;
          tableItem.discount = 0;
          tableItem.fullPrice = 0;
          tableItem.reducePrice = 0;
          tableItem.fullAddOther = 0;
          tableItem.images = [];

          //skuName
          tableItem.skuName = this.spuInfo.spuName + " " + decarResult[i];
          tableItem.skuDesc = "";
          tableItem.skuTitle = this.spuInfo.spuName + " " + decarResult[i];
          tableItem.skuSubtitle = this.spuInfo.spuName + " " + decarResult[i];
          tableItem.weight = 0;
        }
        this.tableFinalData.push(tableItem);
      }
      console.log("准备好的数据...", this.tableFinalData);
    },
    getAttrGroups() {
      console.log("三级分类，", this.spuInfo.catalogId);
      this.$http({
        url: this.$http.adornApiUrl(
          "/pms/attrgroup/list/category/" + this.spuInfo.catalogId
        ),
        method: "get",
        params: this.$http.adornParams({
          limit: 10000
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          // [0][1] [0][2] [1][0] [1][2]
          //外面的每个valueSelect需要处理成数组方便遍历
          this.attrGroups = data.data.list;
          this.finalValues = []; // [[1],[2],[3]]
          for (var i = 0; i < this.attrGroups.length; i++) {
            console.log(
              "this.attrGroups==>150",
              this.attrGroups[i].attrEntities
            );
            this.finalValues.push([]);
            if (this.attrGroups[i].attrEntities) {
              for (var j = 0; j < this.attrGroups[i].attrEntities.length; j++) {
                this.attrGroups[i].attrEntities[
                  j
                ].valueSelect = this.attrGroups[i].attrEntities[
                  j
                ].valueSelect.split(",");
                this.finalValues[i].push({
                  attrId: this.attrGroups[i].attrEntities[j].attrId,
                  attrName: this.attrGroups[i].attrEntities[j].attrName,
                  valueSelected: []
                });
                // this.finalValues[i][j] = {};
                //{attrId:this.attrGroups[i].attrEntities[j].attrId,valueSelected:[]}
              }
            }
          }
          // 保存每一组选中的值
          console.log(this.attrGroups, "159~~~~~~~~~~~~~~~~~~~~~~~~");
          //赋值表单的，防止为null
          this.spuInfo.attrGroups = data.data.list;
          console.log("this.spuInfo.attrGroups", this.spuInfo.attrGroups);
        } else {
        }
      });
    },
    getSaleGroups(catalogId) {
      this.$http({
        url: this.$http.adornApiUrl("/pms/attr/sale/" + catalogId),
        method: "get",
        params: this.$http.adornParams({
          limit: 10000
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.saleGroups = data.data.list;
          console.log(this.saleGroups, "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
          for (var i = 0; i < this.saleGroups.length; i++) {
            this.saleGroupValue.push({
              attrId: this.saleGroups[i].attrId,
              attrName: this.saleGroups[i].attrName,
              valueSelect: []
            });
            if (this.saleGroups[i].valueSelect) {
              this.saleGroups[i].valueSelect = this.saleGroups[
                i
              ].valueSelect.split(",");
            }
          }
        } else {
        }
      });
    },
    emitInput(list) {
      console.log("上传的文件", list);
      this.spuInfo.images = list;
    },
    skuUploadImage(row, list) {
      row.images = list;
    },
    next() {
      if (this.spuStep == 0) {
        //第一步，验证表单
        this.$refs.spuInfoform.validate(valid => {
          var flag = true;
          // var flag = valid;
          if (flag) {
            //查询出当前分类的所有属性分组
            this.getAttrGroups();
          } else {
            return false;
          }
        });
      }

      if (this.spuStep == 1) {
        this.btnText = "确认保存";
        //第二步，需要按照第一步选择的分类id，查询当前分类下的属性和属性组，如果没有则维护属性组
        if (this.spuInfo.catalogId) {
          //获取三级分类中的所有属性组
          this.getSaleGroups(this.spuInfo.catalogId);
        }
        //验证表单数据
      }

      if (this.spuStep == 2) {
        var data = this.getPostData();
        if (data == null) {
          this.$alert("数据不完整，请正确录入相关信息", "警告");
        }
        const loading = this.$loading({
          lock: true,
          text: "数据量较大，正在拼命保存中...",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)"
        });

        this.$http({
          url: this.$http.adornApiUrl("/pms/spuinfo/save"),
          method: "post",
          data: this.$http.adornData(data, false)
        })
          .then(({ data }) => {
            this.$message("保存成功，" + data.msg);
            loading.close();
            //路由跳转到指定位置
            this.$router.push({
              path: `/pms-spu/list`, // 只是把query改了，其他都没变
              query: {
                catId: this.catId
              }
            });
          })
          .catch(err => {
            this.$message("网络故障，出现错误。" + err);
            loading.close();
          });

        //
        return true;
      }

      this.spuStep++;
    },
    getPostData() {
      //获取需要提交保存的整个商品信息
      var data = {};
      data.spuName = this.spuInfo.spuName;
      data.brandId = this.spuInfo.brandId;
      data.catalogId = this.spuInfo.catalogId;
      data.publishStatus = this.spuInfo.publishStatus;
      data.spuDescription = this.spuInfo.spuDescription;
      data.spuImages = this.spuInfo.images;
      data.baseAttrs = [];
      //整理返回基本属性组合 this.finalValues
      if (this.finalValues.length <= 0) {
        return null;
      }
      for (var i = 0; i < this.finalValues.length; i++) {
        if (this.finalValues[i].length > 0) {
          for (var j = 0; j < this.finalValues[i].length; j++) {
            data.baseAttrs.push(this.finalValues[i][j]);
          }
        }
      }

      data.skus = [];
      //整理返回所有sku信息  this.tableFinalData
      if (this.tableFinalData.length <= 0) {
        return null;
      }
      for (var i = 0; i < this.tableFinalData.length; i++) {
        var item = this.tableFinalData[i];
        item.saleAttrs = [];
        for (var attr in item) {
          // console.log(i+"->当前遍历的属性是",attr,typeof attr);
          if (attr.startsWith("attr_")) {
            //说明当前遍历的是属性id
            var attrId = attr.split("_")[1];
            item.saleAttrs.push({
              attrId: attr.split("_")[1],
              attrValue: item["" + attr]
            });
          }
        }
        data.skus.push(item);
      }
      console.log("最终的数据是：", data);
      return data;
    },

    prev() {
      this.spuStep--;
    },
    selectedValue(val) {
      this.spuInfo.catalogId = val;
      console.log("spuInfo.catalogId", this.spuInfo.catalogId);
    },
    brandSelectedVal(val) {
      this.spuInfo.brandId = val;
      console.log("spuInfo.brandId", this.spuInfo.brandId);
    },
    handleCheckedAttrChange(val) {
      //  finalValues:[
      //   {attrId:'',valueSelected:''}
      // ],
      // for(var i=0;i<this.finalValues.length;i++){
      //     if(a==this.finalValues[i].attrId && this.finalValues[attrId]){

      //     }
      // }
      //()
      console.log(val);
      console.log("绑定的数据=>", this.finalValues);
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