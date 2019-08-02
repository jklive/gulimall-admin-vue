<template>
  <el-dialog :title="dataForm.attrId ? '修改' : '新增'" :visible.sync="visible">
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      label-width="120px"
      @keyup.enter.native="dataFormSubmit()"
    >
      <el-form-item label="属性名称" prop="attrName">
        <el-input v-model="dataForm.attrName"></el-input>
      </el-form-item>
      <el-form-item
        v-for="(itemAttr, index) in dataForm.valueSelect"
        :label="'属性值'+(index+1)"
        :key="index"
        :prop="dataForm.valueSelect[index]"
      >
        <el-input v-model="dataForm.valueSelect[index]" style="width:40%;margin-right:20px;"></el-input>
        <el-button @click.prevent="removeAttr(itemAttr,index)">删除</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" class="el-button--mini" @click.prevent="addAttribute()">添加属性值</el-button>
      </el-form-item>
      <el-form-item label="属性类型" prop="attrType">
        <el-select v-model="dataForm.attrType" placeholder="请选择属性类型">
          <el-option label="销售属性" :value="0"></el-option>
          <el-option label="基本属性" :value="1"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item v-if="dataForm.attrType == 1" label="属性分组" prop="attrGroupId">
        <el-select v-model="dataForm.attrGroupId" placeholder="请选择属性类型">
          <el-option
            v-for="item in attrGroups"
            :key="item.attrGroupId"
            :label="item.attrGroupName"
            :value="item.attrGroupId"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item v-if="dataForm.attrType == 1" label="是否需要检索" prop="searchType">
        <el-switch
          v-model="dataForm.searchType"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
      <el-form-item label="可否选择多个值" v-if="dataForm.attrType == 1" prop="valueType">
        <el-switch
          v-model="dataForm.valueType"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
      <el-form-item label="是否启用" prop="enable">
        <el-switch
          v-model="dataForm.enable"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
      <el-form-item label="展示在介绍" prop="showDesc" v-if="dataForm.attrType == 1">
        <el-switch
          v-model="dataForm.showDesc"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
      <!-- <el-input v-model="dataForm.logo" placeholder="品牌logo地址"></el-input> -->
      <el-form-item label="图标" prop="icon">
        <SingleUpload
          v-model="dataForm.icon"
          style="width: 300px;display: inline-block;margin-left: 10px"
        ></SingleUpload>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import SingleUpload from "@/components/upload/singleUpload";
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        attrId: 0,
        attrName: "",
        valueSelect: [""],
        attrType: "",
        attrGroupId: "",
        searchType: 0,
        valueType: 0,
        icon: "",
        catelogId: 0,
        enable: 1,
        showDesc: 0
      },
      dataRule: {
        attrName: [
          { required: true, message: "属性名称不能为空", trigger: "blur" }
        ],
        valueSelect: [
          {
            required: true,
            message: "属性值不能为空",
            trigger: "blur",
            type: Array
          }
        ],
        attrType: [
          { required: true, message: "属性类型为必选项", trigger: "blur" }
        ]
      },
      attrGroups: []
    };
  },
  props: ["catId"],
  watch: {
    catId(val) {
      this.dataForm.catelogId = val;
    }
  },
  components: { SingleUpload },
  methods: {
    init() {
      this.dataForm.catelogId = this.catId;
      this.visible = true;
      this.$nextTick(() => {
        // this.$refs["dataForm"].resetFields();
        // 修改的时候回显分组数据
        if (this.catId) {
          this.$http({
            url: this.$http.adornApiUrl(
              `/pms/attrgroup/list/category/${this.catId}`
            ),
            method: "get",
            params: this.$http.adornParams({
              limit: 1000
            })
          }).then(({ data }) => {
            console.log(data, "999999999999");
            if (data && data.code === 0) {
              this.attrGroups = data.data.list;
            }
          });
        }
      });
    },
    // 获取属性的分组信息
    getgroupInfo() {
      this.$http({
        url: this.$http.adornApiUrl(
          `/pms/attrgroup/list/category/${this.catId}`
        ),
        method: "get"
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataForm.attrName = data.data.attrName;
          this.dataForm.valueSelect = data.data.valueSelect;
          this.dataForm.attrType = data.data.attrType;
          this.dataForm.attrGroupId = data.data.attrGroupId;
          this.dataForm.searchType = data.data.searchType;
          this.dataForm.icon = data.data.icon;
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        console.log(this.dataForm);
        if (valid) {
          //验证通过
          var data = this.dataForm;
          data.valueSelect = data.valueSelect.join(",");
          //1、保存属性信息
          var url =
            this.dataForm.attrId && this.dataForm.attrId > 0
              ? "/pms/attr/update"
              : "/pms/attr/save";
          this.$http({
            url: this.$http.adornApiUrl(url),
            method: "post",
            data: this.$http.adornData(data, false, "json")
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "属性保存成功",
                type: "success"
              });
              this.$emit("saveSuccess", this.dataForm.attrType);
              //3、清空数据
              this.clearFileds();
              //2、关闭模态框
              this.visible = false;
            } else {
              this.$message({
                message: "属性保存失败,请检查填写的值",
                type: "error"
              });
            }
          });
        }
      });
    },
    addAttribute() {
      this.dataForm.valueSelect.push("");
    },
    removeAttr(itemAttr, index) {
      if (index == 0) {
        return false;
      }
      this.dataForm.valueSelect.splice(index, 1);
    },
    clearFileds() {
      this.dataForm = {
        attrId: 0,
        attrName: "",
        valueSelect: [""],
        attrType: "",
        attrGroupId: "",
        searchType: 0,
        valueType: 0,
        icon: "",
        catelogId: 0,
        enable: 1
      };
    },
    showDetails(data) {
      //当前属性赋值；
      // this.dataForm = data;
      // //将valueSelect转化为数组
      // this.dataForm.valueSelect = ;
      this.dataForm = {
        attrId: data.attrId,
        attrName: data.attrName,
        valueSelect: data.valueSelect.split(","),
        attrType: data.attrType,
        attrGroupId: data.group ? data.group.attrGroupId : 0,
        searchType: data.searchType,
        valueType: data.valueType,
        icon: data.icon,
        catelogId: data.catelogId,
        enable: data.enable,
        showDesc: data.showDesc
      };
      console.log("准备回显的数据，", this.dataForm);
      //初始化弹出
      this.init();
    }
  }
};
</script>
