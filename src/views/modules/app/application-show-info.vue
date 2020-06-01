<template>
  <div>
  <el-dialog
    title="应用详情"
    :close-on-click-modal="false"
    :visible.sync="visible" class="phone-dialog" :modal="true">
    <el-row :gutter="20" style="border-bottom: 1px solid #ccc; padding-bottom: 5px;">
      <el-col :span="24" style="text-align:left">
        <b>应用名：</b>{{dataForm.applicationName}}
        <span style="margin-left:40px"></span>
        <b>项目方币利账号：</b>{{dataForm.projectAccount}}
        <span style="margin-left:40px"></span>
        <!--<span class="message-number">-->
          <!--<img src="~@/assets/img/messeges1.png" class="img" @click="msglist()">-->
          <!--<span class="text">{{msgTotal}}</span>-->
        <!--</span>-->
      </el-col>
    </el-row>

    <el-row :gutter="20">
      <el-col :span="4">
        <a style="cursor:pointer" @click="openMemberList()">{{totalCount}}</a>
        <br />
        <a style="cursor:pointer" @click="openMemberList()">会员总量（人）</a>
      </el-col>
      <el-col :span="4">
        {{todayLoginCount}}
        <br />
        今日登录（人）
      </el-col>
      <el-col :span="4">
        {{orderCount.todayRecharge}}
        <br />
        今日充币（CNY）
      </el-col>
      <el-col :span="4">
        {{orderCount.todayRechargeQuantity}}
        <br />
        今日充币数量（个）
      </el-col>
      <el-col :span="4">
        {{orderCount.todayWithdraw}}
        <br />
        今日提币（{{dataForm.otcBuyCoin}}）
      </el-col>
      <el-col :span="4">
        {{orderCount.todayWithdrawQuantity}}
        <br />
        今日提币数量（个）
      </el-col>
    </el-row>

    <el-row :gutter="20">
      <el-col :span="4">
        {{totalBalance}}
        <!--<br />-->
        <!--<el-button  type="primary" @click="toWithdraw">提现</el-button>-->
        <br />
        <el-button  type="primary" @click="toWithdrawCoin1">提币</el-button>
        <br />
        项目账户剩余资金（{{dataForm.otcBuyCoin}}）
      </el-col>
      <el-col :span="4">
        {{totalCnyBalance}}
        <br />
        项目账户剩余资金（CNY）
      </el-col>
      <el-col :span="4">
        <span v-if="dataForm.smsResidueNum">{{dataForm.smsResidueNum}}</span>
        <span v-else>0</span>
        <br />
        剩余可用短信（条）
      </el-col>
    </el-row>

    <el-row :gutter="20">
      <el-col :span="4">
        <a style="cursor:pointer" @click="openRechargeOrderList()">{{orderCount.totalRecharge}}</a>
        <br />
        <a style="cursor:pointer" @click="openRechargeOrderList()">充币累计（CNY）</a>
      </el-col>
      <el-col :span="4">
        <a style="cursor:pointer" @click="openWithdrawOrderList()">{{orderCount.totalWithdraw}}</a>
        <br />
        <a style="cursor:pointer" @click="openWithdrawOrderList()">提币累计（个）</a>
      </el-col>
      <el-col :span="4">
        <a style="cursor:pointer" @click="openWithdrawCashOrderList()">{{orderCount.totalWithdrawCashCny}}</a>
        <br />
        <a style="cursor:pointer" @click="openWithdrawCashOrderList()">提现累计（CNY）</a>
      </el-col>
      <el-col :span="4">
        <a style="cursor:pointer" @click="openWithdrawCoinOrderList()">{{orderCount.totalWithdrawCoin}}</a>
        <br />
        <a style="cursor:pointer" @click="openWithdrawCoinOrderList()">项目方提币（个）</a>
      </el-col>
    </el-row>

  </el-dialog>
<!-- 提现 -->
    <withdraw-customize v-if="showWithdraw"  ref="showWithdrawRef" ></withdraw-customize>
    <!-- 提币 -->
    <withdraw-coin v-if="showWithdrawCoin"  ref="showWithdrawCoinRef" ></withdraw-coin>
    <!--支付方式-->
    <pay-method v-if="showPayMethod" ref="showPayMethodRef"></pay-method>
    <!--提币地址-->
    <withdraw-coin-edit v-if="showWithdrawCoinEdit"  ref="showWithdrawCoinEditRef" ></withdraw-coin-edit>

  </div>
</template>
<style>
  .el-row {
    margin-bottom: 30px;
  }
  .el-col {
    text-align: center;
    border-radius: 4px;
  }
  .message-number{
    position: relative;
    display: inline-block;
  }
  .message-number .text{font-size:12px;position: absolute;right: -5px;top: -5px;
  width: 16px;height: 16px;
  background: #F24E4E;color: #fff;border-radius: 50%;text-align: center;line-height: 16px;overflow: hidden;
    text-overflow: ellipsis;white-space: nowrap;}
</style>
<script>
  import withdrawCustomize from './withdrawCustomize'
  import withdrawCoin from './withdrawCoin'
import payMethod from './payment-method'
  import WithdrawCoinEdit from './withdrawCoinEdit'

  export default {
    data () {
      return {
        showPayMethod: false,
        showWithdraw: false,
        showWithdrawCoin: false,
        showWithdrawCoinEdit: false,
        visible: false,
        dataForm: {},
        totalCount: '0', // 会员总量
        orderCount: {
          todayRecharge: '', // 今日充币（CNY）
          todayWithdraw: '', // 今日提币（BTC）
          totalRecharge: '', // 充币累计（CNY）
          totalWithdraw: '', // 提币累计（个）
          todayRechargeQuantity: '', // 今日充币数量（个）
          todayWithdrawQuantity: '', // 今日提币数量（个）
          totalWithdrawCashCny: '', // 提现累计（CNY）
          totalWithdrawCoin: '' // 提币累计（个）
        },
        msgTotal: '',
        smsResidueNum: '0',
        todayLoginCount: '0',
        // usdtTotalBalance: '0.00000000',
        totalBalance: '0.00000000', // 项目账户剩余资金
        totalCnyBalance: '0.00000000' // 项目账户剩余资金（CNY）
      }
    },
    components:{
      withdrawCustomize,
      withdrawCoin,
      payMethod,
      WithdrawCoinEdit
    },
    methods: {

      toWithdraw(){
        const loading = this.$loading({
          lock: true,
          text: 'Loading',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)'
        });
        this.$axios({
        url: this.$http.adornUrl('/sys/payMethod/list'),
        method: 'post',
        //发送格式为json
        data: JSON.stringify({
          "applicationId": this.dataForm.applicationId,
        }),
        headers: {
          'Content-Type': 'application/json; charset=utf-8',
          'token': this.$cookie.get('token')
        }
      }).then((data) => {
        loading.close();
        if(data.data.code === 0) {
          let hasInfo=false
          data.data.data.map(item=>{
            if(item.trueName){
              hasInfo=true
            }
          })
          if(hasInfo){
            //如果有收款方式
            this.showWithdraw=true
            this.$nextTick(() => {
            this.$refs.showWithdrawRef.init(this.dataForm.applicationId,data)
            })
          }else{
            //如果没有收款方式
            this.$confirm('您还没有添加收款方式, 是否添加?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              this.visible=false;
              this.showPayMethod=true
              this.$nextTick(() => {
                this.$refs.showPayMethodRef.init(this.dataForm.applicationId)
              })

            }).catch(() => {
              this.$message({
                type: 'info',
                message: '已取消添加'
              });
            });
          }

        } else {
          this.dialogVisible=false
           this.$alert(data.data.msg, '提示', {
          confirmButtonText: '确定',
        });
        }
      });

      },
      // 提币按钮跳转
      toWithdrawCoin1 () {

        this.$http({
          url: this.$http.adornUrl('/sys/withdraw/coin/getCoinAddressList'),
          method: 'post',
          data: this.$http.adornData({
            'applicationId': this.dataForm.applicationId
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            if (Array.isArray(data.data.list) && data.data.list.length === 0) {
              // 如果没有提币地址
              this.$confirm('您还没有添加提币地址, 是否添加?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
              }).then(() => {
                this.visible = false
                this.showWithdrawCoinEdit = true
                this.$nextTick(() => {
                  this.$refs.showWithdrawCoinEditRef.init(this.dataForm.applicationId)
                })
              }).catch(() => {
                this.$message({
                  type: 'info',
                  message: '已取消添加'
                })
              })
            } else {
              // 如果有提币地址
              this.showWithdrawCoin = true
              this.$nextTick(() => {
                this.$refs.showWithdrawCoinRef.init(this.dataForm.applicationId, this.dataForm.otcBuyCoin)
              })
            }
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/payapplication/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm = data.payApplication;
              var totalCount = data.totalCount[this.dataForm.applicationId];
              var todayLoginCount = data.todayLoginCount[this.dataForm.applicationId];
              var orderCount = data.orderCount[this.dataForm.applicationId];
              var withdrawCashCount = data.withdrawCashCount // 提现统计
              var withdrawCoinCount = data.withdrawCoinCount // 提币统计
              this.orderCount.todayRecharge = orderCount ? orderCount.todayRecharge : '0';
              this.orderCount.todayWithdraw = orderCount ? orderCount.todayWithdraw : '0';
              this.orderCount.totalRecharge = orderCount ? orderCount.totalRecharge : '0';
              this.orderCount.totalWithdraw = orderCount ? orderCount.totalWithdraw : '0';
              this.orderCount.todayRechargeQuantity = orderCount ? orderCount.todayRechargeQuantity : '0';
              this.orderCount.todayWithdrawQuantity = orderCount ? orderCount.todayWithdrawQuantity : '0';
              this.orderCount.totalWithdrawCashCny = withdrawCashCount ? withdrawCashCount.totalWithdrawCashCny : '0'
              this.orderCount.totalWithdrawCoin = withdrawCoinCount ? withdrawCoinCount.arriveWithdrawCoin : '0'
              this.totalCount = totalCount ? totalCount : '0';
              this.todayLoginCount = todayLoginCount ? todayLoginCount : '0';
              //获取消息列表
            this.getMessageList()
            }
          }).then(() => {
            if (this.dataForm.applicationId) {
              this.$http({
                url: this.$http.adornUrl('/payapplication/projectBalance/' + this.dataForm.applicationId),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  var coin = data.digitalInfo
                  this.totalBalance = coin.total ? coin.total : '0.00'
                  this.totalCnyBalance = coin.total_cny ? coin.total_cny : '0.00'
                } else {
                  this.$message.error(data.msg)
                }
              })
            }
          })


        }
      },
      getMessageList(){
        this.$http({
          url: this.$http.adornUrl('/paychatmsg/getBackUserMessage'),
          method: 'get',
          params: this.$http.adornParams({
            'pageIndex': 1,
            'pageSize': 1,
            'applicationId': this.dataForm.applicationId
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.msgTotal=data.data.msgTotal
          } else {
            this.$alert(data.msg, '提示', {
              confirmButtonText: '确定',
            });
          }
        })
      },
      openMemberList () {
        this.$router.push({path: '/members-list', query:{applicationId: this.dataForm.applicationId}});
      },
      openRechargeOrderList () {
        this.$router.push({path: '/recharge-order-list', query:{applicationId: this.dataForm.applicationId}});
      },
      openWithdrawOrderList () {
        this.$router.push({path: '/withdraw-order-list', query:{applicationId: this.dataForm.applicationId, otcBuyCoin: this.dataForm.otcBuyCoin}});
      },
      openWithdrawCashOrderList () {
        // 提现记录
        this.$router.push({path: '/withdraw-cash-order-list', query: {applicationId: this.dataForm.applicationId}})
      },
      openWithdrawCoinOrderList () {
        // 提币记录
        this.$router.push({path: '/withdraw-coin-order-list', query: {applicationId: this.dataForm.applicationId}})
      },
      // 消息中心
      msglist () {
        this.$router.push({path: '/message-centre-list', query: {applicationId: this.dataForm.applicationId}})
      }
    }
  }
</script>
