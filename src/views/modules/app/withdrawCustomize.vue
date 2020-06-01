<template>
  <div>
    <el-dialog title="提现"  :close-on-click-modal="false" :visible.sync="dialogVisible" :append-to-body="true">
      <div class="withdraw-coin-wrap withdraw-customize">

        <div class="money-box" style="border-top: none;">
          <div>
            <h1>提现数量</h1>
            <input type="text" v-if="notFinish" placeholder="请输入提现数量" v-model="currency" :readonly="readonly" @blur="getArrivalAmount">
            <input type="text" v-else v-model="currency" :readonly="readonly">
          </div>
          <div class="mention" v-if="notFinish">可提现数量：{{balance}} {{withdrawName}}</div>
        </div>
        <div class="money-box">
          <div>
            <h1 class="nobottom clearfix">{{arriveText}}<div class="number">{{withdrawArrivalAmount}} CNY</div></h1>
          </div>
        </div>
        <div class="money-box money-box1">
          <div class="shou clearfix">收款方式
            <div class="edit" v-if="edit" @click="editForm">编辑</div>
          </div>
          <div class="way-wrap">
            <div class="way" v-if="aliInfo.trueName" :class="{active:payWay==1}" @click="chooseWay(1)"><img src="/static/img/ic_order_pay@2x (2).png" class="img" />支付宝</div>
            <div class="way weixin" v-if="weiXinInfo.trueName" :class="{active:payWay==2}" @click="chooseWay(2)"><img src="/static/img/ic_order_chart@2x (2).png" class="img" />微信</div>
            <div class="way bank" v-if="bankInfo.trueName" :class="{active:payWay==3}" @click="chooseWay(3)"><img src="/static/img/ic_order_card@2x.png" class="img" />银行转账</div>
          </div>
          <div class="info">
            <div class="detail clearfix" v-if="payWay===1">收款账户：{{aliInfo.account}}</div>
            <div class="detail clearfix" v-else-if="payWay===2">收款账户：{{weiXinInfo.account}}</div>
            <div class="detail clearfix" v-else-if="payWay===3">银行卡号：{{bankInfo.account}}</div>
            <div v-if="payWay===1">姓名：{{aliInfo.trueName}}</div>
            <div v-if="payWay===2">姓名：{{weiXinInfo.trueName}}</div>
            <div v-if="payWay===3">姓名：{{bankInfo.trueName}}</div>
          </div>
        </div>
        <div v-if="!notFinish" class="money-box" @click="getMerchantOrder">
          <div>
            <h1 class="nobottom clearfix">联系商家<img src="/static/img/ic_message_blue@2x.png" class="mechant" alt="" /></h1>
          </div>
        </div>
        <div v-if="!notFinish" class="money-box-contract-tip">注：若长时间未收到，可联系商家查询提现进度</div>
        <div class="btn-box bottom1">
          <el-button type="primary" class="w100" v-if="notFinish" @click="confirmWithdraw">确认提现</el-button>
          <div v-else>
            <el-button type="primary" class="w100" v-if="orderStatus==100">待收款</el-button>
            <el-button type="primary" class="w100" v-if="orderStatus==104">超时取消</el-button>
            <el-button type="primary" class="w100" v-if="orderStatus==200">提现成功</el-button>
            <el-button type="info" class="w100" v-if="orderStatus==999">交易取消</el-button>
          </div>

        </div>
        <!--<span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>-->
      </div>
      <!--联系商家-->
      <!--<contact-merchant></contact-merchant>-->
    </el-dialog>
    <!--支付方式-->
    <pay-method v-if="showPayMethod" ref="showPayMethodRef"></pay-method>
  </div>
</template>
<style type="text/css">
  .w100 {
    width: 100%;
    margin-top: .29rem;
  }
</style>
<script>
  import payMethod from './payment-method'
  //import contactMerchant from './contactMerchant'
  export default {
    data() {
      return {
        showPayMethod: false,
        dialogVisible: true,
        arriveText: "预计到账",
        receiveTypeName: null,
        edit: true,
        aliInfo: {},
        weiXinInfo: {},
        bankInfo: {},
        payWay: null,
        btcAmount: '',
        balance: '',
        notFinish: true,
        backTime: 3,
        time: null,
        validateStatus: false,
        currency: '',
        withdrawArrivalAmount: "",
        readonly: false,
        notHistory: true,
        orderStatus: "",
        withdrawName: "",
        orderSn: "",
        id: "",
      };
    },
    //  created() {
    //
    //  },
    components: {
      //    contactMerchant,
      payMethod
    },
    watch: {
      currency(val, oldVal) {
        if(isNaN(Number(val))) {
          this.currency = oldVal
        }
      }
    },
    methods: {
      init(applicationId, data) {
        this.resetVal()
        this.applicationId = applicationId || 0
        this.dialogVisible = true
        data.data.data.map((item) => {
          switch(item.receiveType) {
            case 1:
              this.aliInfo = item;
              break;
            case 2:
              this.weiXinInfo = item;
              break;
            case 3:
              this.bankInfo = item;
              break;
          }
        })
        this.$axios({
          url: this.$http.adornUrl('/sys/withdraw/cash/getBalance'),
          method: 'post',
          //发送格式为json
          data: JSON.stringify({
            "applicationId": this.applicationId.toString(),
          }),
          headers: {
            'Content-Type': 'application/json; charset=utf-8',
            'token': this.$cookie.get('token')
          }
        }).then((data) => {
          console.log(data)
          if(data.data.code == 200) {
            this.balance=data.data.data.otcValue
            this.withdrawName=data.data.data.coinName

          } else {
            this.$alert(data.data.msg, '提示', {
              confirmButtonText: '确定',
            });
          }
        });
      },
      resetVal(){
        this.currency=""
        this.balance=""
        this.withdrawName=""
        this.payWay=null
        this.aliInfo= {},
        this.weiXinInfo= {},
        this.bankInfo= {}
        this.withdrawArrivalAmount=""
      },
      editForm() {
        this.dialogVisible = false;
        this.showPayMethod = true
        this.$nextTick(() => {
          this.$refs.showPayMethodRef.init(this.applicationId)
        })
      },
      getMerchantOrder() {
        this.$axios({
          url: this.$http.adornUrl('/app/contact'),
          method: 'post',
          //发送格式为json
          data: JSON.stringify({
            "orderSn": this.orderSn,
          }),
          headers: {
            'Content-Type': 'application/json; charset=utf-8',
            'token': this.$cookie.get('token')
          }
        }).then((data) => {
          if(data.data.code == 200) {
            this.$router.push({
              name: 'contactMerchant',
              params: {
                chat: data.data.data
              }
            })
          } else {
            this.$alert(data.data.msg, '提示', {
              confirmButtonText: '确定',
            });
          }
        });
      },
      chooseWay(type) {
        if(this.receiveTypeName) {
          return;
        }
        this.payWay = type
      },

      getArrivalAmount() {
        if(this.readonly) {
          //如果输入框只能看
          return false
        }
        if(this.currency==""){
          this.withdrawArrivalAmount="0"
          return false
        }
        if(Number(this.currency) - Number(this.balance) > 0) {
          this.currency = this.balance;
          return;
        }
        //      var balance=Number(this.balance)
        //      this.currency=balance.toFixed(8)
        //通过第三方代币/游戏币获取BTC数量
        this.$axios({
          url: this.$http.adornUrl('/sys/withdraw/cash/queryArrivalAmount'),
          method: 'post',
          //发送格式为json
          data: JSON.stringify({
            "withdrawAmount": this.currency,
            "applicationId": this.applicationId.toString(),
            coin: this.withdrawName
          }),
          headers: {
            'Content-Type': 'application/json; charset=utf-8',
            'token': this.$cookie.get('token')
          }
        }).then((data) => {
          if(data.data.code == 200) {
            this.withdrawArrivalAmount = data.data.data.withdrawArrivalAmount
          } else {
            this.$alert(data.data.msg, '提示', {
          confirmButtonText: '确定',
          callback: action => {
          }
          });
          }
        });
      },
      confirmWithdraw() {

        if(!this.currency) {
          this.$alert('请输入提现数量', '提示', {
            confirmButtonText: '确定',
          });
          return false
        }
        if(!this.payWay) {
          this.$alert('请先选择收款方式', '提示', {
            confirmButtonText: '确定',
          });
          return false
        }
         const loading = this.$loading({
          lock: true,
          text: 'Loading',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)'
        });
        this.$axios({
          url: this.$http.adornUrl('/sys/withdraw/cash/create'),
          method: 'post',
          //发送格式为json
          data: JSON.stringify({
            "applicationId":this.applicationId.toString(),
            "withdrawAmount": this.currency.toString(),
            "receiveType": this.payWay.toString(),
          }),
          headers: {
            'Content-Type': 'application/json; charset=utf-8',
            'token': this.$cookie.get('token')
          }
        }).then((data) => {
          loading.close();
          if(data.data.code == 200) {
            this.dialogVisible=false
            this.$router.push({path: '/withdraw-cash-order-list', query: {applicationId: this.applicationId}})
//          this.notFinish = false
//          this.readonly = true
//          this.edit = false
//          this.receiveTypeName = data.data.data.receiveTypeName
//          this.orderStatus = data.data.data.orderStatus;
//          //应为已到帐
//          if(data.data.data.actualArrivalAmount) {
//            this.withdrawArrivalAmount = data.data.data.actualArrivalAmount;
//          } else {
//            this.withdrawArrivalAmount = data.data.data.withdrawArrivalAmount;
//          }
//          this.orderSn = data.data.data.orderNo;
//          if(this.orderStatus == 200) {
//            this.arriveText = "已到帐"
//          }
          } else {
            this.$alert(data.data.msg, '提示', {
              confirmButtonText: '确定',
            });
          }
        });

      },
      getValidate() {
        //手机号
        if(!this.currency) {
          this.$alert("请输入提现数量", '提示', {
            confirmButtonText: '确定',
          });
          return false
        }
        let loader = this.$loading.show();
        this.$axios({
          url: this.$http.adornUrl('/app/getSmsCode'),
          method: 'post',
          headers: {
            'Content-Type': 'application/json; charset=utf-8',
            'token': this.$cookie.get('token')
          }
        }).then((data) => {
          loader.hide()
          if(data.data.code == 200) {
            this.validateStatus = true;
            this.time = 60;
            window.validateInterval = setInterval(() => {
              if(this.time > 0) {
                this.time--;
              } else {
                this.validateStatus = false;
                clearInterval(window.validateInterval)
              }
            }, 1000)
          } else {
            this.$alert(data.data.msg, '提示', {
              confirmButtonText: '确定',
            });
          }
        });
      }
    }
  }
</script>
