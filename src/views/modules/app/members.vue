<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="会员账号">
        <el-input v-model="dataForm.memberAccount" placeholder="会员账号" clearable></el-input>
      </el-form-item>
      <el-form-item label="注册时间">
        <el-date-picker v-model="dataForm.startRegisterTime" format="yyyy-MM-dd" value-format="yyyy-MM-dd" align="right" type="date" placeholder="开始时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="-">
        <el-date-picker v-model="dataForm.endRegisterTime" format="yyyy-MM-dd" value-format="yyyy-MM-dd" align="right" type="date" placeholder="结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column prop="id" header-align="center" align="center" width="80" type="index" :index="1" label="序号">
      </el-table-column>
      <el-table-column prop="memberAccount" header-align="center" align="center" label="会员账号">
      </el-table-column>
      <el-table-column header-align="center" align="center" label="项目币">
        <template slot-scope="scope">
          <el-tag v-if="projectCurrencyType === 1" size="small">游戏币</el-tag>
          <el-tag v-if="projectCurrencyType === 2" size="small">代币</el-tag>
          <span v-if="scope.row.projectCurrencyType"></span>
        </template>
      </el-table-column>
      <el-table-column prop="balance" header-align="center" align="center" width="100" label="余额">
      </el-table-column>
      <el-table-column prop="btcNum" header-align="center" align="center" label="BTC">
      </el-table-column>
      <el-table-column prop="usdtNum" header-align="center" align="center" label="USDT">
      </el-table-column>
      <el-table-column prop="loginTime" header-align="center" align="center" width="200" label="登录时间">
      </el-table-column>
      <el-table-column prop="registerTime" header-align="center" align="center" width="200" label="注册时间">
      </el-table-column>
    </el-table>

    <div class="phone-table">
      <div class="tr" v-for="item,i in dataList">
        <phone-column :data="dataList" :index="i" label="会员账号">
          <template slot-scope="scope">
            <span>{{scope.row.memberAccount}}</span>
          </template>
        </phone-column>
        <phone-column  label="项目币">
          <template slot-scope="scope">
            <el-tag v-if="projectCurrencyType === 1" size="small">游戏币</el-tag>
            <el-tag v-if="projectCurrencyType === 2" size="small">代币</el-tag>
            <span v-if="scope.row.projectCurrencyType"></span>
          </template>
        </phone-column>
        <phone-column  label="余额" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.balance}}</span>
          </template>
        </phone-column>
        <phone-column  label="BTC" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.btcNum}}</span>
          </template>
        </phone-column>
        <phone-column  label="USDT" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.usdtNum}}</span>
          </template>
        </phone-column>
        <phone-column  label="登录时间" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.loginTime}}</span>
          </template>
        </phone-column>
        <phone-column label="注册时间" :data="dataList" :index="i">
          <template slot-scope="scope">
            <span>{{scope.row.registerTime}}</span>
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
  export default {
    data () {
      return {
        dataForm: {
          memberAccount: '',
          startRegisterTime: null,
          endRegisterTime: null
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        applicationId: '',
        projectCurrencyType: '',
        dataListLoading: false,
        dataListSelections: []
      }
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        if (!this.applicationId) {
          this.applicationId = this.$route.query.applicationId;
        }
        this.$http({
          url: this.$http.adornUrl('/paymembers/list'),
          method: 'post',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'applicationId': this.applicationId,
            'memberAccount': this.dataForm.memberAccount,
            'startRegisterTime': this.dataForm.startRegisterTime,
            'endRegisterTime': this.dataForm.endRegisterTime
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list;
            this.totalPage = data.page.totalCount;
            this.projectCurrencyType = data.projectCurrencyType;
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false;
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
