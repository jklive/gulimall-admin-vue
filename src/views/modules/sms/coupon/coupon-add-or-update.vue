<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      label-width="200px"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-position="right"
    >
      <el-form-item label="优惠卷类型" prop="couponType">
        <el-select v-model="dataForm.couponType" placeholder="优惠卷类型">
          <el-option label="全场赠券" :value="0"></el-option>
          <el-option label="会员赠券" :value="1"></el-option>
          <el-option label="购物赠券" :value="2"></el-option>
          <el-option label="注册赠券" :value="3"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="优惠卷名字" prop="couponName">
        <el-input v-model="dataForm.couponName" placeholder="优惠卷名字"></el-input>
      </el-form-item>
      <el-form-item label="数量" prop="num">
        <el-input-number v-model="dataForm.num" placeholder="数量"></el-input-number>
      </el-form-item>
      <el-form-item label="金额" prop="amount">
        <el-input-number v-model="dataForm.amount" placeholder="金额"></el-input-number>
      </el-form-item>
      <el-form-item label="每人限领" prop="perLimit">
        <el-input v-model="dataForm.perLimit" placeholder="每人限领张数"></el-input>
      </el-form-item>
      <el-form-item label="使用门槛" prop="minPoint">
        
        <el-input v-model="dataForm.minPoint" placeholder="使用门槛">
          <template slot="prepend">达到</template>
          <template slot="append">积分</template>
        </el-input>
      </el-form-item>
      <el-form-item label="有效期">
        <el-col :span="11">
          <el-form-item prop="startTime">
            <el-date-picker v-model="dataForm.startTime" type="datetime" placeholder="开始时间" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
          </el-form-item>
        </el-col>
        <el-col :span="2">-</el-col>
        <el-col :span="11">
          <el-form-item prop="endTime">
            <el-date-picker v-model="dataForm.endTime" type="datetime" placeholder="结束时间" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
          </el-form-item>
        </el-col>
      </el-form-item>
      <el-form-item label="使用类型" prop="useType">
        <el-select v-model="dataForm.useType" placeholder="使用类型">
          <el-option label="全场通用" :value="0"></el-option>
          <el-option label="指定分类" :value="1"></el-option>
          <el-option label="指定商品" :value="2"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="备注" prop="note">
        <el-input v-model="dataForm.note" placeholder="备注"></el-input>
      </el-form-item>
      <el-form-item label="发行数量" prop="publishCount">
        <el-input v-model="dataForm.publishCount" placeholder="发行数量"></el-input>
      </el-form-item>
      <!-- id不为null，修改的时候显示这几个信息 -->
      <el-form-item v-if="dataForm.id" label="已使用数量" >{{dataForm.useCount}}</el-form-item>
      <el-form-item v-if="dataForm.id" label="领取数量" >{{dataForm.receiveCount}}</el-form-item>

      <el-form-item label="领取有效期">
        <el-col :span="11">
          <el-form-item prop="enableStartTime">
            <el-date-picker v-model="dataForm.enableStartTime" type="datetime" placeholder="开始时间" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
          </el-form-item>
        </el-col>
        <el-col :span="2">-</el-col>
        <el-col :span="11">
          <el-form-item prop="enableEndTime">
            <el-date-picker v-model="dataForm.enableEndTime" type="datetime" placeholder="结束时间" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
          </el-form-item>
        </el-col>
      </el-form-item>
      <el-form-item v-if="dataForm.id" label="优惠码" >{{dataForm.code}}</el-form-item>
      <el-form-item label="可以领取的会员等级" prop="memberLevel">
        <el-select v-model="dataForm.memberLevel" placeholder="可以领取的会员等级">
          <el-option label="不限等级" :value="0"></el-option>
          <el-option label="黑铁" :value="1"></el-option>
          <el-option label="青铜" :value="2"></el-option>
          <el-option label="白银" :value="2"></el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        couponType: "",
        couponName: "",
        num: "",
        amount: "",
        perLimit: "",
        minPoint: "",
        startTime: "",
        endTime: "",
        useType: "",
        note: "",
        publishCount: "",
        useCount: "",
        receiveCount: "",
        enableStartTime: "",
        enableEndTime: "",
        code: "",
        memberLevel: ""
      },
      dataRule: {
        couponType: [
          {
            required: true,
            message:
              "优惠卷类型[0->全场赠券；1->会员赠券；2->购物赠券；3->注册赠券]不能为空",
            trigger: "blur"
          }
        ],
        couponName: [
          { required: true, message: "优惠卷名字不能为空", trigger: "blur" }
        ],
        num: [{ required: true, message: "数量不能为空", trigger: "blur" }],
        amount: [{ required: true, message: "金额不能为空", trigger: "blur" }],
        perLimit: [
          { required: true, message: "每人限领张数不能为空", trigger: "blur" }
        ],
        minPoint: [
          { required: true, message: "使用门槛不能为空", trigger: "blur" }
        ],
        startTime: [
          { required: true, message: "开始时间不能为空", trigger: "blur" }
        ],
        endTime: [
          { required: true, message: "结束时间不能为空", trigger: "blur" }
        ],
        useType: [
          { required: true, message: "使用类型不能为空", trigger: "blur" }
        ],
        note: [{ required: true, message: "备注不能为空", trigger: "blur" }],
        publishCount: [
          { required: true, message: "发行数量不能为空", trigger: "blur" }
        ],
        useCount: [
          { required: true, message: "已使用数量不能为空", trigger: "blur" }
        ],
        receiveCount: [
          { required: true, message: "领取数量不能为空", trigger: "blur" }
        ],
        enableStartTime: [
          {
            required: true,
            message: "可以领取的开始日期不能为空",
            trigger: "blur"
          }
        ],
        enableEndTime: [
          {
            required: true,
            message: "可以领取的结束日期不能为空",
            trigger: "blur"
          }
        ],
        code: [{ required: true, message: "优惠码不能为空", trigger: "blur" }],
        memberLevel: [
          {
            required: true,
            message: "可以领取的会员等级[0->不限等级，其他-对应等级]不能为空",
            trigger: "blur"
          }
        ]
      }
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornApiUrl(`/sms/coupon/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.couponType = data.data.couponType;
              this.dataForm.couponName = data.data.couponName;
              this.dataForm.num = data.data.num;
              this.dataForm.amount = data.data.amount;
              this.dataForm.perLimit = data.data.perLimit;
              this.dataForm.minPoint = data.data.minPoint;
              this.dataForm.startTime = data.data.startTime;
              this.dataForm.endTime = data.data.endTime;
              this.dataForm.useType = data.data.useType;
              this.dataForm.note = data.data.note;
              this.dataForm.publishCount = data.data.publishCount;
              this.dataForm.useCount = data.data.useCount;
              this.dataForm.receiveCount = data.data.receiveCount;
              this.dataForm.enableStartTime = data.data.enableStartTime;
              this.dataForm.enableEndTime = data.data.enableEndTime;
              this.dataForm.code = data.data.code;
              this.dataForm.memberLevel = data.data.memberLevel;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornApiUrl(
              `/sms/coupon/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              couponType: this.dataForm.couponType,
              couponName: this.dataForm.couponName,
              num: this.dataForm.num,
              amount: this.dataForm.amount,
              perLimit: this.dataForm.perLimit,
              minPoint: this.dataForm.minPoint,
              startTime: this.dataForm.startTime,
              endTime: this.dataForm.endTime,
              useType: this.dataForm.useType,
              note: this.dataForm.note,
              publishCount: this.dataForm.publishCount,
              useCount: this.dataForm.useCount,
              receiveCount: this.dataForm.receiveCount,
              enableStartTime: this.dataForm.enableStartTime,
              enableEndTime: this.dataForm.enableEndTime,
              code: this.dataForm.code,
              memberLevel: this.dataForm.memberLevel
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    }
  }
};
</script>
