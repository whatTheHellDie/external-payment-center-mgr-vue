<template>
  <div>
    <el-dialog title="提币" :inline="true" :close-on-click-modal="false" :visible.sync="dialogVisible" :append-to-body="true" class="withdraw-coin-box">
      <el-form :model="dataForm" :rules="rules" ref="dataForm"  label-width="100px">
        <el-form-item label="提币地址" prop="name">
          <el-select v-model="dataForm.address" @visible-change="getAddrList" placeholder="请添加地址">
            <el-option
              v-for="item in addrList"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        <el-button type="primary" class="btn" @click="addressManage">提币地址管理</el-button>
        </el-form-item>
        <el-form-item label="提币数量" prop="num">
          <el-input v-model="dataForm.num"></el-input>
          <el-button type="text" class="btn1" @click="allWithdraw">全部提币</el-button>
        </el-form-item>
        <div class="has-coin">可提数量：{{this.totalBalance}} {{this.proCoin}}</div>
        <el-form-item label="验证码" prop="verifyCode">
          <el-input v-model="dataForm.verifyCode"></el-input>
          <el-button type="primary" class="btn" v-if="!withdrawValidateStatus" @click="getVerifyCode">获取验证码</el-button>
          <span v-else>剩余{{time}}秒</span>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm('dataForm')">提币</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <withdraw-coin-edit v-if="showWithdrawCoinEdit"  ref="showWithdrawCoinEditRef" @success="init" ></withdraw-coin-edit>
  </div>
</template>
<style type="text/css" scoped="scoped">

</style>
<script>
  import WithdrawCoinEdit from './withdrawCoinEdit'
  export default {
    data () {
      const validateNumValue = (rule, value, callback) => { // 校验币种价值
        if (value.length === 0) callback(new Error('请输入数字, 且仅保留两位小数点'))
        if (value.length > 0 && (!/^\d+(\.\d{1,2})?$/.test(value) || value > 99999999.99 || value <= 0)) {
          callback(new Error('请输入数字, 且仅保留两位小数点'))
        } else {
          callback()
        }
      }
      return {
        applicationId: '',
        showWithdrawCoinEdit: false,
        dialogVisible: false,
        withdrawValidateStatus: false, // 验证码显示状态
        time: null, // 倒计时间
        addrList: [],
        totalBalance: '',
        proCoin: '', // 项目方购买币种
        dataForm: {
          address: '',
          num: '',
          verifyCode: ''
        },
        rules: {
          address: [{
            required: true,
            message: '请选择提币地址',
            trigger: 'blur'
          }],
          num: [{ validator: validateNumValue, trigger: 'blur' }],
          verifyCode: [{
            required: true,
            message: '请输入验证码',
            trigger: 'blur'
          }]
        }
      }
    },
    watch: {
      dialogVisible (val) {
        if (!val) { // 关闭当前页面
          clearInterval(window.withdrawValidateInterval)
        }
      }
    },
    components: {
      WithdrawCoinEdit
    },
    methods: {
      addressManage () {
        this.dialogVisible = false // 隐藏当前框
        this.showWithdrawCoinEdit = true
        this.$nextTick(() => {
          this.$refs.showWithdrawCoinEditRef.init(this.applicationId)
        })
      },
      init (applicationId, otcBuyCoin) {
        this.applicationId = applicationId
        this.dialogVisible = true
        this.proCoin = otcBuyCoin
        this.addrList = []
        this.dataForm.address = ''
        this.dataForm.num = ''
        this.dataForm.verifyCode = ''
        this.withdrawValidateStatus = false
        this.time = 60
        this.$http({
          url: this.$http.adornUrl('/sys/withdraw/cash/getBalance'),
          method: 'post',
          data: this.$http.adornData({
            'applicationId': this.applicationId.toString()
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.totalBalance = data.data.total
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      getAddrList () {
        this.addrList = []
        this.$http({
          url: this.$http.adornUrl('/sys/withdraw/coin/getCoinAddressList'),
          method: 'post',
          data: this.$http.adornData({
            'applicationId': this.applicationId
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            data.data.list.forEach((item, index) => {
              this.addrList.push({value: item.address, label: item.tag + ' ：' + item.address})
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      // 获取验证码
      getVerifyCode () {
        this.$http({
          url: this.$http.adornUrl('/sys/withdraw/coin/getSmsCode'),
          method: 'post',
          data: this.$http.adornData({
            'applicationId': this.applicationId
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.withdrawValidateStatus = true
            let time = 60
            clearInterval(window.withdrawValidateInterval)
            window.withdrawValidateInterval = setInterval(() => {
              if (time > 0) {
                this.time = time
                time--
              } else {
                this.time = 60
                this.withdrawValidateStatus = false
                clearInterval(window.withdrawValidateInterval)
              }
            }, 1000)
            this.$message({
              message: '验证码发送成功，请注意查收',
              type: 'success',
              duration: 1500
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      // 全部提币
      allWithdraw () {
        this.dataForm.num = this.totalBalance
      },
      // 提交表单
      submitForm (formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl('/sys/withdraw/coin/createWithdrawCoin'),
              method: 'post',
              data: this.$http.adornData({
                'applicationId': this.applicationId,
                'address': this.dataForm.address,
                'num': this.dataForm.num,
                'verifyCode': this.dataForm.verifyCode
              })
            }).then(({data}) => {
              if (data && data.code === 200) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500
                })
                 // 提转到提币记录页面
                this.dialogVisible = false
                this.$router.push({path: '/withdraw-coin-order-list', query: {applicationId: this.applicationId}})
              } else {
                this.$message.error(data.msg)
              }
            })
          } else {
            return false
          }
        })
      }
    }
  }
</script>
