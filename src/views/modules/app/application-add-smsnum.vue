<template>
  <el-dialog
    title="增加短信条数"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="110px">
      <el-form-item label="增加数量" prop="smsResidueNum" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.smsResidueNum" placeholder="短信条数" maxlength="8"></el-input>
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
    data () {
      const validateSmsResidueNum = (rule, value, callback) => {
        if (value.length > 0 && (!/^\d+?$/.test(value))) {
          callback(new Error('输入的短信条数无效'))
        } else {
          callback()
        }
      }

      return {
        visible: false,
        dataForm: {
          id: 0,
          smsResidueNum: ''
        },
        dataRule: {
          smsResidueNum: [
            { required: true, message: '请输入要添加的短信条数', trigger: 'blur' }
          ],
          smsResidueNum: [
            { validator: validateSmsResidueNum, trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
        })
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/payapplication/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.smsResidueNum = data.payApplication.smsResidueNum;
            }
          })
        }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/payapplication/updateSmsResidueNum`),
              method: 'post',
              data: this.$http.adornData({
                'applicationId': this.dataForm.id || undefined,
                'smsResidueNum': this.dataForm.smsResidueNum
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
