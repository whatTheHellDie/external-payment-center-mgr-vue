<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible" top="5vh" custom-class="wew">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="145px">
      <el-form-item label="应用名" prop="applicationName" :class="{ 'is-required': true }">
        <el-input v-model="dataForm.applicationName" placeholder="应用名"></el-input>
      </el-form-item>
      <el-form-item label="项目方账号" prop="projectAccount" :class="{ 'is-required': true }">
        <el-input v-model="dataForm.projectAccount" placeholder="项目方账号"></el-input>
      </el-form-item>
      <el-form-item label="项目方人名" prop="ownerName" :class="{ 'is-required': true }">
        <el-input v-model="dataForm.ownerName" placeholder="项目方人名"></el-input>
      </el-form-item>

      <el-form-item label="项目货币名" prop="projectCurrencyName" :class="{ 'is-required': true }">
        <el-input v-model="dataForm.projectCurrencyName" placeholder="项目货币名"></el-input>
      </el-form-item>

      <el-row>
        <el-col :span="12">
          <el-form-item label="项目货币类型" prop="projectCurrencyType" :class="{ 'is-required': true }">
            <el-radio-group v-model="dataForm.projectCurrencyType" @change="changeProjectCurrencyType">
              <el-radio :label="1">游戏币</el-radio>
              <el-radio :label="2">代币</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="价值" prop="projectCurrencyValue" v-if="showProjectCurrencyValue" :class="{ 'is-required': showProjectCurrencyValue }">
            <el-input v-model="dataForm.projectCurrencyValue" placeholder="价值" maxlength="8"></el-input>
          </el-form-item>
          <el-form-item label="币种标志" prop="projectCurrencyCoin"  v-if="showProjectCurrencyCoin" :class="{ 'is-required': showProjectCurrencyCoin }">
            <el-input v-model="dataForm.projectCurrencyCoin" placeholder="币种标志"></el-input>
          </el-form-item>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="12">
          <el-form-item label="下单模式" prop="orderAmountType">
            <el-radio-group v-model="dataForm.orderAmountType">
              <el-radio :label="0">按金本位</el-radio>
              <el-radio :label="1">按币本位</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="12">
          <el-form-item label="用户充币到账数量" prop="orderCalculatePrice">
            <el-radio-group v-model="dataForm.orderCalculatePrice">
              <el-radio :label="0">按otc交易计算</el-radio>
              <el-radio :label="1">按订单价格计算</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="24">
          <el-form-item label="首页返回页面" prop="skipUrl">
            <el-radio-group v-model="dataForm.skipUrl" style="width:100%">
              <el-radio :label="'0'">默认</el-radio>
              <el-radio :label="'1'" style="margin-right:10px;">指定页面</el-radio>
              <el-input style="width: 50%" v-model="dataForm.assignSkipUrl" :aria-label="dataForm.assignSkipUrl"></el-input>
            </el-radio-group>
          </el-form-item>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="12">
          <el-form-item label="项目货币小数位数" prop="projectCurrencyPoint" :class="{ 'is-required': true }">
            <el-input v-model="dataForm.projectCurrencyPoint" placeholder="项目货币小数位数"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="支持支付类型" prop="payWay">
            <el-checkbox-group v-model="dataForm.payWay">
              <el-checkbox :label="1">支付宝</el-checkbox>
              <el-checkbox :label="2">微信</el-checkbox>
              <el-checkbox :label="3">BTC</el-checkbox>
              <el-checkbox :label="4">银行卡</el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="12">
          <el-form-item label="预计到账比例(%)" prop="memberShowPercent" :class="{ 'is-required': true }">
            <el-input v-model="dataForm.memberShowPercent" placeholder="预计到账显示比例（%）"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="法币购买币种" prop="otcBuyCoin" :class="{ 'is-required': !dataForm.id }">
            <el-select v-model="dataForm.otcBuyCoin" placeholder="法币购买币种">
              <el-option
                v-for="item in otcBuyCoinOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="24">
          <el-form-item label="可选支付金额" prop="payAmount" :class="{ 'is-required': true }">
            <el-input v-model="dataForm.payAmount" placeholder="填可选支付金额格式（多个用英文逗号隔开）：100,200,300"></el-input>
          </el-form-item>
        </el-col>
      </el-row>

      <el-form-item label="充值/提币api" prop="rechargeOrWithdrawApi">
        <el-input v-model="dataForm.rechargeOrWithdrawApi" placeholder="充值/提币api" maxlength="100"></el-input>
      </el-form-item>

      <el-form-item label="查询用户余额api" prop="selectBalanceApi">
        <el-input v-model="dataForm.selectBalanceApi" placeholder="查询用户余额api" maxlength="100"></el-input>
      </el-form-item>

      <el-row>
        <el-col>
          <el-form-item label="前端显示" prop="frontDisplay">
            <el-checkbox-group v-model="dataForm.frontDisplay">
              <el-checkbox label="recharge">充币</el-checkbox>
              <el-checkbox label="withdraw">提币</el-checkbox>
              <el-checkbox label="digitalAsset">数字资产</el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-col>
      </el-row>

      <el-form-item label="是否启用" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="1">启用</el-radio>
          <el-radio :label="0">未启用</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="短信配置" prop="smsActive">
        <el-radio-group v-model="dataForm.smsActive">
          <el-radio :label="1">启用</el-radio>
          <el-radio :label="2">禁止</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="部署配置" prop="serversAddress">
        <el-radio-group v-model="dataForm.serversAddress">
          <el-radio :label="'1'">原币利</el-radio>
          <el-radio :label="'2'">OTC</el-radio>
        </el-radio-group>
      </el-form-item>

      <!--<el-form-item label="是否指定承兑商" prop="isAppointMerchant">-->
        <!--<el-radio-group v-model="dataForm.isAppointMerchant">-->
          <!--<el-radio :label="0">否</el-radio>-->
          <!--<el-radio :label="1" plain round @change="dialogMerchantAccountFormVisible=true">是</el-radio>-->
        <!--</el-radio-group>-->
      <!--</el-form-item>-->

      <!-- 承兑商账户 -->
      <!--<el-row v-for="(item,inx) in merchantListForm" :key="item.id">-->
        <!--<el-col :span="12">-->
          <!--<el-form-item label="承兑商账户：" prop="">-->
            <!--<el-input v-model="item.valueOne" ></el-input>-->
            <!--<el-button type="primary" @click="saveMerchantAccount(item.id,item.valueOne)" size="medium" circle style="position:absolute;top:-2px;right:-60px;">修改</el-button>-->
          <!--</el-form-item>-->
        <!--</el-col>-->
      <!--</el-row>-->

      <el-row v-if="dataForm.isAppointMerchant===1">
        <el-col :span="12">
          <el-form-item label="承兑商账户" prop="" >
            <el-input v-model="merchantAccount" placeholder="" readonly="true"></el-input>
            <el-button type="primary" @click="dialogMerchantAccountFormVisible=true" size="medium" circle style="position:absolute;top:-2px;right:-60px;">修改</el-button>
          </el-form-item>
        </el-col>
      </el-row>

      <el-dialog title="添加" :visible.sync="dialogMerchantAccountFormVisible" width="25%" append-to-body>
        <el-form :model="addAccountForm" label-width="90px">
          <el-form-item label="承兑商账号">
            <el-input v-model="addAccountForm.account" placeholder="输入交易所账号" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogMerchantAccountFormVisible=false">取 消</el-button>
          <el-button type="primary" @click="saveMerchantAccount()">确 定</el-button>
        </div>
      </el-dialog>


      <el-row><el-col :span="24"><div style="background: #F0F0F0; margin: 10px 0 10px 0; padding: 5px 15px">充币抽成配置</div></el-col></el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="公司账户1" prop="companyAccountOne" :class="{ 'is-required': true }">
              <el-input v-model="dataForm.companyAccountOne" placeholder="公司账户1"></el-input>
            </el-form-item>
          </el-col>
        <el-col :span="12">
          <el-form-item label="抽成（%）" prop="companyOneCommission" :class="{ 'is-required': true }">
              <el-input v-model="dataForm.companyOneCommission" placeholder="抽成" maxlength="6"></el-input>
            </el-form-item>
          </el-col>
      </el-row>

      <el-row>
        <el-col :span="12">
          <el-form-item label="公司账户2" prop="companyAccountTwo" :class="{ 'is-required': true }">
              <el-input v-model="dataForm.companyAccountTwo" placeholder="公司账户2"></el-input>
            </el-form-item>
          </el-col>
        <el-col :span="12">
          <el-form-item label="抽成（%）" prop="companyTwoCommission" :class="{ 'is-required': true }">
              <el-input v-model="dataForm.companyTwoCommission" placeholder="抽成" maxlength="6"></el-input>
            </el-form-item>
          </el-col>
      </el-row>

      <!-- 增加账户 -->
      <el-row v-for="(item,inx) in extraListForm" :key="item.id">
        <el-col :span="12">
          <el-form-item :label="'公司账户'+(2+inx+1)" prop="">
            <el-input v-model="item.valueOne" readonly="true"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="10">
          <el-form-item label="抽成（%）" prop="">
            <el-input v-model="item.valueThree" placeholder="抽成" maxlength="6" readonly="true"></el-input>
            <el-button type="danger" icon="el-icon-delete" @click="deleteChargingAccount(item.id,inx,1)" size="mini" circle style="position:absolute;top:-2px;right:-50px;"></el-button>
          </el-form-item>
        </el-col>
      </el-row>

      <el-form-item label="" prop="">
        <el-button size="mini" plain round @click="dialogAddAccountFormVisible=true">增加抽成账户+</el-button>
      </el-form-item>

      <el-dialog title="添加" :visible.sync="dialogAddAccountFormVisible" width="25%" append-to-body>
        <el-form :model="addAccountForm" label-width="90px">
          <el-form-item label="抽成账号" >
            <el-input v-model="addAccountForm.account" placeholder="输入交易所账号" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="抽成比例">
           <el-input v-model="addAccountForm.proportion" placeholder="最多保留两位小数" maxlength="6"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogAddAccountFormVisible=false">取 消</el-button>
          <el-button type="primary" @click="addChargingAccount">确 定</el-button>
        </div>
      </el-dialog>

      <el-row>
        <el-col :span="12">
          <el-form-item label="中间商账户" prop="middlemanAccount" :class="{ 'is-required': true }">
            <el-input v-model="dataForm.middlemanAccount" placeholder="中间商抽成"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="抽成（%）" prop="middlemanCommission" :class="{ 'is-required': true }">
            <el-input v-model="dataForm.middlemanCommission" placeholder="抽成" maxlength="6"></el-input>
          </el-form-item>
        </el-col>
      </el-row>

      <el-row><el-col :span="24"><div style="background: #F0F0F0; margin: 15px 0px 10px 0px; padding: 5px 15px">提币抽成配置</div></el-col></el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="公司账户" prop="withdrawCompanyAccount" :class="{ 'is-required': false }">
              <el-input v-model="dataForm.withdrawCompanyAccount" placeholder="公司账户"></el-input>
            </el-form-item>
          </el-col>
        <el-col :span="12">
          <el-form-item label="抽成（%）" prop="withdrawCompanyAccountCommission" :class="{ 'is-required': false }">
              <el-input v-model="dataForm.withdrawCompanyAccountCommission" placeholder="抽成" maxlength="6"></el-input>
            </el-form-item>
          </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="中间商账户" prop="withdrawMiddlemanAccount" :class="{ 'is-required': false }">
              <el-input v-model="dataForm.withdrawMiddlemanAccount" placeholder="中间商账户"></el-input>
            </el-form-item>
          </el-col>
        <el-col :span="12">
          <el-form-item label="抽成（%）" prop="withdrawMiddlemanAccountCommission" :class="{ 'is-required': false }">
              <el-input v-model="dataForm.withdrawMiddlemanAccountCommission" placeholder="抽成" maxlength="6"></el-input>
            </el-form-item>
          </el-col>
      </el-row>

      <el-row><el-col :span="24"><div style="background: #F0F0F0; margin: 15px 0px 10px 0px; padding: 5px 15px">提现抽成配置</div></el-col></el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="公司账户" prop="cashoutCompanyAccount" :class="{ 'is-required': false }">
              <el-input v-model="dataForm.cashoutCompanyAccount" placeholder="公司账户"></el-input>
            </el-form-item>
          </el-col>
        <el-col :span="12">
          <el-form-item label="抽成（%）" prop="cashoutCompanyScale" :class="{ 'is-required': false }">
              <el-input v-model="dataForm.cashoutCompanyScale" placeholder="抽成" maxlength="6"></el-input>
            </el-form-item>
          </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="中间商账户" prop="cashoutMiddlemanAccount" :class="{ 'is-required': false }">
              <el-input v-model="dataForm.cashoutMiddlemanAccount" placeholder="中间商账户"></el-input>
            </el-form-item>
          </el-col>
        <el-col :span="12">
          <el-form-item label="抽成（%）" prop="cashoutMiddlemanScale" :class="{ 'is-required': false }">
              <el-input v-model="dataForm.cashoutMiddlemanScale" placeholder="抽成" maxlength="6"></el-input>
            </el-form-item>
          </el-col>
      </el-row>

    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>
<script>
  export default {
    data () {
      const validateCommission = (rule, value, callback) => {
        if (value == null || value.length <= 0 || !/^\d+(\.\d{1,2})?$/.test(value) || value > 99.99 || value < 0) {
          callback(new Error('抽成值在 0.00 至 99.99之间, 且仅保留两位小数点'));
        } else {
          callback();
        }
      }
      const validateCurrencyValue = (rule, value, callback) => { // 校验币种价值
        if (value.length > 0 && (!/^\d+(\.\d{1,2})?$/.test(value) || value > 99999.99 || value <= 0)) {
          callback(new Error('数值在 0.01 至 99999.99之间, 且仅保留两位小数点'));
        } else {
          callback();
        }
      }
      const validateProjectCurrencyPoint = (rule, value, callback) => {
        if (value == '0') {
          callback();
          return;
        }
        if (value == '' || (!/^\d+?$/.test(value))) {
          callback(new Error('请输入有效的小数位数，0表示整数'));
        } else {
          callback();
        }
      }
      const validateMemberShowPercent = (rule, value, callback) => {
        if (value.length <= 0 || !/^\d+(\.\d{1,2})?$/.test(value) || value > 100.00 || value < 0) {
          callback(new Error('比例值在 0.00 至 100.00之间, 且仅保留两位小数点'));
        } else {
          callback();
        }
      }
      const validatePayAmount = (rule, value, callback) => {
        var resultFlag = true;
        var arrayItem = value.split(',');
        for (var i in arrayItem) {
          if (arrayItem[i] == '' || (!/^\d+?$/.test(arrayItem[i]))) {
            resultFlag = false;
            break;
          }
        }
        if (!resultFlag) {
          callback(new Error('正确的可选支付金额格式为（多个用英文逗号隔开， 且不能有空格）：100,200,300'));
        } else {
          callback();
        }
      }

      return {
        visible: false,
        showProjectCurrencyValue: false,
        showProjectCurrencyCoin: false,
        showProjectSkipValue: false,
        otcBuyCoinOptions:[{value:'BTC', label:'BTC'}, {value:'USDT', label:'USDT'}],
        dataForm: {
          id: 0,
          applicationName: '',
          projectAccount: '',
          projectCurrencyName: '',
          ownerName: '',
          projectCurrencyType: '',
          projectCurrencyValue: '',
          projectCurrencyCoin: '',
          projectCurrencyPoint: '',
          payWay: [],
          frontDisplay:[],
          payAmount: '',
          otcBuyCoin: '',
          middlemanAccount: '',
          middlemanCommission: '',
          companyAccountOne: '',
          companyOneCommission: '',
          companyAccountTwo: '',
          companyTwoCommission: '',
          withdrawCompanyAccount:'',
          withdrawCompanyAccountCommission:'',
          withdrawMiddlemanAccount:'',
          withdrawMiddlemanAccountCommission:'',
          cashoutCompanyAccount: '',
          cashoutCompanyScale: 0,
          cashoutMiddlemanAccount: '',
          cashoutMiddlemanScale: 0,
          memberShowPercent: '',
          rechargeOrWithdrawApi: '',
          selectBalanceApi: '',
          status: 1,
          smsActive: 0,
          serversAddress: '',
          orderCalculatePrice: 0,
          orderAmountType: 0,
          skipUrl: '',
          assignSkipUrl: '',
          isAppointMerchant: '' // 是否指定承兑商
        },
        extraListForm: [],
        merchantAccount: '', // 承兑商账号
        addAccountForm: {
          account: '',
          proportion: ''
        },
        dialogAddAccountFormVisible: false,
        dialogMerchantAccountFormVisible: false, // 承兑商账号弹框
        dataRule: {
          applicationName: [
            { required: true, message: '应用名不能为空', trigger: 'blur' }
          ],
          projectAccount: [
            { required: true, message: '项目账号不能为空', trigger: 'blur' }
          ],
          ownerName: [
            { required: true, message: '项目方人名不能为空', trigger: 'blur' }
          ],
          projectCurrencyName: [
            { required: true, message: '项目币名不能为空', trigger: 'blur' }
          ],
          projectCurrencyType: [
            { required: true, message: '请选择项目货币类型', trigger: 'blur' }
          ],
          memberShowPercent: [
            { validator: validateMemberShowPercent, trigger: 'blur' }
          ],
          payWay: [
            { required: true, message: '请选择支付类型', trigger: 'blur' }
          ],
          payAmount: [
            { validator: validatePayAmount, trigger: 'blur' }
          ],
          projectCurrencyPoint: [
            { validator: validateProjectCurrencyPoint, trigger: 'blur' }
          ],
          projectCurrencyValue: [ // 检验游戏币价值
            { validator: validateCurrencyValue, trigger: 'blur' }
          ],
          otcBuyCoin: [
            { required: true, message: '请选择法币购买币种', trigger: 'blur' }
          ],

          // 充币抽成账户校验
          companyAccountOne: [
            { required: true, message: '公司账号1不能为空', trigger: 'blur' }
          ],
          companyOneCommission: [
            { validator: validateCommission, trigger: 'blur' }
          ],
          companyAccountTwo: [
            { required: true, message: '公司账号2不能为空', trigger: 'blur' }
          ],
          companyTwoCommission: [
            { validator: validateCommission, trigger: 'blur' }
          ],
          middlemanAccount: [
            { required: true, message: '中间商账户不能为空', trigger: 'blur' }
          ],
          middlemanCommission: [
            { validator: validateCommission, trigger: 'blur' }
          ],

          // 提现抽成账户校验
          cashoutCompanyScale: [
            { validator: validateCommission, trigger: 'blur' }
          ],
          cashoutMiddlemanScale: [
            { validator: validateCommission, trigger: 'blur' }
          ],

          // 币种标志校验
          projectCurrencyCoin: [
            { required: true, message: '币种标志不能为空', trigger: 'blur' }
          ],

          frontDisplay: [
            { required: true, message: '前端显示不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
        })
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/payapplication/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.applicationName = data.payApplication.applicationName;
              this.dataForm.projectAccount = data.payApplication.projectAccount;
              this.dataForm.projectCurrencyName = data.payApplication.projectCurrencyName;
              this.dataForm.ownerName = data.payApplication.ownerName;
              this.dataForm.projectCurrencyType = data.payApplication.projectCurrencyType;
              this.dataForm.projectCurrencyValue = data.payApplication.projectCurrencyValue;
              this.dataForm.projectCurrencyCoin = data.payApplication.projectCurrencyCoin;
              this.dataForm.projectCurrencyPoint = data.payApplication.projectCurrencyPoint;
              this.dataForm.payWay = data.payApplication.payWay ? data.payApplication.payWay.split(',').map(Number) : [];
              this.dataForm.payAmount = data.payApplication.payAmount;
              this.dataForm.otcBuyCoin = data.payApplication.otcBuyCoin;
              this.dataForm.middlemanAccount = data.payApplication.middlemanAccount;
              this.dataForm.middlemanCommission = data.payApplication.middlemanCommission;
              this.dataForm.status = data.payApplication.status;
              this.dataForm.companyAccountOne = data.payApplication.companyAccountOne;
              this.dataForm.companyOneCommission = data.payApplication.companyOneCommission;
              this.dataForm.companyAccountTwo = data.payApplication.companyAccountTwo;
              this.dataForm.companyTwoCommission = data.payApplication.companyTwoCommission;
              this.dataForm.withdrawCompanyAccount = data.payApplication.withdrawCompanyAccount;
              this.dataForm.withdrawCompanyAccountCommission = data.payApplication.withdrawCompanyAccountCommission;
              this.dataForm.withdrawMiddlemanAccount = data.payApplication.withdrawMiddlemanAccount;
              this.dataForm.withdrawMiddlemanAccountCommission = data.payApplication.withdrawMiddlemanAccountCommission;
              this.dataForm.cashoutCompanyAccount = data.payApplication.cashoutCompanyAccount;
              this.dataForm.cashoutCompanyScale = data.payApplication.cashoutCompanyScale;
              this.dataForm.cashoutMiddlemanAccount = data.payApplication.cashoutMiddlemanAccount;
              this.dataForm.cashoutMiddlemanScale = data.payApplication.cashoutMiddlemanScale;
              this.dataForm.memberShowPercent = data.payApplication.memberShowPercent;
              this.dataForm.rechargeOrWithdrawApi = data.payApplication.rechargeOrWithdrawApi;
              this.dataForm.selectBalanceApi = data.payApplication.selectBalanceApi;
              this.changeProjectCurrencyType();
              this.dataForm.frontDisplay = data.payApplication.frontDisplay ? JSON.parse(data.payApplication.frontDisplay) : [];
              this.dataForm.smsActive = data.payApplication.smsActive;
              this.dataForm.serversAddress = data.payApplication.serversAddress;
              this.dataForm.orderCalculatePrice = data.payApplication.orderCalculatePrice;
              this.dataForm.orderAmountType = data.payApplication.orderAmountType;
              this.extraListForm = data.extraList ? data.extraList : [];
              this.merchantAccount = data.merchantComfig.valueOne // 承兑商账号
              if (data.payApplication.skipUrl === '0') {
                this.dataForm.skipUrl = data.payApplication.skipUrl;
              } else {
                this.dataForm.skipUrl = '1';
                this.dataForm.assignSkipUrl = data.payApplication.skipUrl;
              }
              this.dataForm.isAppointMerchant = data.payApplication.isAppointMerchant // 是否指定承兑商
            }
          })
        } else {
          this.showProjectCurrencyValue = false;
          this.showProjectCurrencyCoin = false;
        }
      },
      // 项目货币类型
      changeProjectCurrencyType () {
        if (this.dataForm.projectCurrencyType === 1) {
          this.showProjectCurrencyValue = true;
          this.showProjectCurrencyCoin = false;
        }
        if (this.dataForm.projectCurrencyType === 2) {
          this.showProjectCurrencyValue = false;
          this.showProjectCurrencyCoin = true;
        }
      },

      // 新增充币抽成账号
      addChargingAccount () {
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl('/payapplication/saveAccount'),
            method: 'post',
            params: this.$http.adornParams({
              'id': this.dataForm.id,
              'takeAccount': this.addAccountForm.account,
              'takScale': this.addAccountForm.proportion
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.extraListForm.push(data.extra)
              this.dialogAddAccountFormVisible = false
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }
      },

      // 新增承兑商账号
      saveMerchantAccount () {
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl('/payapplication/saveMerchantAccount'),
            method: 'post',
            params: this.$http.adornParams({
              'appId': this.dataForm.id,
              'account': this.addAccountForm.account
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.merchantAccount = data.extra.valueOne
              this.dialogMerchantAccountFormVisible = false
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }
      },

      // 删除充币抽成账号
      deleteChargingAccount (id, inx, type) {
        if (id) {
          this.$confirm('确认是否删除?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$http({
              url: this.$http.adornUrl('/payapplication/deleteAccount'),
              method: 'post',
              params: this.$http.adornParams({
                'id': id
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.extraListForm.splice(inx,1)
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          })
        }
      },

      // 表单提交
      dataFormSubmit () {
        if(this.dataForm.assignSkipUrl.indexOf("://")==-1&&this.dataForm.skipUrl==1){
          this.$message({
            message: '请输入合法的url链接，如http://开头或https://或app链接头',
            type: 'error',
            duration: 1500,
            onClose: () => {
              this.visible = false
              this.$emit('refreshDataList')
            }
          })
          return;
        }
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/payapplication/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'applicationId': this.dataForm.id || undefined,
                'applicationName': this.dataForm.applicationName,
                'projectAccount': this.dataForm.projectAccount,
                'ownerName': this.dataForm.ownerName,
                'projectCurrencyName': this.dataForm.projectCurrencyName,
                'projectCurrencyType': this.dataForm.projectCurrencyType,
                'projectCurrencyValue': this.dataForm.projectCurrencyValue,
                'projectCurrencyCoin': this.dataForm.projectCurrencyCoin,
                'projectCurrencyPoint': this.dataForm.projectCurrencyPoint,
                'payWay': this.dataForm.payWay.join(','),
                'payAmount': this.dataForm.payAmount,
                'otcBuyCoin': this.dataForm.otcBuyCoin,
                'middlemanAccount': this.dataForm.middlemanAccount,
                'middlemanCommission': this.dataForm.middlemanCommission,
                'companyAccountOne': this.dataForm.companyAccountOne,
                'companyOneCommission': this.dataForm.companyOneCommission,
                'companyAccountTwo': this.dataForm.companyAccountTwo,
                'companyTwoCommission': this.dataForm.companyTwoCommission,
                'withdrawCompanyAccount': this.dataForm.withdrawCompanyAccount,
                'withdrawCompanyAccountCommission': this.dataForm.withdrawCompanyAccountCommission,
                'withdrawMiddlemanAccount': this.dataForm.withdrawMiddlemanAccount,
                'withdrawMiddlemanAccountCommission': this.dataForm.withdrawMiddlemanAccountCommission,
                'cashoutCompanyAccount': this.dataForm.cashoutCompanyAccount,
                'cashoutCompanyScale': this.dataForm.cashoutCompanyScale,
                'cashoutMiddlemanAccount': this.dataForm.cashoutMiddlemanAccount,
                'cashoutMiddlemanScale': this.dataForm.cashoutMiddlemanScale,
                'memberShowPercent': this.dataForm.memberShowPercent,
                'rechargeOrWithdrawApi': this.dataForm.rechargeOrWithdrawApi,
                'selectBalanceApi': this.dataForm.selectBalanceApi,
                'frontDisplay': JSON.stringify(this.dataForm.frontDisplay),
                'status': this.dataForm.status,
                'smsActive': this.dataForm.smsActive,
                'serversAddress': this.dataForm.serversAddress,
                'orderCalculatePrice': this.dataForm.orderCalculatePrice,
                'orderAmountType': this.dataForm.orderAmountType,
                'skipUrl': this.dataForm.skipUrl=='0'?this.dataForm.skipUrl:this.dataForm.assignSkipUrl,
                'isAppointMerchant': this.dataForm.isAppointMerchant
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
