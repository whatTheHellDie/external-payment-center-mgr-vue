<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="会员账号">
        <el-input v-model="dataForm.memberAccount" placeholder="会员账号" clearable></el-input>
      </el-form-item>
      <el-form-item label="订单编号">
        <el-input v-model="dataForm.orderNo" placeholder="订单编号" clearable></el-input>
      </el-form-item>
      <el-form-item label="法币订单备注">
        <el-input v-model="dataForm.otcOrderRemark" placeholder="法币订单备注" clearable></el-input>
      </el-form-item>
      <el-form-item label="第三方订单号">
        <el-input v-model="dataForm.thirdPartyOrderNo" placeholder="第三方订单号" clearable></el-input>
      </el-form-item>
      <!--<el-form-item label="状态">-->
        <!--<el-select v-model="dataForm.orderStatus" placeholder="状态">-->
          <!--<el-option-->
            <!--v-for="item in orderStatusOptions"-->
            <!--:key="item.value"-->
            <!--:label="item.label"-->
            <!--:value="item.value">-->
          <!--</el-option>-->
        <!--</el-select>-->
      <!--</el-form-item>-->
      <!--<br />-->
      <!--<el-form-item label="充币时间">-->
        <!--<el-date-picker v-model="dataForm.startCreateTime" format="yyyy-MM-dd" value-format="yyyy-MM-dd" align="right" type="date" placeholder="开始时间"></el-date-picker>-->
      <!--</el-form-item>-->
      <!--<el-form-item label="-">-->
        <!--<el-date-picker v-model="dataForm.endCreateTime" format="yyyy-MM-dd" value-format="yyyy-MM-dd" align="right" type="date" placeholder="结束时间"></el-date-picker>-->
      <!--</el-form-item>-->
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <!--<el-button @click="exportExcel()" type="info">导出</el-button>-->
      </el-form-item>
    </el-form>

    <!--<el-tag style="margin-bottom: 10px">充币金额总计：{{totalRecharge}}</el-tag>-->
    <!--<el-tag style="margin-bottom: 10px; margin-left: 10px">到账数量总计：{{rechargeArrivalAmount}}</el-tag>-->

    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column prop="id" header-align="center" align="center" width="80" type="index" :index="1" label="序号">
      </el-table-column>
      <el-table-column prop="orderNo" header-align="center" align="center" width="200" label="订单号">
      </el-table-column>

      <el-table-column prop="otcOrderRemark" header-align="center" align="center" width="200" label="法币订单备注">
      </el-table-column>
      <el-table-column prop="createTime" header-align="center" align="center" width="200" label="充币时间">
      </el-table-column>
      <el-table-column prop="thirdPartyOrderNo" header-align="center" align="center" width="200" label="第三方订单号">
      </el-table-column>
      <el-table-column prop="memberAccount" header-align="center" align="center" label="会员账号">
      </el-table-column>
      <el-table-column prop="rechargeAmount" header-align="center" align="center" label="充值金额（CNY）">
      </el-table-column>
      <el-table-column prop="arrivalAmount" header-align="center" align="center" label="到账数量">
      </el-table-column>
      <el-table-column prop="orderStatus" header-align="center" align="center" label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.orderStatus === '100'" size="small">待付款</el-tag>
          <el-tag v-if="scope.row.orderStatus === '101'" size="small" type="warning">待放币</el-tag>
          <el-tag v-if="scope.row.orderStatus === '102'" size="small" type="success">交易完成</el-tag>
          <el-tag v-if="scope.row.orderStatus === '104'" size="small" type="info">超时取消</el-tag>
          <el-tag v-if="scope.row.orderStatus === '103'" size="small" type="danger">交易失败</el-tag>
          <el-tag v-if="scope.row.orderStatus === '999'" size="small" type="info">交易关闭</el-tag>
        </template>
      </el-table-column>
      <!--<el-table-column fixed="right" header-align="center" align="center" width="200" label="操作">-->
        <!--<template slot-scope="scope">-->
          <!--<el-button v-if="scope.row.isNotifySuccess != 1 && scope.row.orderStatus === '102'" type="primary" size="small" @click="notice(scope.row.orderId)">通知第三方充币</el-button>-->
        <!--</template>-->
      <!--</el-table-column>-->
    </el-table>

    <div class="phone-table">
      <div class="tr" v-for="item,i in dataList">
        <phone-column  label="订单号" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.orderNo}}</span>
          </template>
        </phone-column>
        <phone-column  label="法币订单备注" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.otcOrderRemark}}</span>
          </template>
        </phone-column>
        <phone-column  label="充币时间" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.createTime}}</span>
          </template>
        </phone-column>
        <phone-column  label="第三方订单号" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.thirdPartyOrderNo}}</span>
          </template>
        </phone-column>
        <phone-column  label="会员账号" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.memberAccount}}</span>
          </template>
        </phone-column>
        <phone-column label="充值金额（CNY）" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.rechargeAmount}}</span>
          </template>
        </phone-column>
        <phone-column  label="到账数量" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.arrivalAmount}}</span>
          </template>
        </phone-column>
        <phone-column :data="dataList" :index="i" label="状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.orderStatus === '100'" size="small">待付款</el-tag>
            <el-tag v-if="scope.row.orderStatus === '101'" size="small" type="warning">待放币</el-tag>
            <el-tag v-if="scope.row.orderStatus === '102'" size="small" type="success">交易完成</el-tag>
            <el-tag v-if="scope.row.orderStatus === '104'" size="small" type="info">超时取消</el-tag>
            <el-tag v-if="scope.row.orderStatus === '103'" size="small" type="danger">交易失败</el-tag>
            <el-tag v-if="scope.row.orderStatus === '999'" size="small" type="info">交易关闭</el-tag>
          </template>
        </phone-column>
        <!--<phone-column :data="dataList" :index="i" label="操作">-->
          <!--<template slot-scope="scope">-->
            <!--<el-button v-if="scope.row.isNotifySuccess != 1 && scope.row.orderStatus === '102'" type="primary" size="small" @click="notice(scope.row.orderId)">通知第三方充币</el-button>-->
          <!--</template>-->
        <!--</phone-column>-->
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
  </div>
</template>

<script>
import FileSaver from 'file-saver'
import XLSX from 'xlsx'
  export default {
    data () {
      return {
        dataForm: {
          orderStatus: '',
          memberAccount: '',
          orderNo: '',
          thirdPartyOrderNo: '', // 第三方订单号
          otcOrderRemark: '', // 法币订单备注
          startCreateTime: null,
          endCreateTime: null
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        applicationId: '',
        dataListLoading: false,
        totalRecharge: 0,
        rechargeArrivalAmount: 0,
        orderStatusOptions:[
          {value:'', label:'全部'},
          {value:'100', label:'待付款'},
          {value:'101', label:'待收币'},
          {value:'102', label:'交易完成'},
          {value:'104', label:'超时取消'},
          {value:'103', label:'交易失败'},
          {value:'999', label:'交易关闭'}
          ],
      }
    },
    activated () {
      // this.getDataList();
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        if (!this.applicationId) {
          this.applicationId = this.$route.query.applicationId;
        }
        this.$http({
          url: this.$http.adornUrl('/payorder/acceptanceList'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'orderType': '1',
            'applicationId': '',
            'orderStatus': this.dataForm.orderStatus,
            'memberAccount': this.dataForm.memberAccount,
            'orderNo': this.dataForm.orderNo,
            'thirdPartyOrderNo': this.dataForm.thirdPartyOrderNo,
            'otcOrderRemark': this.dataForm.otcOrderRemark,
            'startCreateTime': this.dataForm.startCreateTime,
            'endCreateTime': this.dataForm.endCreateTime
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
              url: this.$http.adornUrl(`/payorder/count`),
              method: 'get',
              params: this.$http.adornParams({
                'orderType': '1',
                'applicationId': this.applicationId
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.totalRecharge = data.totalRecharge;
                this.rechargeArrivalAmount = data.rechargeArrivalAmount;
              }
            })
          }
        })
      },
      // 导出excel
      exportExcel () {
        let _this = this;
        this.$http({
          url: this.$http.adornUrl('/payorder/excel'),
          method: 'post',
          params: this.$http.adornParams({
            'orderType': '1',
            'applicationId': this.applicationId,
            'orderStatus': this.dataForm.orderStatus,
            'memberAccount': this.dataForm.memberAccount,
            'startCreateTime': this.dataForm.startCreateTime,
            'endCreateTime': this.dataForm.endCreateTime
          })
        }).then(function (data) {
          if (!data.data || !data.data.excelData) {
            return;
          }
          const wb = { SheetNames: ['Sheet1'], Sheets: {}, Props: {} };
          //通过json_to_sheet转成单页(Sheet)数据
          wb.Sheets['Sheet1'] = XLSX.utils.json_to_sheet(data.data.excelData);
          const wbout = XLSX.write(wb, { bookType: 'xlsx', bookSST: true, type: 'array' });
          try {
            FileSaver.saveAs(new Blob([wbout], { type: 'application/octet-stream' }), new Date().toLocaleDateString() + '.xlsx');
          } catch (e) {
            if (typeof console !== 'undefined')
              console.log(e, wbout)
          }
          return wbout
        });
      },
      notice (orderId) {
        this.$confirm(`确定进行[通知第三方充币]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/payorder/notice'),
            method: 'post',
            params:this.$http.adornParams({'orderId':orderId})
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                  message: data.msg,
                  type: 'success',
                  duration: 1500
                })
                this.getDataList();
            }
          })
        }).catch(() => {})
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
