<template>
  <div>

    <ul class="message-list">
      <li class="item" v-for="item in dataList" :key="item.id"  @click="toContactMerchant(item.orderSn)">
        <div class="left">
          <img class="img" src="~@/assets/img/messeges2.png"/>
          <span class="span">{{item.otcOrderSn}}</span>
        </div>
        <div class="right">
          <span>{{item.unRead}}</span>
        </div>
      </li>
    </ul>
    <div v-if="noMessage">
        <img src="/static/img/no-data.jpg" class="no-data" alt="暂无数据" />
      </div>
     <!--联系商家-->
    <contact-merchant v-if="showcontactMerchant" ref="showcontactMerchantRef"></contact-merchant>
  </div>
</template>
<style>
  ul{
    list-style-type: none;
  }
  .message-list {
    width: 100%;
    padding: 0;
    margin: 0 ;
  }
   .message-list .img{
    width: 24px;
    vertical-align: top;
  }
  .message-list .item {
    border-bottom: 1px solid #ccc;
    margin-bottom: 20px;
    padding-bottom: 5px;
  }
  .message-list .item::after {
    content: "";
    display: block;
    clear: both;
    height: 0;
    visibility: hidden;
  }
  .message-list .item .left {
    float: left;
    font-size: 0;
  }
  .message-list .item .left .span {
    vertical-align: 0;
    font-size: 18px;
    margin-left: 5px;
    
  }
  .message-list .item .right {
    float: right;
    font-size: 14px;
  }
</style>

<script>
  import contactMerchant from './contactMerchant'
  import FileSaver from 'file-saver'
  import XLSX from 'xlsx'
  export default {
    data () {
      return {
        noMessage:false,
        showcontactMerchant:false,
        dataForm: {
          orderSn: '',
          withdrawCoin: '',
          withdrawAmount: '',
          actualArrivalAmount: '',
          receiveType: '',
          orderStatus: '',
          flag: '',
          startCreateTime: null,
          endCreateTime: null
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        applicationId: '',
        totalWithdrawQuantity: 0,
        totalWithdrawCashCny: 0,
        dataListLoading: false
      }
    },
    components:{
        contactMerchant
      },
    activated () {
      this.noMessage=false
      this.getDataList()
    },
    methods: {
      toContactMerchant(orderSn){
          this.$axios({
          url: this.$http.adornUrl('/paychatmsg/contact'),
          method: 'post',
          //发送格式为json
          data: JSON.stringify({
            "orderSn": orderSn,
          }),
          headers: {
            'Content-Type': 'application/json; charset=utf-8',
            'token': this.$cookie.get('token')
          }
        }).then((data) => {
          if(data.data.code == 200) {
             this.showcontactMerchant = true
        this.$nextTick(() => {
          this.$refs.showcontactMerchantRef.init({chat:data.data.data,applicationId:this.applicationId})
        })
//          this.$router.push({
//            name: 'contactMerchant',
//            params: {
//              chat: data.data.data
//            }
//          })
          } else {
            this.$alert(data.data.msg, '提示', {
              confirmButtonText: '确定',
            });
          }
        });
        },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.applicationId = this.$route.query.applicationId
        this.$http({
          url: this.$http.adornUrl('/paychatmsg/getBackUserMessage'),
          method: 'get',
          params: this.$http.adornParams({
            'pageIndex': this.pageIndex,
            'pageSize': this.pageSize,
            'applicationId': this.applicationId
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.dataList = data.data.list
            if(data.data.list.length===0){
              this.noMessage=true;
            }
          } else {
            this.$alert(data.msg, '提示', {
              confirmButtonText: '确定',
            });
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      }
    }
  }
</script>

<style scoped>

</style>
