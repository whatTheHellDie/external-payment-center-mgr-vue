<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="会员账号">
        <el-input v-model="dataForm.memberAccount" placeholder="会员账号" clearable></el-input>
      </el-form-item>
      <el-form-item label="提币时间">
        <el-date-picker v-model="dataForm.startCreateTime" format="yyyy-MM-dd" value-format="yyyy-MM-dd" align="right" type="date" placeholder="开始时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="-">
        <el-date-picker v-model="dataForm.endCreateTime" format="yyyy-MM-dd" value-format="yyyy-MM-dd" align="right" type="date" placeholder="结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <!--<el-button @click="exportExcel()" type="info">导出</el-button>-->
      </el-form-item>
    </el-form>

    <el-tag style="margin-bottom: 10px">提币数量总计：{{totalWithdrawCoin}}</el-tag>
    <el-tag style="margin-bottom: 10px; margin-left: 10px">到账数量总计：{{arriveWithdrawCoin}}</el-tag>

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
      <el-table-column prop="coin" header-align="center" align="center" label="币种">
      </el-table-column>
      <el-table-column prop="num" header-align="center" align="center" label="提币数量（个）">
      </el-table-column>
      <el-table-column prop="status" header-align="center" align="center" label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === '0'" size="small" type="info">待审核</el-tag>
          <el-tag v-if="scope.row.status === '1'" size="small" type="success">交易完成</el-tag>
          <el-tag v-if="scope.row.status === '2'" size="small" type="info">订单已拒绝</el-tag>
          <el-tag v-if="scope.row.status === '3'" size="small" type="info">订单已锁定</el-tag>
          <el-tag v-if="scope.row.status === '4'" size="small" type="info">订单已撤销</el-tag>
        </template>
      </el-table-column>
    </el-table>

    <!--适配手机-->
    <div class="phone-table">
      <div class="tr" v-for="item,i in dataList">
        <phone-column :prop="item.createTime" label="提币时间"></phone-column>
        <phone-column :prop="item.memberAccount" label="会员账号"></phone-column>
        <phone-column :prop="item.coin" label="币种"></phone-column>
        <phone-column :prop="item.num" label="提币数量（个）"></phone-column>

        <phone-column :data="dataList" :index="i" label="状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status === '0'" size="small" type="info">待审核</el-tag>
            <el-tag v-if="scope.row.status === '1'" size="small" type="success">交易完成</el-tag>
            <el-tag v-if="scope.row.status === '2'" size="small" type="info">订单已拒绝</el-tag>
            <el-tag v-if="scope.row.status === '3'" size="small" type="info">订单已锁定</el-tag>
            <el-tag v-if="scope.row.status === '4'" size="small" type="info">订单已撤销</el-tag>
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
            memberAccount: '',
            startCreateTime: null,
            endCreateTime: null
          },
          dataList: [],
          pageIndex: 1,
          pageSize: 10,
          totalPage: 0,
          applicationId: '',
          totalWithdrawCoin: 0, // 总提币数量
          arriveWithdrawCoin: 0, // 到账提币数量
          dataListLoading: false
        }
      },
      activated () {
        this.getDataList()
      },
      methods: {
        // 获取数据列表
        getDataList () {
          this.dataListLoading = true
          if (this.$route.query.applicationId) {
            this.applicationId = this.$route.query.applicationId
          }
          this.$http({
            url: this.$http.adornUrl('/sys/withdraw/coin/orderList'),
            method: 'get',
            params: this.$http.adornParams({
              'page': this.pageIndex,
              'limit': this.pageSize,
              'applicationId': this.applicationId,
              'memberAccount': this.dataForm.memberAccount,
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
                url: this.$http.adornUrl(`/sys/withdraw/coin/count`),
                method: 'post',
                data: this.$http.adornData({
                  'applicationId': this.applicationId,
                  'memberAccount': this.dataForm.memberAccount,
                  'startCreateTime': this.dataForm.startCreateTime,
                  'endCreateTime': this.dataForm.endCreateTime
                })
              }).then(({data}) => {
                if (data && data.code === 200) {
                  this.totalWithdrawCoin = data.data.totalWithdrawCoin
                  this.arriveWithdrawCoin = data.data.arriveWithdrawCoin
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
