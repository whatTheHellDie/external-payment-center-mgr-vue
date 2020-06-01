<template>
  <el-dialog
    title="收款方式"
    :close-on-click-modal="false"
    :visible.sync="visible" width="1000px">
  <div class="receipt-edit-wrap wrap-white">

    <template>
      <div v-if="receiveType === 1 || receiveType === 2" class="receipt-edit-box">
        <div class="head">
          <div class="receipt-edit">
            <h1>收款方式</h1>
            <span>{{receiveType === 1 ? '支付宝' : '微信'}}</span>
          </div>
          <div class="receipt-edit name">
            <h1>姓名</h1>
            <input type="text" v-model="trueName" placeholder="请输入姓名">
          </div>
        </div>
        <div class="body">
          <div class="receipt-edit">
            <h1>{{receiveType === 1 ? '支付宝账号' : '微信支付账号'}}</h1>
            <div class="cont">
              <input type="text" v-model="account" :placeholder="receiveType === 1 ? '请输入支付宝账号' : '请输入微信支付账号'">
            </div>
          </div>
          <div class="receipt-edit qr-code">
            <h1>上传收款二维码</h1>
            <div class="cont">
              <img v-if="this.receivePicUrl" :src="receivePicUrl" />
              <img v-else class="add" src="/static/img/ic_add_icon@2x.png" />
              <input @change='add_img($event,0)' accept="image/*;capture=camera" type="file" id="saveImage">
            </div>
          </div>
        </div>
      </div>
      <div v-else class="receipt-edit-box">
        <div class="head">
          <div class="receipt-edit">
            <h1>收款方式</h1>
            <span>银行转账</span>
          </div>
          <div class="receipt-edit name">
            <h1>姓名</h1>
            <input type="text" v-model="trueName" placeholder="请输入姓名">
          </div>
        </div>
        <div class="body">
          <div class="receipt-edit">
            <h1>开户银行</h1>
            <div class="cont">
              <input type="text" v-model="bankName" placeholder="请输入开户银行">
            </div>
          </div>
          <div class="receipt-edit">
            <h1>开户支行</h1>
            <div class="cont">
              <input type="text" v-model="bankBranchName" placeholder="请输入开户支行">
            </div>
          </div>
          <div class="receipt-edit">
            <h1>银行卡账号</h1>
            <div class="cont">
              <input type="text" v-model="account" placeholder="请输入银行卡账号">
            </div>
          </div>
        </div>
      </div>
    </template>
    <el-button type="primary" style="width: 100%;margin-top: 29px;height: 40px;"  @click="confirmEdit">确认</el-button>

  </div>
  </el-dialog>
</template>

<script>

  export default {
    data () {
      return {
        visible:false,
        showPayMethod:true,
        receiveType: 1, // 收款方式
        receiveTypeName: '', // 收款名
        trueName: '', // 真实姓名
        account: '', // 收款账号
        bankName: '', // 开户银行
        bankBranchName: '', // 开户支行
        receivePicUrl: '', //收款二维码图片
        time: '', // 验证码剩余时间
        validateStatus: false,
        method: '', // add or edit
        memberId:'',
        id:''
    }
  },
  methods:{
    init(params){
      this.visible=true
      this.trueName=""
  this.account=""
  this.receivePicUrl=""
        this.getRouteParams(params);
        if(this.method === 'edit') {
          this.findReceipt();
        }
      },
    handleClose(done) {
        this.$confirm('取消后您做的更改不会保留，确认关闭？')
          .then(_ => {
            done();
          })
          .catch(_ => {});
      },
      getRouteParams(params) {
        if(Object.keys(params).length !== 0 || this.$cookie.get('receiptEditParams')) {
          if(this.$cookie.get('receiptEditParams') && Object.keys(params).length === 0) {
            params = JSON.parse(this.$cookie.get('receiptEditParams'))
          } else {
            this.$cookie.set('receiptEditParams', JSON.stringify(params))
          }
          this.receiveType = params.receiveType;
          this.method = params.method;
          this.memberId=params.applicationId
          if(params.id){
            this.id=params.id
          }
        }
      },
      // 查询收款
      findReceipt() {
        this.$axios({
          url: this.$http.adornUrl('/sys/payMethod/info'),
          method: 'post',
          data: JSON.stringify({
            "id": this.id
          }),
          headers: {
            'Content-Type': 'application/json; charset=utf-8',
            'token': this.$cookie.get('token')
          }
        }).then(data => {
          if(data.data.code === 0) {
            const res = data.data.data;
            this.receiveTypeName = res.receiveTypeName;
            this.trueName = res.trueName;
            this.account = res.account;
            this.bankBranchName = res.bankBranchName;
            this.bankName = res.bankName;
            this.receivePicUrl = res.receivePicUrl;
          } else {
            this.message = data.data.msg;
            this.showMessage = true;
          }
        });
      },
      // 更新收款
      updateReceipt() {
        let obj,url;
        if(this.method=='add'){
          url="/sys/payMethod/save"
          obj={
            "memberId":this.memberId,
            "receiveType": this.receiveType,
            "trueName": this.trueName,
            "account": this.account,
            "receivePicUrl": this.receivePicUrl,
            "bankName": this.bankName,
            "bankBranchName": this.bankBranchName
          }
        }else{
          url="/sys/payMethod/update"
          obj={
            "id":this.id,
            "receiveType": this.receiveType,
            "trueName": this.trueName,
            "account": this.account,
            "receivePicUrl": this.receivePicUrl,
            "bankName": this.bankName,
            "bankBranchName": this.bankBranchName
          }
        }
        const loading = this.$loading({
          lock: true,
          text: 'Loading',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)'
        });
        this.$axios({
          url: this.$http.adornUrl(url),
          method: 'post',
          data: JSON.stringify(obj),
          headers: {
            'Content-Type': 'application/json; charset=utf-8',
            'token': this.$cookie.get('token')
          }
        }).then(data => {
          loading.close();
          if(data.data.code == 0) {
            this.$message('修改支付方式成功');
            this.visible=false
          } else {
            this.$alert(data.data.msg, '提示', {
          confirmButtonText: '确定',
          callback: action => {
          }
          });
          }
        });
      },
      // 确认编辑
      confirmEdit() {
        if(!this.trueName) {
          this._showMessage('请填写姓名');
          return;
        }
        if(this.receiveType === 3 && !this.bankName) {
          this._showMessage('请填写开户银行');
          return;
        }
        if(this.receiveType === 3 && !this.bankBranchName) {
          this._showMessage('请填写开户支行');
          return;
        }
        const account = this.account;
        const phone = /^1[3456789]\d{9}$/.test(account);
        const email = /[\w]+(\.[\w]+)*@[\w]+(\.[\w])+/.test(account);
        const zfb = phone || email;
        const wx = /^[a-zA-Z][\w-]{5,}$/.test(account);
        const card = /^[0-9]*$/.test(account);
        if(!account) {
          this._showMessage('请填写账号');
          return;
        } else if(this.receiveType === 1 && !zfb) {
          this._showMessage('账号格式不正确');
          return;
        } else if(this.receiveType === 2 && !wx) {
          this._showMessage('账号格式不正确');
          return;
        } else if(this.receiveType === 3 && !card) {
          this._showMessage('账号格式不正确');
          return;
        }
        if(this.receiveType !== 3 && !this.receivePicUrl) {
          this._showMessage('请上传收款二维码');
          return;
        }
        this.updateReceipt();
      },
      // 上传图片
      add_img(event, index) {
        var img1 = event.target.files[0]


        if(index == 0) {
          let x = document.getElementById('saveImage').files[0];
          let params = new FormData()
          console.log(this.memberId)
           params.append('applicationId', this.memberId)
          params.append('file', x)
          params.append('receiveType', this.receiveType)
          this.$axios.post(this.$http.adornUrl('/paychatmsg/uploadImage'),
            params, {
              headers: {
                "Content-Type": "multipart/form-data",
                'token': this.$cookie.get('token')
              }
            }).then(({
            data
          }) => {
            if(data.code == 200) {
              this.receivePicUrl = data.data.url;

            } else {
              this.message = data.msg;
              this.showMessage = true
            }
          }).catch(({
            error
          }) => {

          })
        }
      },
      _showMessage(msg) {
         this.$alert(msg, '提示', {
          confirmButtonText: '确定',
          callback: action => {
          }
          });
      }
  }
  }
</script>
<style>
</style>
