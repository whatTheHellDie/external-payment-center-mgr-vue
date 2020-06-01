<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.applicationName" placeholder="应用名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.projectAccount" placeholder="项目方" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('app:payapplication:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('app:payapplication:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle" style="width: 100%;">
      <el-table-column type="selection" header-align="center" align="center" width="40">
      </el-table-column>
      <el-table-column prop="id" header-align="center" align="center" width="50" type="index" :index="1" label="序号">
      </el-table-column>
      <el-table-column prop="applicationName" header-align="center" align="left" width="390" label="应用">
        <template slot-scope="scope">
          应用名：<span>{{scope.row.applicationName}}</span>
          <br /> app_key：
          <span>{{scope.row.applicationKey}}</span>
          <br /> app_secret：
          <span>{{scope.row.applicationSecret}}</span>
        </template>
      </el-table-column>
      <el-table-column prop="projectAccount" header-align="left" align="left" label="项目方">
        <template slot-scope="scope">
          &nbsp;&nbsp;&nbsp;&nbsp; 姓&nbsp;&nbsp;名：
          <span>{{scope.row.ownerName}}</span>
          <br /> 币利账户：
          <span>{{scope.row.projectAccount}}</span>
        </template>
      </el-table-column>
      <el-table-column header-align="center" align="left" label="会员数">
        <template slot-scope="scope">
          今日登录：<span>{{scope.row.todayLoginCount}}</span>
          <br /> 会员总数：
          <span>{{scope.row.totalCount}}</span>
        </template>
      </el-table-column>
      <el-table-column prop="mobile" header-align="center" align="left" label="数据统计">
        <template slot-scope="scope">
          今日充币：<span>{{scope.row.todayRecharge}}（CNY）</span>
          <br /> 今日提币：
          <span>{{scope.row.todayWithdraw}}（个）</span>
        </template>
      </el-table-column>
      <el-table-column prop="smsResidueNum" header-align="center" align="center" label="剩余可用短信">
        <template slot-scope="scope">
          <span v-if="scope.row.smsResidueNum">{{scope.row.smsResidueNum}}</span>
          <span v-else>0</span> &nbsp;&nbsp;
          <el-button v-if="isAuth('app:payapplication:addsms')" type="primary" size="mini" @click="addSmsNum(scope.row.applicationId)">增加</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="status" header-align="center" align="center" width="70" label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">禁用</el-tag>
          <el-tag v-else size="small">正常</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" header-align="center" align="center" width="160" label="创建时间">
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="200" label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('app:payapplication:info')" type="text" size="small" @click="showInfoHandle(scope.row.applicationId)">查看</el-button>
          <el-button v-if="isAuth('app:payapplication:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.applicationId)">修改</el-button>
          <!--<el-button v-if="isAuth('app:payapplicationreceipt:update')" type="text" size="small" @click="applicationReceipt(scope.row.applicationId)">收款方式</el-button>-->
          <el-button v-if="isAuth('app:payapplication:delete')" type="text" size="small" @click="deleteHandle(scope.row.applicationId,scope.row.applicationName)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="phone-table">
      <div class="tr" v-for="item,i in dataList">
        <!--<phone-column :prop="i+1" label="序号"></phone-column>-->
        <phone-column label="应用" :data="dataList" :index="i">
          <template slot-scope="scope">
            应用名：<span>{{scope.row.applicationName}}</span>
            <br /> app_key：
            <span>{{scope.row.applicationKey}}</span>
            <br /> app_secret：
            <span>{{scope.row.applicationSecret}}</span>
          </template>
        </phone-column>

         <phone-column label="项目方" :data="dataList" :index="i">
           <template slot-scope="scope">
          &nbsp;&nbsp;&nbsp;&nbsp; 姓&nbsp;&nbsp;名：
          <span>{{scope.row.ownerName}}</span>
          <br /> 币利账户：
          <span>{{scope.row.projectAccount}}</span>
        </template>
        </phone-column>
        <phone-column label="会员数" :data="dataList" :index="i">
           <template slot-scope="scope">
          今日登录：<span>{{scope.row.todayLoginCount}}</span>
          <br /> 会员总数：
          <span>{{scope.row.totalCount}}</span>
        </template>
        </phone-column>
       <phone-column label="数据统计" :data="dataList" :index="i">
           <template slot-scope="scope">
          今日充币：<span>{{scope.row.todayRecharge}}（CNY）</span>
          <br /> 今日提币：
          <span>{{scope.row.todayWithdraw}}（个）</span>
        </template>
        </phone-column>
        <phone-column label="剩余可用短信" :data="dataList" :index="i">
           <template slot-scope="scope">
          <span v-if="scope.row.smsResidueNum">{{scope.row.smsResidueNum}}</span>
          <span v-else>0</span> &nbsp;&nbsp;
          <el-button v-if="isAuth('app:payapplication:addsms')" type="primary" size="mini" @click="addSmsNum(scope.row.applicationId)">增加</el-button>
        </template>
        </phone-column>
        <phone-column label="状态" :data="dataList" :index="i">
           <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">禁用</el-tag>
          <el-tag v-else size="small">正常</el-tag>
        </template>
        </phone-column>

        <phone-column :prop="item.createTime" label="创建时间"></phone-column>

        <phone-column label="操作" :data="dataList" :index="i">
          <template slot-scope="scope">
          <el-button v-if="isAuth('app:payapplication:info')" type="text" size="small" @click="showInfoHandle(scope.row.applicationId)">查看</el-button>
          <el-button v-if="isAuth('app:payapplication:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.applicationId)">修改</el-button>
          <el-button v-if="isAuth('app:payapplicationreceipt:update')" type="text" size="small" @click="applicationReceipt(scope.row.applicationId)">收款方式</el-button>
          <el-button v-if="isAuth('app:payapplication:delete')" type="text" size="small" @click="deleteHandle(scope.row.applicationId,scope.row.applicationName)">删除</el-button>
        </template>
        </phone-column>
      </div>
      <div v-if="dataList.length==0">
        <img src="/static/img/no-data.jpg" class="no-data" alt="暂无数据" />
      </div>
    </div>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex" :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage" layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>

    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <!-- 查看 -->
    <show-info v-if="showInfoVisible" ref="showInfo" @refreshDataList="getDataList"></show-info>
    <!-- 添加短息条数 -->
    <add-smsnum v-if="addSmsnumVisible" ref="addSmsnum" @refreshDataList="getDataList"></add-smsnum>
    <!--支付方式-->
    <pay-method v-if="showPayMethod" ref="showPayMethodRef"></pay-method>
  </div>
</template>
<script>
  import Vue from 'vue'
  import AddOrUpdate from './application-add-or-update'
  import ShowInfo from './application-show-info'
  import addSmsnum from './application-add-smsnum'
  import payMethod from './payment-method'
  import { log } from 'util';
  export default {
    data() {
      return {
        showPayMethod: false,
        dataForm: {
          applicationName: '',
          projectAccount: ''
        },
        applicationReceiptForm: {
          applicationId: '',
          receiveType: '',
          receiveTypeName: '',
          trueName: '',
          account: '',
          receivePicUrl: '',
          bankName: '',
          bankBranchName: '',
          applicationReceiptId: '',
        },
        applicationReceiptList: {
          wxReceiptId: '',
          wxReceiptName: '',
          wxReceiptAccount: '',
          zfbReceiptId: '',
          zfbReceiptName: '',
          zfbReceiptAcount: '',
          bankReceiptId: '',
          bankReceiptName: '',
          bankReceiptAccount: ''

        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        applicationReceiptObject: {},
        headersParams: {
          token: Vue.cookie.get('token')
        },
        dataListLoading: false,
        dataListSelections: [],
        showInfoVisible: false,
        addSmsnumVisible: false,
        addOrUpdateVisible: false,
        bankReceiptVisible: false,
        applicationReceiptVisible: false,
        dataRule: {
          trueName: [{
            required: true,
            message: '收款方真实姓名不能为空',
            trigger: 'blur'
          }],
          account: [{
            required: true,
            message: '收款账号不能为空',
            trigger: 'blur'
          }],
          receivePicUrl: [{
            required: true,
            message: '333',
            trigger: 'blur'
          }]
        }
      }
    },
    components: {
      ShowInfo,
      addSmsnum,
      AddOrUpdate,
      payMethod
    },
    activated() {
      this.getDataList()
    },
    deactivated() {
    this.addOrUpdateVisible=false
    this.showInfoVisible=false
    this.addSmsnumVisible=false
    this.showPayMethod=false
    },
    methods: {
      // 获取数据列表
      getDataList() {
        this.dataListLoading = true;
        this.bankReceiptVisible = false;
        this.applicationReceiptVisible = false;
        this.$http({
          url: this.$http.adornUrl('/payapplication/list'),
          method: 'post',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'applicationName': this.dataForm.applicationName,
            'projectAccount': this.dataForm.projectAccount
          })
        }).then(({
          data
        }) => {
          if(data && data.code === 0) {
            var orderCount = data.orderCount;
            var totalCountData = data.totalCount;
            var todayLoginCountData = data.todayLoginCount;
            this.dataList = data.page.list
            for(var i = 0; i < this.dataList.length; i++) {
              var applicationId = this.dataList[i].applicationId;
              var totalCount = totalCountData[applicationId];
              var todayLoginCount = todayLoginCountData[applicationId];
              this.dataList[i].totalCount = totalCount ? totalCount : '0';
              this.dataList[i].todayLoginCount = todayLoginCount ? todayLoginCount : '0';

              var orderData = orderCount[applicationId];
              this.dataList[i].todayRecharge = orderData ? orderData.todayRecharge : '0';
              this.dataList[i].todayWithdraw = orderData ? orderData.todayWithdraw : '0';
            }

            //this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle(val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle(val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle(val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle(id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 添加短信条数
      addSmsNum(id) {
        this.addSmsnumVisible = true;
        this.$nextTick(() => {
          this.$refs.addSmsnum.init(id)
        })
      },
      // 查看
      showInfoHandle(id) {
        this.showInfoVisible = true;
        this.$nextTick(() => {
          this.$refs.showInfo.init(id)
        })
      },
      // 打开收款方式窗口
      applicationReceipt(applicationId) {
        if(!applicationId) {
          this.$message.error('应用编号参数无效!');
        }
        this.showPayMethod = true
        this.$nextTick(() => {
          this.$refs.showPayMethodRef.init(applicationId)
        })
        //      this.applicationReceiptForm.applicationId = applicationId;
        //      this.applicationReceiptVisible = true;
        //      this.applicationReceiptList.zfbReceiptName =  '';
        //      this.applicationReceiptList.zfbReceiptAcount =  '';
        //      this.applicationReceiptList.zfbReceiptId = '';
        //      this.applicationReceiptList.wxReceiptName = '';
        //      this.applicationReceiptList.wxReceiptAccount = '';
        //      this.applicationReceiptList.wxReceiptId = '';
        //      this.applicationReceiptList.bankReceiptName =  '';
        //      this.applicationReceiptList.bankReceiptAccount = '';
        //      this.applicationReceiptList.bankReceiptId = '';
        //      this.applicationReceiptObject = {};
        //      this.$http({
        //            url: this.$http.adornUrl(`/payapplicationreceipt/list`),
        //            method: 'post',
        //            data: this.$http.adornData({'applicationId': applicationId})
        //          }).then(({data}) => {
        //            if (data && data.length > 0) {
        //              var self = this;
        //              data.forEach(function(v){
        //                self.applicationReceiptObject[v.receiveType] = v;
        //                if (v.receiveType === 1) {
        //                  self.applicationReceiptList.zfbReceiptName = v.trueName ? v.trueName : '';
        //                  self.applicationReceiptList.zfbReceiptAcount = v.account ? v.account : '';
        //                  self.applicationReceiptList.zfbReceiptId = v.applicationReceiptId ? v.applicationReceiptId : '';
        //                }
        //                if (v.receiveType === 2) {
        //                  self.applicationReceiptList.wxReceiptName = v.trueName ? v.trueName : '';
        //                  self.applicationReceiptList.wxReceiptAccount = v.account ? v.account : '';
        //                  self.applicationReceiptList.wxReceiptId = v.applicationReceiptId ? v.applicationReceiptId : '';
        //                }
        //                if (v.receiveType === 3) {
        //                  self.applicationReceiptList.bankReceiptName = v.trueName ? v.trueName : '';
        //                  self.applicationReceiptList.bankReceiptAccount = v.account ? v.account : '';
        //                  self.applicationReceiptList.bankReceiptId = v.applicationReceiptId ? v.applicationReceiptId : '';
        //                }
        //              });
        //            }
        //          })
      },
      // 监听图片上传成功后的方法
      handleAvatarSuccess(res, file) {
        this.imageUrl = URL.createObjectURL(file.raw);
      },
      // 监听图片上传前的方法
      beforeAvatarUpload(file) {
        const isJPG = true; //file.type === 'image/jpeg';
        const isLt2M = file.size / 1024 / 1024 < 2;

        //if (!isJPG) {
        //  this.$message.error('上传头像图片只能是 JPG 格式!');
        //}
        //if (!isLt2M) {
        //  this.$message.error('上传头像图片大小不能超过 2MB!');
        //}
        this.getBase64(file);
        return false; // isJPG && isLt2M;
      },
      // 将上传图片转为base64数据
      getBase64(file) {
        let self = this;
        return new Promise(function(resolve, reject) {
          let reader = new FileReader();
          let imgResult = "";
          reader.readAsDataURL(file);
          reader.onload = function() {
            self.applicationReceiptForm.receivePicUrl = reader.result;
          };
          reader.onerror = function(error) {
            this.$message.error('图片上传失败!');
          };
          reader.onloadend = function() {
            console.log("图片加载完成");
          };
        });
      },
      // 删除
      deleteHandle(id, applicationName) {
        var applicationIds = id ? [id] : this.dataListSelections.map(item => {
          return item.applicationId
        })
        var applicationNames = applicationName ? [applicationName] : this.dataListSelections.map(item => {
          return item.applicationName
        })
        this.$confirm(`确定对[${applicationNames.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/payapplication/delete'),
            method: 'post',
            data: this.$http.adornData(applicationIds, false)
          }).then(({
            data
          }) => {
            if(data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      }
    }
  }
</script>
<style>
  .receipt_left {
    margin-left: 30px
  }

  .el-card__body {
    padding: 0px
  }

  .wew {
    height: 90%;
  }

  .wew .el-dialog__body {
    height: 86.5%;
    overflow-y: auto;
  }

  .el-dialog__header {
    border-bottom: 1px solid #ccc;
  }

  .el-dialog__footer {
    border-top: 1px solid #ccc;
  }

  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }

  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }

  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 150px;
    height: 150px;
    line-height: 150px;
    text-align: center;
  }

  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>
