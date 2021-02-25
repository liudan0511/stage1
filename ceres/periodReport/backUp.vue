<template>
  <div class="backUp">
    <el-form
      ref="form"
      :inline="true"
      size="mini"
      @submit.native.prevent
      :label-position="labelPosition"
    >
      <div class="manger">管理人信息</div>
      <div class="mangerInfo">
        <el-row>
          <el-form-item label="名称:" label-width="156px">
            <el-input v-model="form.managerName" style="width: 320px;" class="filter-item"/>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="法务代表人:" label-width="156px">
            <el-input
              v-model="form.legalRepresentative"
              placeholder
              style="width: 320px;"
              class="filter-item"
            />
          </el-form-item>
          <el-form-item label="注册地址:" label-width="156px" class="ml298">
            <el-input v-model="form.registeredAddress" style="width: 320px;" class="filter-item"/>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="办公地址:" label-width="156px">
            <el-input v-model="form.officeAddress" style="width: 320px;" class="filter-item"/>
          </el-form-item>
          <el-form-item label="邮编:" label-width="156px" class="ml298">
            <el-input
              v-model="form.postalCode"
              placeholder
              style="width: 320px;"
              class="filter-item"
            />
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="信披负责人姓名:" label-width="156px">
            <el-input
              v-model="form.disclosureName  "
              placeholder
              style="width: 320px;"
              class="filter-item"
            />
          </el-form-item>
          <el-form-item label="信披负责人联系方式:" label-width="156px" class="ml298">
            <el-input v-model="form.disclosureContact" style="width: 320px;" class="filter-item"/>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="信披负责人传真:" label-width="156px">
            <el-input
              v-model="form.disclosureFax"
              placeholder
              style="width: 320px;"
              class="filter-item"
            />
          </el-form-item>
          <el-form-item label="信披负责人电子邮件:" label-width="156px" class="ml298">
            <el-input
              v-model="form.disclosureEmail"
              placeholder
              style="width: 320px;"
              class="filter-item"
            />
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="会计师事务所名称:" label-width="156px">
            <el-input
              v-model="form.accountingFirm"
              placeholder="请填写基金管理人每年聘请的会计事务所名称"
              style="width: 320px;"
              class="filter-item"
            />
          </el-form-item>
          <el-form-item label="会计师事务所办公地址:" label-width="157px" class="ml298">
            <el-input
              v-model="form.accountingFirmOfficeAddress"
              style="width: 320px;"
              class="filter-item"
            />
          </el-form-item>
        </el-row>
      </div>
      <div class="product">产品信息</div>
      <div class="proInfo">
        <el-row>
          <el-form-item label="产品名称:" label-width="156px" prop="fundCode">
            <template slot-scope="scope">
              <el-select
                filterable
                placeholder="请输入产品代码或产品名称"
                v-model="form.fundCode"
                style="width: 320px;"
                @change="onSelectedDrug($event)"
              >
                <el-option
                  v-for="item in options"
                  :key="item.code"
                  :label="item.name"
                  :value="item.code"
                  style="width: 320px;"
                ></el-option>
              </el-select>
            </template>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="信息披露方式:" label-width="156px" pop="disclosureMethod">
            <el-input
              type="textarea"
              autosize
              :row="10"
              v-model="form.disclosureMethod"
            ></el-input>
          </el-form-item>
          <el-tooltip popper-class="tooltip-redesign-class" :content="tipText" placement="right">
            <el-button icon="el-icon-preview"></el-button>
          </el-tooltip>
        </el-row>
        <el-row>
          <el-form-item label="投资目标:" label-width="156px" prop="investmentObjectives">
            <el-input
              type="textarea"
              autosize
              :row="10"
              placeholder="请按照基金合同约定填写"
              v-model="form.investmentObjectives"
            ></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="投资策略:" label-width="156px" prop="investmentStrategy">
            <el-input
              type="textarea"
              autosize
              :row="10"
              placeholder="请按照基金合同约定填写"
              v-model="form.investmentStrategy"
            ></el-input>
          </el-form-item>
        </el-row>

        <div v-if="obj.type==='2'">
          <el-row>
            <el-form-item label="投资范围:" label-width="156px" prop="investmentScope">
              <el-input
                type="textarea"
                autosize
                :row="10"
                placeholder="请按照基金合同约定填写"
                v-model="form.investmentScope"
              ></el-input>
            </el-form-item>
          </el-row>
          <el-row>
            <el-form-item label="关注行业:" label-width="156px" prop="attentionIndustry">
              <el-input
                type="textarea"
                autosize
                :row="10"
                placeholder="请按照基金合同约定填写"
                class="filter-item"
                v-model="form.attentionIndustry"
              ></el-input>
            </el-form-item>
          </el-row>
          <el-row>
            <el-form-item label="关注阶段:" label-width="156px" prop="attentionStage">
              <el-input
                type="textarea"
                autosize
                :row="10"
                placeholder="请按照基金合同约定填写"
                class="filter-item"
                v-model="form.attentionStage"
              ></el-input>
            </el-form-item>
          </el-row>
          <el-row>
            <el-form-item label="其他需说明事项:" label-width="156px" prop="otherMatter">
              <el-input
                type="textarea"
                autosize
                :row="10"
                placeholder="请按照基金合同约定填写"
                class="filter-item"
                v-model="form.otherMatter"
              ></el-input>
            </el-form-item>
          </el-row>
        </div>
        <div v-else>
          <el-row>
            <el-form-item label="业绩比较基准:" label-width="156px" prop="performanceBaseline">
              <el-input
                type="textarea"
                autosize
                :row="10"
                placeholder="请按照基金合同约定填写"
                v-model="form.performanceBaseline"
              ></el-input>
            </el-form-item>
          </el-row>
          <el-row>
            <el-form-item label="风险收益特征:" label-width="156px" prop="riskReturnCharacteristics">
              <el-input
                type="textarea"
                autosize
                :row="10"
                placeholder="请根据产品实际情况，格式举例：高风险高收益、低风险低收益"
                class="filter-item"
                v-model="form.riskReturnCharacteristics"
              ></el-input>
            </el-form-item>
          </el-row>
        </div>
      </div>
      <div class="footer">
        <el-button type="primary" @click="doSubmit">保存</el-button>
        <el-button type="primary" @click="nextStep">下一步</el-button>
      </div>
    </el-form>
  </div>
</template>

<script>
  import store from "@/store/index";
  import {
    getManager,
    getProInfo,
    doSubmit,
    managerFund
  } from "@/api/ceres/periodReport";
  import Bus from '@/utils/eventBus'

  export default {
    name: "BackUp",
    props: {
      sup_this: {
        type: Object,
        default: null
      }
    },
    data() {
      return {
        tipText:
          "指私募基金管理人向投资者进行基金信息披露的方式，" +
          "包括采取信件、传真、电子邮件、官方网站或第三方服务机构登录查询等非公开披露的方式。" +
          "如信息披露方式为官方网站或第三方服务机构登录查询，应填写相关网址。",
        obj: {
          type: ""
        },
        labelPosition: "left",
        value: "",
        // flag: true,
        options: [],
        hintFlag: false,
        form: {
          //管理人信息
          managerName: "",
          // 法务代表人
          legalRepresentative: "",
          //注册地址
          registeredAddress: "",
          // 办公地址
          officeAddress: "",
          // 邮编
          postalCode: "",
          // 信披负责人姓名
          disclosureName: "",
          // 信披负责人联系方式
          disclosureContact: "",
          // 信披负责人传真
          disclosureFax: "",
          // 信披负责人电子邮件
          disclosureEmail: "",
          // 会计师事务所名称
          accountingFirm: "",
          // 会计师事务所办公地址
          accountingFirmOfficeAddress: "",
          //产品信息

          fundCode: "",
          // 信息披露方式
          disclosureMethod: "",
          // 投资目标
          investmentObjectives: "",
          // 投资策略
          investmentStrategy: "",
          // 业绩比较基准
          performanceBaseline: "",
          // 风险收益特征
          riskReturnCharacteristics: "",
          //投资范围
          investmentScope: '',
          //关注行业
          attentionIndustry: '',
          // 关注阶段
          attentionStage: '',
          // 其他需说明事项
          otherMatter: ''
        }
      };
    },
    created() {
      this.getManager();
    },
    mounted() {
      this.getProInfo();
    },
    methods: {
      //获取管理人信息
      getManager() {
        let form = this.form
        getManager(
          store.getters.user.userId
          // 'CS02955'
        ).then(res => {
          if (res.status === 'success') {
            let result = res.data
            if (result) {
              form.managerName = result.managerName;
              form.legalRepresentative = result.legalRepresentative;
              form.registeredAddress = result.registeredAddress;
              form.officeAddress = result.officeAddress;
              form.postalCode = result.postalCode;
              form.disclosureName = result.disclosureName;
              form.disclosureContact = result.disclosureContact;
              form.disclosureFax = result.disclosureFax;
              form.disclosureEmail = result.disclosureEmail;
              form.accountingFirm = result.accountingFirm;
              form.accountingFirmOfficeAddress = result.accountingFirmOfficeAddress;
            }
          } else {
            this.$notify({
              title: res.msg,
              type: "error",
              duration: 2500
            });
          }
        });
      },

      //获取所有产品
      getProInfo() {
        getProInfo(
          store.getters.user.userId
        ).then(res => {
          if (res.status === 'success') {
            if (res.data) {
              this.options = res.data;
            }
          } else {
            this.$notify({
              title: res.msg,
              type: "error",
              duration: 2500
            });
          }
        });
      },

      //1为证券类，2为股权类
      //  触发下拉框
      onSelectedDrug(event) {
        this.fundCode = event
        this.options.map(val => {
          if (event === val.code) {
            this.obj = val;
          }
        });
        managerFund(event).then(res => {
          if (res.status === 'success') {
            let result = res.data
            let form = this.form
            if (result) {
              form.disclosureMethod = result.disclosureMethod
              form.investmentObjectives = result.investmentObjectives
              form.investmentStrategy = result.investmentStrategy
              form.performanceBaseline = result.performanceBaseline
              form.riskReturnCharacteristics = result.riskReturnCharacteristics
              form.investmentScope = result.investmentScope
              form.attentionIndustry = result.attentionIndustry
              form.attentionStage = result.attentionStage
              form.otherMatter = result.otherMatter
            }
          } else {
            this.$notify({
              title: res.msg,
              type: "error",
              duration: 2500
            });
          }
        });
      },
      //保存
      doSubmit() {
        let form = this.form
        let data = {
          /*管理员*/
          userId: store.getters.user.userId,
          managerName: form.managerName,
          legalRepresentative: form.legalRepresentative,
          registeredAddress: form.registeredAddress,
          officeAddress: form.officeAddress,
          postalCode: form.postalCode,
          disclosureName: form.disclosureName,
          disclosureContact: form.disclosureContact,
          disclosureFax: form.disclosureFax,
          disclosureEmail: form.disclosureEmail,
          accountingFirm: form.accountingFirm,
          accountingFirmOfficeAddress: form.accountingFirmOfficeAddress,
          /*产品信息*/
          fundCode: form.fundCode,
          disclosureMethod: form.disclosureMethod,
          investmentObjectives: form.investmentObjectives,
          investmentStrategy: form.investmentStrategy,
          performanceBaseline: form.performanceBaseline,
          riskReturnCharacteristics: form.riskReturnCharacteristics,
          investmentScope: form.investmentScope,
          attentionIndustry: form.attentionIndustry,
          attentionStage: form.attentionStage,
          otherMatter: form.otherMatter
        };
        doSubmit(data).then(res => {
          if (res.status === 'success') {
            this.$notify({
              title: "保存成功",
              type: "success",
              duration: 2500
            });
          } else {
            this.$notify({
              title: res.msg,
              type: "error",
              duration: 2500
            });
          }
        });
      },

      //下一步
      nextStep() {
        this.doSubmit()
        if (this.fundCode) {
          Bus.$emit('getFundCode', this.fundCode)
        }
        this.sup_this.activeName = "manager";
      }
    }
  };
</script>
<!--慎用全局样式-->
<style lang="scss">
  /*修改黄色感叹号按钮样式*/
  .el-tooltip__popper.is-dark.tooltip-redesign-class {
    width: 200px;
    max-width: 220px;
    background: white;
    color: #ff824b;
    border: 1px solid #ff824b;
  }

  .el-tooltip__popper[x-placement^="right"] .popper__arrow {
    border-right-color: #ff824b;
    border-left-color: #ff824b;
  }

  .el-tooltip__popper[x-placement^="left"] .popper__arrow {

    border-right-color: #ff824b;
    border-left-color: #ff824b;
  }

  /*.el-tooltip__popper[x-placement^="top"] .popper__arrow {*/
  /*  border-right-color: #ff824b;*/
  /*  border-left-color: #ff824b;*/

  /*}*/

  /*.el-tooltip__popper[x-placement^="bottom"] .popper__arrow {*/
  /*  border-right-color: #ff824b;*/
  /*  border-left-color: #ff824b;*/
  /*}*/

  .el-tooltip__popper.is-dark.tooltip-redesign-class .popper__arrow::after {
    border-right-color: #fff;
    border-left-color: #fff;
  }

</style>

<style scoped>

  /deep/ .el-select__tags .el-select__input {
    width: 100%;
  }

  /deep/ .el-input__inner {
    height: 30px;
  }

  .manger,
  .product {
    width: 100%;
    height: 42px;
    line-height: 42px;
    background: rgba(245, 247, 250, 1);
    border: 1px solid rgba(224, 229, 233, 1);
    font-size: 16px;
    font-family: Source Han Sans SC;
    padding-left: 15px;
  }

  .mangerInfo,
  .proInfo {
    width: 1284px;
    margin: 0 auto;
    color: rgba(100, 100, 100, 1);
    padding: 20px 0px;
  }

  .ml298 {
    margin-left: 245px;
  }

  /*文本框样式*/
  /deep/ .el-textarea {
    width: 1055px;
  }

  /deep/ .el-textarea__inner {
    resize: none;
  }

  .footer {
    text-align: center;
  }

  .footer .el-button {
    width: 120px;
    height: 42px;
  }

  /deep/ .proInfo .el-button {
    border: none;
    padding: 0;
    position: absolute;
    bottom: 25px;
  }

  /deep/ .el-icon-preview {
    background: url("../../../assets/periodReport/hint.png") center no-repeat;
    background-size: cover;
    /*margin-left: -11px;*/
  }

  /deep/ .el-icon-preview:before {
    content: "替";
    font-size: 16px;
    visibility: hidden;
  }
</style>
