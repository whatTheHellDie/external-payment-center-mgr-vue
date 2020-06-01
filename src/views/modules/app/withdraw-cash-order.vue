<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="会员账号">
        <el-input v-model="dataForm.memberAccount" placeholder="会员账号" clearable></el-input>
      </el-form-item>
      <el-form-item label="提现对象">
        <el-select v-model="dataForm.flag" placeholder="提现对象">
          <el-option
            v-for="item in flagType"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="状态">
        <el-select v-model="dataForm.orderStatus" placeholder="状态">
          <el-option
            v-for="item in orderStatusOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="提现时间">
        <el-date-picker v-model="dataForm.startCreateTime" format="yyyy-MM-dd" value-format="yyyy-MM-dd" align="right" type="date" placeholder="开始时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="-">
        <el-date-picker v-model="dataForm.endCreateTime" format="yyyy-MM-dd" value-format="yyyy-MM-dd" align="right" type="date" placeholder="结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button @click="exportExcel()" type="info">导出</el-button>
      </el-form-item>
    </el-form>

    <el-tag style="margin-bottom: 10px">提现数量总计：{{totalWithdrawQuantity}}</el-tag>
    <el-tag style="margin-bottom: 10px; margin-left: 10px">到账金额总计：{{totalCashCny}}</el-tag>

    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column prop="id" header-align="center" align="center" width="80" type="index" :index="1" label="序号">
      </el-table-column>
      <el-table-column prop="createTime" header-align="center" align="center" width="200" label="提现时间">
      </el-table-column>
      <el-table-column prop="flag" header-align="center" align="center" width="200" label="提现对象">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.flag === 0" size="small">用户</el-tag>
          <el-tag v-if="scope.row.flag === 1" size="small">项目方</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="memberAccount" header-align="center" align="center" label="会员账号">
      </el-table-column>
      <el-table-column prop="withdrawCoin" header-align="center" align="center" label="提现类型">
      </el-table-column>
      <el-table-column prop="withdrawAmount" header-align="center" align="center" label="提现数量（个）">
      </el-table-column>
      <el-table-column prop="actualArrivalAmount" header-align="center" align="center" label="到账金额（CNY）">
      </el-table-column>
      <el-table-column prop="receiveType" header-align="center" align="center" label="收款方式">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.receiveType === 1" size="small">支付宝</el-tag>
          <el-tag v-if="scope.row.receiveType === 2" size="small">微信</el-tag>
          <el-tag v-if="scope.row.receiveType === 3" size="small">银行卡</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="orderStatus" header-align="center" align="center" label="状态">
        <template slot-scope="scope">
          <div v-if="scope.row.orderStatus === '100'">
            <el-tag size="small">待收款</el-tag>
            <el-button v-if="scope.row.flag === 1" type="text" class="mechant-btn" @click="toContactMerchant(scope.row.orderNo)">联系商家</el-button>
          </div>
          <el-tag v-if="scope.row.orderStatus === '200'" size="small" type="success">提现成功</el-tag>
          <el-tag v-if="scope.row.orderStatus === '104'" size="small" type="info">超时取消</el-tag>
          <el-tag v-if="scope.row.orderStatus === '999'" size="small" type="info">交易关闭</el-tag>
        </template>
      </el-table-column>
    </el-table>

    <!--适配手机-->
    <div class="phone-table">
      <div class="tr" v-for="item,i in dataList">

        <phone-column :prop="item.createTime" label="提现时间"></phone-column>

        <phone-column  label="提现对象" :data="dataList" :index="i">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.flag === 0" size="small">用户</el-tag>
            <el-tag v-if="scope.row.flag === 1" size="small">项目方</el-tag>
          </template>
        </phone-column>

        <phone-column :prop="item.memberAccount" label="会员账号"></phone-column>
        <phone-column :prop="item.withdrawCoin" label="提现类型"></phone-column>
        <phone-column :prop="item.withdrawAmount" label="提现数量（个）"></phone-column>
        <phone-column :prop="item.actualArrivalAmount" label="到账金额（CNY）"></phone-column>

        <phone-column :data="dataList" :index="i" label="收款方式">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.receiveType === 1" size="small">支付宝</el-tag>
            <el-tag v-if="scope.row.receiveType === 2" size="small">微信</el-tag>
            <el-tag v-if="scope.row.receiveType === 3" size="small">银行卡</el-tag>
          </template>
        </phone-column>
        <phone-column :data="dataList" :index="i" label="状态">
          <template slot-scope="scope">
            <div v-if="scope.row.orderStatus === '100'">
              <el-tag size="small">待收款</el-tag>
              <el-button v-if="scope.row.flag === 1" type="text" class="mechant-btn" @click="toContactMerchant(scope.row.orderNo)">联系商家</el-button>
            </div>
            <el-tag v-if="scope.row.orderStatus === '200'" size="small" type="success">提现成功</el-tag>
            <el-tag v-if="scope.row.orderStatus === '104'" size="small" type="info">超时取消</el-tag>
            <el-tag v-if="scope.row.orderStatus === '999'" size="small" type="info">交易关闭</el-tag>
          </template>
        </phone-column>

      </div>
    </div>

    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
     <!--联系商家-->
    <contact-merchant v-if="showcontactMerchant" ref="showcontactMerchantRef"></contact-merchant>
  </div>
</template>

<script>
    import FileSaver from 'file-saver'
    import XLSX from 'xlsx'
    import contactMerchant from './contactMerchant'
    export default {
      data () {
        return {
          showcontactMerchant:false,
          dataForm: {
            memberAccount: '',
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
          totalCashCny: 0,
          dataListLoading: false,
          orderStatusOptions: [
            {value: '', label: '全部'},
            {value: '100', label: '待收款'},
            {value: '104', label: '超时取消'},
            {value: '200', label: '提现成功'},
            {value: '999', label: '交易关闭'}
          ],
          flagType: [
            {value: '', label: '全部'},
            {value: '1', label: '项目方'},
            {value: '0', label: '用户'}
          ]
        }
      },
      activated () {
        this.getDataList()
      },
      components:{
        contactMerchant
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
          if (this.$route.query.applicationId) {
            this.applicationId = this.$route.query.applicationId
          }
          this.$http({
            url: this.$http.adornUrl('/payorder/withdraw/list'),
            method: 'get',
            params: this.$http.adornParams({
              'page': this.pageIndex,
              'limit': this.pageSize,
              'applicationId': this.applicationId,
              'orderStatus': this.dataForm.orderStatus,
              'memberAccount': this.dataForm.memberAccount,
              'startCreateTime': this.dataForm.startCreateTime,
              'endCreateTime': this.dataForm.endCreateTime,
              'flag': this.dataForm.flag
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataList = data.page.list
              this.totalPage = data.page.totalCount
            } else {
              this.dataList = []
              this.totalPage = 0
            }
            this.dataListLoading = false
          }).then(() => {
            if (this.applicationId) {
              this.$http({
                url: this.$http.adornUrl(`/payorder/withdraw/count`),
                method: 'get',
                params: this.$http.adornParams({
                  'applicationId': this.applicationId,
                  'orderStatus': this.dataForm.orderStatus,
                  'memberAccount': this.dataForm.memberAccount,
                  'startCreateTime': this.dataForm.startCreateTime,
                  'endCreateTime': this.dataForm.endCreateTime,
                  'flag': this.dataForm.flag
                })
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.totalWithdrawQuantity = data.totalWithdrawQuantity
                  this.totalCashCny = data.totalCashCny
                }
              })
            }
          })
        },
        // 导出excel
        exportExcel () {
          this.$http({
            url: this.$http.adornUrl('/payorder/withdrawcash/excel'),
            method: 'post',
            params: this.$http.adornParams({
              'applicationId': this.applicationId,
              'orderStatus': this.dataForm.orderStatus,
              'memberAccount': this.dataForm.memberAccount,
              'startCreateTime': this.dataForm.startCreateTime,
              'endCreateTime': this.dataForm.endCreateTime,
              'flag': this.dataForm.flag
            })
          }).then(function (data) {
            if (!data.data || !data.data.excelData) {
              return
            }
            const wb = { SheetNames: ['Sheet1'], Sheets: {}, Props: {} }
            // 通过json_to_sheet转成单页(Sheet)数据
            wb.Sheets['Sheet1'] = XLSX.utils.json_to_sheet(data.data.excelData)
            const wbout = XLSX.write(wb, { bookType: 'xlsx', bookSST: true, type: 'array' })
            try {
              FileSaver.saveAs(new Blob([wbout], { type: 'application/octet-stream' }), new Date().toLocaleDateString() + '_提现记录.xlsx')
            } catch (e) {
              if (typeof console !== 'undefined') {
                console.log(e, wbout)
              }
            }
            return wbout
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
