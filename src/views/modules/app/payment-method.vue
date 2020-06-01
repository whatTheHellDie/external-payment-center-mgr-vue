<template>
  
  <div class="mod-user">
    <el-dialog
    title="收款方式"
    :close-on-click-modal="false"
    :visible.sync="visible" >
    <div class="receipt-wrap wrap-white">
    <div class="receipt-box">
      <div v-for="(item, index) in accountList" :key="item.receiveType">
        <div v-if="item.account" class="receipt already">
          <div class="body">
            <template>
              <img v-if="item.receiveType === 1" class="ic-left" src="/static/img/ic_order_pay_big@2x.png">
              <img v-else-if="item.receiveType === 2" class="ic-left" src="/static/img/ic_order_chart_big@2x.png">
              <img v-else class="ic-left" src="/static/img/ic_order_card_big@2x.png">
            </template>
            <div class="info">
              <p>账号：{{item.account}}</p>
              <p class="name">姓名：{{item.trueName}}</p>
            </div>
          </div>
          <div class="foot">
            <span class="edit" @click="updateReceipt(item.receiveType, 'edit',item.id)">编辑</span>
          </div>
        </div>
        <div v-else class="receipt not">
          <div class="body">
            <template>
              <img v-if="item.receiveType === 1" class="ic-left" src="/static/img/ic_order_pay_big@2x.png">
              <img v-else-if="item.receiveType === 2" class="ic-left" src="/static/img/ic_order_chart_big@2x.png">
              <img v-else class="ic-left" src="/static/img/ic_order_card_big@2x.png">
            </template>
            <div class="info">
              <span>未添加</span>
            </div>
            <img class="ic-right" @click="updateReceipt(item.receiveType, 'add')" src="/static/img/ic_plus_circle@2x.png">
          </div>
        </div>
      </div>
    </div>
  </div>
  </el-dialog>
  <!-- 弹窗, 收款方式：支付宝微信 -->
  <!--编辑支付方式-->
    <payment-method-edit v-if="showPaymentMethodEdit" ref="showPayMethodEditRef"></payment-method-edit>
  </div>
  
</template>

<script>
import paymentMethodEdit from './payment-method-edit'
  export default {
    data () {
      return {
        showPaymentMethodEdit:false,
      visible:true,
      accountList: [],
      showMessage: false,
      message: '',
      applicationId:''
    }
  },
  components:{
    paymentMethodEdit
  },
  methods: {
    init(applicationId){
      this.visible=true
      this.applicationId=applicationId
      this.$nextTick(()=>{
        this.getReceipt();
      })
      
    },
    // 获取收款
    getReceipt() {
      console.log(JSON.stringify({
          "applicationId": this.applicationId,
        }))
      this.$axios({
        url: this.$http.adornUrl('/sys/payMethod/list'),
        method: 'post',
        //发送格式为json
        data: JSON.stringify({
          "applicationId": this.applicationId,
        }),
        headers: {
          'Content-Type': 'application/json; charset=utf-8',
          'token': this.$cookie.get('token')
        }
      }).then(data => {
        console.log(data)
        if(data.data.code == 0) {
          this.accountList = data.data.data;
        } else {
         this.$alert(data.data.msg, '提示', {
          confirmButtonText: '确定',
          callback: action => {
          }
        });
        }
      })

    },
    // 更新收款方式
    updateReceipt(receiveType, method,id) {
      //如果有收款方式
      let obj;
      if(method=="add"){
          obj={receiveType,method,applicationId:this.applicationId}
      }else{
        obj=obj={receiveType,method,applicationId:this.applicationId,id}
      }
      this.visible=false;
      this.showPaymentMethodEdit=true
            this.$nextTick(() => {
              this.$refs.showPayMethodEditRef.init(obj)
            })
//    this.$router.push({
//      name: 'payment-method-edit',
//      params: {
//        receiveType,
//        method,
//        applicationId:this.applicationId
//      }
//    });
    }
  }
  }
</script>
<style>
</style>
