<template>
  <div>
    <el-form ref="futuresParamsForm"
             :model="futuresParamsForm"
             :rules="computedFormRules"
             :inline="true"
             size="mini"
             label-position="left"
             @submit.native.prevent>
      <!--      期货参数信息-->
      <div class="title parameter">
        期货参数信息
      </div>
      <div class="info futuresInfo">
        <div class="futuresPart futuresPartOne">
          <div class="partTitle">期货参商信息</div>
          <el-row>
            <el-col :span="10">
              <el-form-item label="期货公司:" label-width="156px" prop="futuresCompanyCodeName">
                <el-input
                  v-model="futuresParamsForm.futuresCompanyCodeName"
                  placeholder=""
                  :disabled="true"
                  class="filter-item"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="2">
              <el-form-item label="会员号:" label-width="130px" style="padding-left: 10px">
              </el-form-item>
            </el-col>
            <el-col :span="4">
              <el-form-item label="中金所:" prop="cffexMid">
                <el-input
                  v-model="futuresParamsForm.cffexMid"
                  placeholder=""
                  class="filter-item account_item account"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
            <el-col :span="4">
              <el-form-item label="上期所:" prop="shfeMid">
                <el-input
                  v-model="futuresParamsForm.shfeMid"
                  placeholder=""
                  maxlength="30"
                  class="filter-item account_item account"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
            <el-col :span="4">
              <el-form-item label="大商所:" prop="dceMid">
                <el-input
                  v-model="futuresParamsForm.dceMid"
                  placeholder=""
                  class="filter-item account_item account"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
            <!--            <el-col :span="2">-->
            <!--              &lt;!&ndash; 需要有空格占位&ndash;&gt;-->
            <!--              <el-form-item label="  " label-width="165px">-->
            <!--              </el-form-item>-->
            <!--            </el-col>-->
            <el-col :span="4">
              <el-form-item label="郑商所:" prop="zceMid">
                <el-input
                  v-model="futuresParamsForm.zceMid"
                  placeholder=""
                  class="filter-item account_item account"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
            <el-col :span="4">
              <el-form-item label="能源中心:" prop="ineMid">
                <el-input
                  v-model="futuresParamsForm.ineMid"
                  placeholder=""
                  class="filter-item account_item account"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>

              <el-form-item label="开户行名称:" label-width="156px" prop="openAccountBank">
                <el-input
                  v-model="futuresParamsForm.openAccountBank"
                  placeholder=""
                  class="filter-item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
              <el-form-item label="开户行网点:" label-width="165px" prop="bankOutlets" class="ml">
                <el-input
                  v-model="futuresParamsForm.bankOutlets"
                  placeholder=""
                  class="filter-item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>

          </el-row>
          <el-row>
            <el-col :span="10">
              <el-form-item label="期货公司保证金账号:" label-width="156px" prop="securityDepositAccount">
                <el-input
                  v-model="futuresParamsForm.securityDepositAccount"
                  placeholder=""
                  class="filter-item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
            <!--            <el-col :span="10" style="display: none">-->
            <!--              <el-form-item label="期货公司代码:" label-width="165px" prop="futuresCompanyCode">-->
            <!--                <el-input-->
            <!--                  v-model="futuresParamsForm.futuresCompanyCode"-->
            <!--                  placeholder="请输入期货公司代码"-->
            <!--                  class="filter-item"-->
            <!--                  :readonly="computedChildExhibitMode"-->
            <!--                />-->
            <!--              </el-form-item>-->
            <!--            </el-col>-->
          </el-row>
        </div>
        <div class="futuresPart client">
                    <div class="future-part-top-line"></div>
          <div class="partTitle">客户账户信息</div>
          <el-row>
            <el-col :span="10">
              <el-form-item label="内部资金账号:" label-width="156px" prop="internalCapitalAccount">
                <el-input
                  v-model="futuresParamsForm.internalCapitalAccount"
                  placeholder=""
                  :disabled="true"
                  class="filter-item"
                  maxlength="9"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="2">
              <el-form-item label="中金所交易编码:" label-width="145px">
              </el-form-item>
            </el-col>
            <el-col :span="4">
              <el-form-item label="(投机)" prop="cffexSpeculationCode">
                <el-input
                  v-model="futuresParamsForm.cffexSpeculationCode"
                  placeholder=""
                  class="filter-item account_item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
            <el-col :span="4">
              <el-form-item label="(套保)" prop="cffexHedgingCode">

                <el-input
                  v-model="futuresParamsForm.cffexHedgingCode"
                  placeholder=""
                  class="filter-item account_item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
            <el-col :span="4">
              <el-form-item label="(套利)" prop="cffexSpreadingCode">
                <el-input
                  v-model="futuresParamsForm.cffexSpreadingCode"
                  placeholder=""
                  class="filter-item account_item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="2">
              <el-form-item label="上期所交易编码:" label-width="145px">
              </el-form-item>
            </el-col>
            <el-col :span="30">
              <el-form-item label="(投机)" prop="shfeSpeculationCode">
                <el-input
                  v-model="futuresParamsForm.shfeSpeculationCode"
                  placeholder=""
                  class="filter-item account_item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="2">
              <el-form-item label="郑期所交易编码:" label-width="145px">
              </el-form-item>
            </el-col>
            <el-col :span="8">
              <el-form-item label="(投机)" prop="zceSpeculationCode">
                <el-input
                  v-model="futuresParamsForm.zceSpeculationCode"
                  placeholder=""
                  class="filter-item account_item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="2">
              <el-form-item label="大商所交易编码:" label-width="145px">
              </el-form-item>
            </el-col>
            <el-col :span="8">
              <el-form-item label="(投机)" prop="dceSpeculationCode">

                <el-input
                  v-model="futuresParamsForm.dceSpeculationCode"
                  placeholder=""
                  class="filter-item account_item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>
            </el-col>
          </el-row>
        </div>
        <div class="futuresPart ">
                    <div class="future-part-top-line"></div>
          <div class="partTitle">期货市场监控中心账户信息</div>
          <el-row>

              <el-form-item label="账号:" label-width="165px" prop="futuresMonitorAccount">
                <el-input
                  v-model="futuresParamsForm.futuresMonitorAccount"
                  placeholder=""
                  class="filter-item"
                  maxlength="30"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode"
                />
              </el-form-item>

              <el-form-item label="密码:" label-width="165px" prop="futuresMonitorPassword" class="ml">
                <el-input
                  v-model="futuresParamsForm.futuresMonitorPassword"
                  placeholder=""
                  maxlength="20"
                  class="filter-item"
                  :type="pwdType"
                  auto-complete="new-password"
                  :disabled="computedChildExhibitMode"
                  :readonly="computedChildExhibitMode">
                  <i slot="suffix" class="el-icon-view" @click="showPwd"></i>
                </el-input>
              </el-form-item>
          </el-row>
        </div>
      </div>
    </el-form>
  </div>
</template>
<script>


  export default {
    name: 'futuresParamsForm',
    components: {},
    props: {
      //  form
      futuresParamsForm: {
        type: Object,
        required: true,
      },
      globalReadOnly: {
        type: Boolean,
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
    computed: {
      computedFormRules() {
        if (this.exhibitMode || this.computedEditModeByOnlyRead) {
          return this.notRequiredRules
        } else {
          return this.rules
        }
      },
      computedEditModeByOnlyRead() {
        return this.globalReadOnly
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
        return this.exhibitMode
      },
      computedVisibleForExpress() {
        // 初审驳回时候, 编辑的时候展示上传组件
        if (this.futuresParamsForm.visible === '1') {
          if (!this.exhibitMode) {
            return true
          } else {
            return false
          }
        } else {
          return false
        }
      },
    },
    data() {
      const validateNumber = (rule, value, callback) => {
        if (!(/^\d+$/.test(value))) {
          callback(new Error('请输入数字!'))
        }
        callback()
      }
      const validateNumberAllowEmpty = (rule, value, callback) => {
        if (value) {
          if (!(/^\d+$/.test(value))) {
            callback(new Error('请输入数字!'))
          }
          callback()
        } else {
          callback()
        }
      }
      const validatePassword = (rule, value, callback) => {
        if (value) {
          if (/[\u4E00-\u9FA5]/g.test(
            value)) {
            callback(new Error('密码不能输入中文!'))
          } else if (/[\u3002\uff1b\uff0c\uff1a\u201c\u201d\uff08\uff09\u3001\uff1f\u300a\u300b]/g.test(
            value)) {
            callback(new Error('密码不能输入中文标点!'))
          } else {
            callback()
          }
        }
        callback()
      }

      return {
        labelPosition: 'left',
        form_sup_this: this,
        // 密码类型
        pwdType: 'password',

        rules: {
          cffexMid: [
            {required: false, message: '请输入中金所会员号', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          shfeMid: [
            {required: false, message: '请输入上期所会员号', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          dceMid: [
            {required: false, message: '请输入大商所会员号', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          zceMid: [
            {required: false, message: '请输入郑商所会员号', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          ineMid: [
            {required: false, message: '请输入能源中心会员号', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          openAccountBank: [
            {required: true, message: '请输入开户行名称', trigger: ['blur', 'change']},
          ],
          bankOutlets: [
            {required: true, message: '请输入开户行网点', trigger: ['blur', 'change']},
          ],
          securityDepositAccount: [
            {required: true, message: '请输入期货公司保证金账号', trigger: ['blur', 'change']},
            {validator: validateNumber, trigger: ['blur', 'change']},
          ],
          futuresCompanyCode: [
            {required: false, message: '请输入期货公司代码', trigger: ['blur', 'change']},
          ],
          internalCapitalAccount: [
            {required: true, message: '请输入内部资金账号', trigger: ['blur', 'change']},
            {validator: validateNumber, trigger: ['blur', 'change']},
          ],
          cffexSpeculationCode: [
            {required: false, message: '请输入中金所交易编码_投机', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          cffexHedgingCode: [
            {required: false, message: '请输入中金所交易编码_套保', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          cffexSpreadingCode: [
            {required: false, message: '请输入中金所交易编码_套利', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          shfeSpeculationCode: [
            {required: false, message: '请输入上期所交易编码_投机', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          zceSpeculationCode: [
            {required: false, message: '请输入郑期所交易编码_投机', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          dceSpeculationCode: [
            {required: false, message: '请输入大商所交易编码_投机', trigger: ['blur', 'change']},
            {validator: validateNumberAllowEmpty, trigger: ['blur', 'change']},
          ],
          futuresMonitorAccount: [
            {required: true, message: '请输入期货监控中心_账户', trigger: ['blur', 'change']},
            {validator: validateNumber, trigger: ['blur', 'change']},
          ],
          futuresMonitorPassword: [
            {required: true, message: '请输入期货监控中心_密码', trigger: ['blur', 'change']},
            {validator: validatePassword, trigger: ['blur', 'change']},
          ],

        },
        notRequiredRules: {

          openAccountBank: [
            {required: false, message: '', trigger: []},
          ],
          bankOutlets: [
            {required: false, message: '', trigger: []},
          ],
          securityDepositAccount: [
            {required: false, message: '', trigger: []},
          ],

          internalCapitalAccount: [
            {required: false, message: '', trigger: []},

          ],
          futuresMonitorAccount: [
            {required: false, message: '', trigger: []},

          ],
          futuresMonitorPassword: [
            {required: false, message: '', trigger: []},

          ],

        },
      }
    },
    methods: {
      showPwd() {
        this.pwdType === 'password' ? this.pwdType = '' : this.pwdType = 'password'
        let e = document.getElementsByClassName('el-icon-view')[0]
        if (!this.pwdType) {
          e.setAttribute('style', 'color: #409EFF')
        } else {
          e.setAttribute('style', 'color: #c0c4cc')
        }
      },
      updateUploadingInfo(info) {
        this.$emit('updateUploadingInfo', info)
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
    margin-left: 245px;
  }

  .title {
    height: 42px;
    line-height: 42px;
    background: rgba(245, 247, 250, 1);
    border: 1px solid rgba(224, 229, 233, 1);
    font-size: 16px;
    text-align: center;
    font-weight: normal;
    color: rgba(50, 50, 50, 1);
  }

  /*期贷参数信息*/
  .futuresPart {
    padding-top: 20px;
    /*border-top: 1px solid rgba(224, 229, 233, 1);*/
  }

  .future-part-top-line {
    height: 1px;
    width: 100%;
    /*background:red;*/
    background: rgba(224, 229, 233, 1);
    margin-bottom: 10px;
  }

  .partTitle {
    font-size: 16px;
    font-weight: 400;
    color: rgba(50, 50, 50, 1);
    border-left: 3px solid #ff0000;
    /*margin-left: 5%;*/
    margin-bottom: 10px;
    padding: 0 10px;
  }

  .el-col-2 {
    width: auto;
    text-align: center;
  }

  .el-col-4 {
    width: 395px;
  }

  .futuresPartOne .el-col-4 {
    width: 237px;
  }

  .el-col-8 {
    width: 460px;
  }

  .el-col-10 {
    /*width: 38%;*/
  }

  /deep/ .el-form-item__label {
    font-size: 14px;
    font-weight: 400;
    color: rgba(100, 100, 100, 1);
  }

  /*输入框大小*/
  /deep/ .el-input .el-input__inner {
    width: 360px;
    height: 36px;
  }

  /deep/ .account_item.account.el-input .el-input__inner {
    width: 135px;
  }

  /deep/ .account_item.el-input .el-input__inner {
    width: 305px;
  }

  /deep/ .el-icon-view {
    line-height: 36px;
    margin-right: 10px;
  }
  /*
   /deep/ .el-form-item--mini.el-form-item {
    margin-right: -10px !important;
  }
  */


  .futuresPartOne /deep/ .el-form-item--mini.el-form-item {
    margin-right: 25px !important;
  }

</style>
