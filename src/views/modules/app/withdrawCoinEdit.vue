<template>
  <div>
    <el-dialog title="提币" :inline="true" :close-on-click-modal="false" :visible.sync="dialogVisible" :append-to-body="true" class="withdraw-coin-box">
      <el-form :model="dataForm" :rules="rules" ref="dataForm"  label-width="100px">
        <el-form-item label="币种" prop="name">
          <el-select v-model="dataForm.coin" placeholder="请选择币种">
            <el-option :label="dataForm.coin" :value="dataForm.coin"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="提币地址" prop="address">
          <el-input v-model="dataForm.address"></el-input>
        </el-form-item>
        <el-form-item label="地址标签" prop="tag">
          <el-input v-model="dataForm.tag"></el-input>
        </el-form-item>
        <el-form-item label="验证码" prop="verifyCode">
          <el-input v-model="dataForm.verifyCode"></el-input>
          <el-button type="primary" class="btn" v-if="!validateStatus" @click="getVerifyCode">获取验证码</el-button>
          <span v-else>剩余{{time}}秒</span>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm('dataForm')">添加地址</el-button>
        </el-form-item>
      </el-form>
      <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle" style="width: 100%;">
      <el-table-column prop="coin"  label="币种"></el-table-column>
      <el-table-column prop="address"  label="地址" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="tag"  label="地址标签"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="200" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="deleteAddr(scope.row.address)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="phone-table">
      <div class="tr" v-for="item,i in dataList">
        <phone-column :prop="item.coin" label="币种"></phone-column>
        <phone-column :prop="item.address" label="地址"></phone-column>
        <phone-column :prop="item.tag" label="地址标签"></phone-column>

        <phone-column label="操作" :data="dataList" :index="i">
          <template slot-scope="scope">
          <el-button type="text" size="small" @click="deleteAddr(scope.row.address)">删除</el-button>
        </template>
        </phone-column>
      </div>
      <div v-if="dataList.length==0">
        <img src="/static/img/no-data.jpg" class="no-data" alt="暂无数据" />
      </div>
    </div>
    </el-dialog>
  </div>
</template>
<style type="text/css" scoped="scoped">

</style>
<script>
  export default {
    data () {
      return {
        dataList: [],
        dataListLoading: false,
        applicationId: '',
        dialogVisible: false,
        validateStatus: false, // 验证码显示状态
        time: null, // 倒计时间
        dataForm: {
          coin: '',
          address: '',
          tag: '',
          verifyCode: ''
        },
        rules: {
          address: [{
            required: true,
            message: '请输入提币地址',
            trigger: 'blur'
          }],
          tag: [{
            required: true,
            message: '请输入地址标签',
            trigger: 'blur'
          }],
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
        if (!val) {
          clearInterval(window.validateIntervalx)
        }
      }
    },
    methods: {
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      init (applicationId) {
        this.applicationId = applicationId
        this.dialogVisible = true
        this.dataForm.address = ''
        this.dataForm.tag = ''
        this.dataForm.verifyCode = ''
        this.validateStatus = false
        this.time = 60
        this.getList()
      },
      getList () {
        this.$http({
          url: this.$http.adornUrl('/sys/withdraw/coin/getCoinAddressList'),
          method: 'post',
          data: this.$http.adornData({
            'applicationId': this.applicationId
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.dataList = data.data.list
            this.dataForm.coin = data.data.application.otcBuyCoin
          } else {
            this.dataList = []
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
            this.validateStatus = true
            let time = 60
            clearInterval(window.validateIntervalx)
            window.validateIntervalx = setInterval(() => {
              if (this.time > 0) {
                this.time = time
                time--
              } else {
                this.time = 60
                this.validateStatus = false
                clearInterval(window.validateIntervalx)
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
      submitForm (formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl('/sys/withdraw/coin/createCoinAddress'),
              method: 'post',
              data: this.$http.adornData({
                'applicationId': this.applicationId,
                'address': this.dataForm.address,
                'tag': this.dataForm.tag,
                'verifyCode': this.dataForm.verifyCode
              })
            }).then(({data}) => {
              if (data && data.code === 200) {
                this.dataForm.address = ''
                this.dataForm.tag = ''
                this.dataForm.verifyCode = ''
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500
                })
                // 父组件向上传递值
                this.$emit('success', this.applicationId, this.dataForm.coin)
                this.dialogVisible = false // 隐藏当前框
              } else {
                this.$message.error(data.msg)
              }
            })
          } else {
            console.log('error submit!!')
            return false
          }
        })
      },
      deleteAddr (address) {
        this.$http({
          url: this.$http.adornUrl('/sys/withdraw/coin/deleteCoinAddress'),
          method: 'post',
          data: this.$http.adornData({
            'applicationId': this.applicationId,
            'address': address
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500
            })
            this.getList()
          } else {
            this.$message.error(data.msg)
          }
        })
      }
    }
  }
</script>
