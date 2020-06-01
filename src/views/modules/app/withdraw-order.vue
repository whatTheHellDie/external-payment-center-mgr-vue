<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="会员账号">
        <el-input v-model="dataForm.memberAccount" placeholder="会员账号" clearable></el-input>
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
      <el-form-item label="提币时间">
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

    <el-tag style="margin-bottom: 10px">提币数量总计：{{totalWithdraw}}</el-tag>
    <el-tag style="margin-bottom: 10px; margin-left: 10px">到账数量总计：{{withdrawArrivalAmount}}</el-tag>

    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column prop="id" header-align="center" align="center" width="80" type="index" :index="1" label="序号">
      </el-table-column>
      <el-table-column prop="createTime" header-align="center" align="center" width="200" label="提币时间">
      </el-table-column>
      <el-table-column prop="memberAccount" header-align="center" align="center" label="会员账号">
      </el-table-column>
      <el-table-column prop="withdrawAmount" header-align="center" align="center" label="提币数量">
      </el-table-column>
      <el-table-column prop="withdrawBtcAmount" header-align="center" align="center" :label="otcBuyCoin">
      </el-table-column>
      <el-table-column prop="orderStatus" header-align="center" align="center" label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.orderStatus === '200'" size="small">提币中</el-tag>
          <el-tag v-if="scope.row.orderStatus === '201'" size="small" type="success">提币成功</el-tag>
          <el-tag v-if="scope.row.orderStatus === '104'" size="small" type="success">超时取消</el-tag>
          <el-tag v-if="scope.row.orderStatus === '202'" size="small" type="danger">提币失败</el-tag>
          <el-tag v-if="scope.row.orderStatus === '999'" size="small" type="info">交易关闭</el-tag>
        </template>
      </el-table-column>
    </el-table>

    <!--适配手机-->
    <div class="phone-table">
      <div class="tr" v-for="item,i in dataList">

        <phone-column :prop="item.createTime" label="提币时间"></phone-column>
        <phone-column :prop="item.memberAccount" label="会员账号"></phone-column>
        <phone-column :prop="item.withdrawAmount" label="提币数量"></phone-column>
        <phone-column :prop="item.withdrawBtcAmount" :label="otcBuyCoin"></phone-column>
        <phone-column :data="dataList" :index="i" label="状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.orderStatus === '200'" size="small">提币中</el-tag>
            <el-tag v-if="scope.row.orderStatus === '201'" size="small" type="success">提币成功</el-tag>
            <el-tag v-if="scope.row.orderStatus === '104'" size="small" type="success">超时取消</el-tag>
            <el-tag v-if="scope.row.orderStatus === '202'" size="small" type="danger">提币失败</el-tag>
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
          startCreateTime: null,
          endCreateTime: null
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        otcBuyCoin: '',
        applicationId:'',
        totalWithdraw: 0,
        withdrawArrivalAmount: 0,
        dataListLoading: false,
        orderStatusOptions:[
          {value:'', label:'全部'},
          {value:'200', label:'提币中'},
          {value:'201', label:'提币成功'},
          {value:'202', label:'提币失败'},
          {value:'999', label:'交易关闭'}
          ],
      }
    },
    activated () {
      if (!this.otcBuyCoin) {
        this.otcBuyCoin = "到账数量（" + this.$route.query.otcBuyCoin + "）";
        }
      this.getDataList();
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        if (this.$route.query.applicationId) {
          this.applicationId = this.$route.query.applicationId
        }
        this.$http({
          url: this.$http.adornUrl('/payorder/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'orderType': '2',
            'applicationId': this.applicationId,
            'orderStatus': this.dataForm.orderStatus,
            'memberAccount': this.dataForm.memberAccount,
            'startCreateTime': this.dataForm.startCreateTime,
            'endCreateTime': this.dataForm.endCreateTime
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list;
            this.totalPage = data.page.totalCount;
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
                'orderType': '2',
                'applicationId': this.applicationId,
                'orderStatus': this.dataForm.orderStatus,
                'memberAccount': this.dataForm.memberAccount,
                'startCreateTime': this.dataForm.startCreateTime,
                'endCreateTime': this.dataForm.endCreateTime
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.totalWithdraw = data.totalWithdraw;
                this.withdrawArrivalAmount = data.withdrawArrivalAmount;
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
            'orderType': '2',
            'applicationId': this.applicationId,
            'orderStatus': this.dataForm.orderStatus,
            'memberAccount': this.dataForm.memberAccount,
            'otcBuyCoin': this.$route.query.otcBuyCoin,
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
