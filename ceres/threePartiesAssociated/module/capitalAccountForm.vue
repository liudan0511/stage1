<template>
  <div class="capital-account-box">
    <!--    资金账户信息-->
    <el-form ref="capitalAccountForm"
             :inline="true"
             :model="capitalAccountForm"
             :rules="computedFormRules"
             size="mini"
             label-position="left"
             @submit.native.prevent>
      <!--      资金账户信息-->
      <div class="title fund">
        资金账户信息
      </div>
      <div class="info fundInfo">
        <el-row>
          <el-col :span="10">
            <el-form-item label="产品名称:" label-width="156px" prop="productCode">
              <template slot-scope="scope">
                <el-select
                  filterable
                  @change="onProductCodeChange"
                  v-model="capitalAccountForm.productCode"
                  placeholder="请选择"
                  style="width: 360px;"
                  :disabled="computedChildExhibitMode || computedChildEditMode"
                >
                  <el-option
                    v-for="item in productCodeOptions"
                    :key="item.productCode"
                    :label="item.productFullName"
                    :value="item.productCode"
                    style="width: 360px;">
                  </el-option>
                </el-select>
              </template>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="16">
            <el-form-item label="签约类型:" label-width="156px" prop="singedType">>
              <template slot-scope="scope">
                <!--编辑和展示时候禁用 -->
                <el-radio-group
                  v-model="capitalAccountForm.singedType"
                  :disabled="computedChildExhibitMode || computedChildEditMode"
                  @change="agreeChange">
                  <el-radio label="01" v-show="computedSignTypeForForm">三方存管</el-radio>
                  <el-radio label="02" v-show="computedSignTypeForForm">融资融券</el-radio>
                  <el-radio label="03" v-show="computedSignTypeForForm">银期签约</el-radio>
                  <el-radio label="04" v-show="computedSignTypeForForm">银衍签约</el-radio>
                  <el-radio label="05" v-show="!computedSignTypeForForm">解除银期签约</el-radio>
                  <el-radio label="06" v-show="!computedSignTypeForForm">解除银衍签约</el-radio>
                </el-radio-group>
              </template>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>

            <el-form-item label="托管账户名称:" label-width="156px" prop="escrowAccountName">
              <el-input
                v-model="capitalAccountForm.escrowAccountName"
                :disabled="true"
                placeholder=""
                class="filter-item"
                :readonly="computedChildExhibitMode"
              />
            </el-form-item>

            <el-form-item label="托管账户账号:" label-width="156px" prop="escrowAccount" class="ml">
              <el-input
                v-model="capitalAccountForm.escrowAccount"
                :disabled="true"
                placeholder=""
                class="filter-item"
                :readonly="computedChildExhibitMode"
              />
            </el-form-item>

        </el-row>
        <el-row>

            <el-form-item label="资金账号:" label-width="156px" prop="capitalAccount">
              <el-input
                v-model="capitalAccountForm.capitalAccount"
                @change="updateCapitalAccountInfo"
                placeholder=""
                class="filter-item"
                maxlength="30"
                :disabled="computedChildExhibitMode"
                :readonly="computedChildExhibitMode"
              />
            </el-form-item>
            <el-form-item label="资金密码:" label-width="156px" prop="capitalPassword" class="ml">
              <el-input
                v-model="capitalAccountForm.capitalPassword"
                placeholder=""
                :type="pwdType"
                auto-complete="new-password"
                class="filter-item"
                maxlength="6"
                :disabled="computedChildExhibitMode"
                :readonly="computedChildExhibitMode">
                <i slot="suffix" class="el-icon-view" @click="showPwd"></i>
              </el-input>
            </el-form-item>

        </el-row>
        <el-row>

            <el-form-item label="证件类型:" label-width="156px" prop="certificateType">
              <el-input
                v-model="capitalAccountForm.certificateType"
                placeholder=""
                :disabled="true"
                class="filter-item"
              />
            </el-form-item>

            <el-form-item label=" 证件号码:" label-width="156px" prop="certificateNumber" class="ml">
              <el-input
                v-model="capitalAccountForm.certificateNumber"
                placeholder=""
                :disabled="true"
                class="filter-item"
              />
            </el-form-item>

        </el-row>
        <el-row>

            <el-form-item label="签约券商/期货商:" label-width="156px" prop="signedContractId">
              <template slot-scope="scope">
                <el-select
                  filterable
                  @change="onSingedContractChange"
                  v-model="capitalAccountForm.signedContractId"
                  placeholder="请选择"
                  :disabled="computedChildExhibitMode || computedChildEditMode"
                  style="width: 360px;">
                  <el-option
                    v-for="item in singedContractOptions"
                    :key="item.signedContractId"
                    :label="item.signedContractName"
                    :value="item.signedContractId"
                    style="width: 360px;">
                  </el-option>
                </el-select>
              </template>
            </el-form-item>
            <el-form-item label="营业部:" label-width="156px" prop="salesDepartment" class="ml">
              <el-input
                v-model="capitalAccountForm.salesDepartment"
                placeholder=""
                class="filter-item"
                maxlength="30"
                :disabled="computedChildExhibitMode"
                :readonly="computedChildExhibitMode"
              />
            </el-form-item>
        </el-row>
        <el-row>
            <el-form-item label="紧急联系人:" label-width="156px" prop="contactName">
              <el-input
                v-model="capitalAccountForm.contactName"
                placeholder=""
                class="filter-item"
                maxlength="15"
                :disabled="computedChildExhibitMode"
                :readonly="computedChildExhibitMode"
              />
            </el-form-item>
            <el-form-item label="联系电话:" label-width="156px" prop="contactMobile" class="ml">
              <el-input
                v-model="capitalAccountForm.contactMobile"
                placeholder=""
                class="filter-item"
                maxlength="11"
                :disabled="computedChildExhibitMode"
                :readonly="computedChildExhibitMode"
              />
            </el-form-item>
        </el-row>
      </div>
    </el-form>
  </div>
</template>

<script>
import store from '@/store'
import {
  getCertificateType, getFuturesTraderInfos,
  getProductInfos,
  getTrusteeshipInfo,
} from 'api/ceres/threePartiesAssociated'

export default {
  name: 'StepBar',
  components: {},
  props: {
    //  form
    capitalAccountForm: {
      type: Object,
      required: true,
    },
    globalReadOnly: {
      type: Boolean,
      required: true,
    },
    signTypeForForm: {
      type: String,
      required: true,
    },
    sup_this: {
      type: Object,
      required: false,
    },
    exhibitMode: {
      type: Boolean,
      required: true,
      default: false,
    },
    editMode: {
      type: Boolean,
      required: true,
      default: false,
    },

  },
  data() {
    const validateNumber = (rule, value, callback) => {
      if (!(/^\d+$/.test(value))) {
        callback(new Error('请输入数字!'))
      }
      callback()
    }

    const validatePassword = (rule, value, callback) => {
      if (value) {
        if (/[\u4E00-\u9FA5]/g.test(
          value)) {
          callback(new Error('密码不能输入中文!'))
        } else if (/[\u3002\uff1b\uff0c\uff1a\u201c\u201d\uff08\uff09\u3001\uff1f\u300a\u300b]/g.test(
          value)) {
          callback(new Error('密码不能输入中文标点!'))
        }else if(!(/^\d+$/.test(value))){
          callback(new Error('请输入数字!'))
        }else if(value.length<6 || value.length>6){
          callback(new Error('密码必须是6位!'))
        } else {
          callback()
        }
      }
      callback()
    }

    return {
      // 密码类型
      pwdType: 'password',
      // 展示签约还是解约radio项
      showSignOptions: false,
      // 产品代码集合
      productCodeOptions: [],
      // 签约券商/期货商:集合
      singedContractOptions: [],
      //选中银期签约是否展示期货参数信息
      paramsFlag: false,

      rules: {
        //管理人信息
        productCode: [
          { required: true, message: '请输入产品名称', trigger: ['blur', 'change'] },
        ],
        singedType: [
          { required: true, message: '请选择签约类型', trigger: ['blur', 'change'] },
        ],
        escrowAccountId: [
          { required: false, message: '请输入托管账户Id', trigger: ['blur', 'change'] },
        ],
        escrowAccountName: [
          { required: true, message: '请输入托管账户名称', trigger: ['blur', 'change'] },
        ],
        escrowAccount: [
          { required: true, message: '请输入托管账户账号', trigger: ['blur', 'change'] },
        ],
        capitalAccount: [
          { required: true, message: '请输入资金账号', trigger: ['blur', 'change'] },
          { validator: validateNumber, trigger: ['blur', 'change'] },
        ],
        capitalPassword: [
          { required: true, message: '请输入资金密码', trigger: ['blur', 'change'] },
          { validator: validatePassword, trigger: ['blur', 'change'] },
        ],
        certificateType: [
          { required: false, message: '请输入证件类型', trigger: ['blur', 'change'] },
        ],
        certificateName: [
          { required: true, message: '请输入证件类型', trigger: ['blur', 'change'] },
        ],
        certificateNumber: [
          { required: false, message: '请输入证件号码', trigger: ['blur', 'change'] },
        ],
        signedContractId: [
          { required: true, message: '请输入签约券商/期货商', trigger: ['blur', 'change'] },
        ],
        salesDepartment: [
          { required: true, message: '请输入营业部', trigger: ['blur', 'change'] },
        ],
        contactName: [
          { required: true, message: '请输入紧急联系人', trigger: ['blur', 'change'] },
        ],
        contactMobile: [
          { required: true, message: '请输入联系电话', trigger: ['blur', 'change'] },
          {
            pattern: /^1[345789]\d{9}$/,
            message: '请输入正确的手机号码!',
            trigger: 'blur',
          },
        ],
      },
      notRequiredRules: {
        //管理人信息
        productCode: [
          { required: false, message: '请输入产品名称', trigger: [] },
        ],
        singedType: [
          { required: false, message: '请选择签约类型', trigger: [] },
        ],
        escrowAccountId: [
          { required: false, message: '请输入托管账户Id', trigger: [] },
        ],
        escrowAccountName: [
          { required: false, message: '请输入托管账户名称', trigger: [] },
        ],
        escrowAccount: [
          { required: false, message: '请输入托管账户账号', trigger: [] },
        ],
        capitalAccount: [
          { required: false, message: '请输入资金账号', trigger: [] },

        ],
        capitalPassword: [
          { required: false, message: '请输入资金密码', trigger: [] },

        ],
        certificateType: [
          { required: false, message: '请输入证件类型', trigger: [] },
        ],
        certificateName: [
          { required: false, message: '请输入证件类型', trigger: [] },
        ],
        certificateNumber: [
          { required: false, message: '请输入证件号码', trigger: [] },
        ],
        signedContractId: [
          { required: false, message: '请输入签约券商/期货商', trigger: [] },
        ],
        salesDepartment: [
          { required: false, message: '请输入营业部', trigger: [] },
        ],
        contactName: [
          { required: false, message: '请输入紧急联系人', trigger: [] },
        ],
        contactMobile: [
          { required: false, message: '请输入联系电话', trigger: [] },
        ],
      },
    }
  },
  mounted() {
  },
  computed: {
    computedFormRules() {
      if (this.exhibitMode || this.globalReadOnly) {
        return this.notRequiredRules
      } else {
        return this.rules
      }
    },
    computedSignTypeForForm() {
      return this.signTypeForForm === 'sign'
    },
    computedChildExhibitMode() {
      if (this.exhibitMode) {
        return true
      } else {
        if (this.globalReadOnly) {
          return true
        } else {
          return false
        }
      }
    },
    computedChildEditMode() {
      return this.editMode
    },
  },
  methods: {
    initEvent() {
      this.queryProductInfo()
      this.queryCertificateType()
      this.setDefaultContactInfo()
      this.queryFuturesTraderInfos()
    },
    //是否展示期货参数信息
    agreeChange(event) {
      // 如果点击了银期签约
      if (event === '03') {
        this.paramsFlag = true
      } else {
        this.paramsFlag = false
      }
      let data = {
        paramsFlag: this.paramsFlag,
        signType: event,
      }
      // show期货信息
      this.$emit('handleFuturesParamsForm', data)
    },
    //获取产品
    queryProductInfo() {
      let params = {
        // creatorId:'CSBY01'
        creatorId: store.getters.user.userId,
      }
      getProductInfos(params).then((res) => {
        if (res.status === 'success') {
          this.productCodeOptions = res.data
        } else {
          this.toast(res.msg, 'warning')
        }
      }).catch((error) => {
        console.error(error)
      })
    },
    queryCertificateType() {
      getCertificateType().then((res) => {
        if (res.status === 'success') {
          const data = res.data
          if (data.length > 0) {
            this.capitalAccountForm.certificateType = data[0].certificateType
            this.capitalAccountForm.certificateNumber = data[0].certificateNumber
            this.capitalAccountForm.certificateName = data[0].certificateName
          }
        } else {
          this.toast(res.msg, 'warning')
        }
      }).catch((error) => {
        console.error(error)
      })
    },
    setDefaultContactInfo() {
      const name = store.getters.user.userName
      if (name) {
        this.capitalAccountForm.contactName = name
      }

      const mobile = store.getters.user.mobile
      if (mobile) {
        this.capitalAccountForm.contactMobile = mobile
      }
    },
    queryFuturesTraderInfos() {
      getFuturesTraderInfos().then((res) => {
        if (res.status === 'success') {
          this.singedContractOptions = res.data
        } else {
          this.toast(res.msg, 'warning')
        }
      }).catch((error) => {
        console.error(error)
      })

    },
    onProductCodeChange(productCode) {
      this.queryTrusteeshipInfo(productCode)
    },
    onSingedContractChange(id) {
      let item = {}
      this.singedContractOptions.some(function(value) {
        if (value.signedContractId === id) {
          item = value
          return true
        }
      })
      this.$emit('updateTrusteeshipInfo', item)
    },
    updateCapitalAccountInfo(value) {
      this.$emit('updateCapitalAccount', value)
    },
    queryTrusteeshipInfo(productCode) {
      let params = {
        productCode,
      }
      getTrusteeshipInfo(params).then((res) => {
        if (res.status === 'success') {
          let data = res.data
          // 受现代 JavaScript 的限制 (以及废弃 Object.observe)，Vue 不能检测到对象属性的添加或删除。
          this.$set(this.capitalAccountForm, 'escrowAccountId', data.escrowAccountId)
          this.$set(this.capitalAccountForm, 'escrowAccountName', data.escrowAccountName)
          this.$set(this.capitalAccountForm, 'escrowAccount', data.escrowAccount)
        } else {
          this.toast(res.msg, 'warning')
        }
      }).catch((error) => {
        console.error(error)
      })
    },
    toast(text, type) {
      if (!type) {
        type = 'success'
      }
      this.$notify({
        title: text,
        type: type,
        duration: 2500,
      })
    },
    showPwd() {
      this.pwdType === 'password' ? this.pwdType = '' : this.pwdType = 'password'
      let e = document.getElementsByClassName('el-icon-view')[0]
      if (!this.pwdType) {
        e.setAttribute('style', 'color: #409EFF')
      } else {
        e.setAttribute('style', 'color: #c0c4cc')
      }
    },
  },
}
</script>

<style scoped lang="scss">
  .info{
    width: 1360px;
    margin: 0 auto;
    padding: 20px 0;
  }
  .ml{
    margin-left: 275px;
  }
  .title {
    background: rgba(245, 247, 250, 1);
    border: 1px solid rgba(224, 229, 233, 1);
    font-size: 16px;
    text-align: center;
    font-weight: normal;
    color: rgba(50, 50, 50, 1);
  }

  .title.fund {
    height: 42px;
    line-height: 42px;
  }

  .footer {
    text-align: center;
    margin-top: 15px;
  }

  .footer .el-button {
    width: 120px;
    height: 42px;
  }

  /deep/ .el-form-item__label {
    font-size: 14px;
    font-weight: 400;
    color: rgba(100, 100, 100, 1);
  }

  /*输入框大小*/
  /deep/ .el-input__inner {
    width: 360px;
    height: 36px;
  }

  /deep/ .el-icon-view {
    line-height: 36px;
    margin-right: 10px;
  }
  /deep/ .el-radio{
    margin-right: 12px;
  }

</style>
