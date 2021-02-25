<template>
  <el-dialog
    :append-to-body="true"
    :visible.sync="associateDialog"
    :title="computedTitle"
    width="90%"
    @close="closeDialog"
    class="el-dialog-three-parties-associated"
    style="height: 90%;">
    <div class="form">
      <step-bar
        :sup_this="form_sup_this"
        :current-step="formFlag"
        :total-steps="totalSteps">
      </step-bar>
      <!-- 资金账户信息-->
      <capital-account-form
        ref="capitalAccountForm"
        v-show="computedVisibleByStep"
        :sup_this="form_sup_this"
        :exhibit-mode="computedExhibitMode"
        :edit-mode="computedEditMode"
        :signTypeForForm="signTypeForForm"
        :capital-account-form="capitalAccountForm"
        :global-read-only="computedEditModeByOnlyRead"
        @handleFuturesParamsForm="handleFuturesParamsForm"
        @updateCapitalAccount="updateCapitalAccount"
        @updateTrusteeshipInfo="updateTrusteeshipInfo">
      </capital-account-form>
      <!--银期签约---期货参数 -->
      <futures-params-form
        ref="futuresParamsForm"
        :sup_this="form_sup_this"
        :exhibit-mode="computedExhibitMode"
        :edit-mode="computedEditMode"
        :futures-params-form="futuresParamsForm"
        :global-read-only="computedEditModeByOnlyRead"
        :uploaded-file-info="computedRecipientAttachment"
        :recipient-upload-file-info="recipientUploadFile"
        v-show="showFutureParamsForm && computedVisibleByStep">
      </futures-params-form>
      <!-- 收件信息 -->
      <recipient-info-form
        v-show="computedVisibleForExpress"
        ref="recipientInfoForm"
        :recipient-info-form="recipientInfoForm"
        :recipient-upload-file-info="recipientUploadFile "
        :exhibit-mode="computedExhibitMode"
        :edit-mode="computedEditMode"
        :uploaded-file-info="computedRecipientAttachment"
        :upload-frame-visible="computedUploadFrameVisible"
        :global-read-only="computedEditModeByOnlyRead"
        @updateUploadingInfo="updateRecipientUploadingInfo"
        :sup_this="form_sup_this">
      </recipient-info-form>
      <!--  第3步骤, 用印文件  -->

      <stamped-file-form
        v-show="!computedVisibleByStep"
        ref="fileUploadRef"
        :upload-frame-id="frameName"
        @updateUploadingInfo="updateUploadingInfo"
        @downloadStampedOriginalFile="downloadStampedOriginalFile"
        :upload-title="uploadFile.title"
        :sup_this="form_sup_this"
        :upload-url="computedUploadUrl"
        :export-file-url="computedExportFileUrl"
        :if-multiple-file="false"
        :file-number-limit="1"
        :uploaded-file-info="computedStampedAttachment"
        :view-file-url="computedViewFileUrl"
        :file-max-size="uploadFile.fileMaxSize"
        :warning-text="uploadFile.warningText"
        :accept-file-type="uploadFile.acceptFileType">
      </stamped-file-form>
      <div class="exhibit-stamped-file-box" v-if="computedStampedAttachment.length">
        <div class="title parameter"
             v-show="computedExhibitMode && computedStampedAttachment.length">
          用印文件
        </div>
        <!-- 展示用用印文件附件列表-->
        <attachment
          v-show="computedExhibitMode && computedStampedAttachment.length"
          attachment-title="附件"
          :delete-forbidden="true"
          :pdf-src="attachmentPDFSrc"
          @previewCurrentFile="previewCurrentFile"
          @downloadCurrentFile="downloadCurrentFile"
          :attachment="computedStampedAttachment">
        </attachment>
      </div>
      <div class="form_footer">
        <el-button
                   @click="terminateFlow()"
                   v-show="processInstanceId && !isExhibit ">终止
        </el-button>
        <el-button
                   @click="refillFlowData()"
                   v-show="this.formFlag === 2 && !isExhibit">重填
        </el-button>
        <el-button @click="saveFlowInfo(false)"
                   v-show="!isExhibit && this.formFlag === 0 ">保存
        </el-button>
        <el-button type="primary" @click="doSubmit" v-show="!computedExhibitMode">提交</el-button>
        <el-button @click="cancelSubmit" v-if="computedExhibitMode">关闭</el-button>
        <el-button @click="cancelSubmit" v-else>取消</el-button>
      </div>
    </div>
  </el-dialog>
</template>

<script>
import store from '@/store/index'
import stepBar from '@/views/components/StepBar'
import capitalAccountForm from '@/views/ceres/threePartiesAssociated/module/capitalAccountForm'
import futuresParamsForm from '@/views/ceres/threePartiesAssociated/module/futuresParamsForm'
import stampedFileForm from '@/views/ceres/threePartiesAssociated/module/stampedFileForm'
import recipientInfoForm from '@/views/ceres/threePartiesAssociated/module/recipientInfoForm'
import Attachment from '@/views/components/Attachment'
import {
  handleApplyThreePartiesAssociateInfo,
  saveThreePartiesAssociateInfo,
  threePartiesAssociatedExportPdfUrl, threePartiesAssociatedStampedOriginalFileUrl,
  threePartiesAssociatedUploadUrl, threePartiesAssociatedViewPdfUrl,
} from 'api/ceres/threePartiesAssociated'

export default {
  name: '',
  components: {
    stepBar,
    Attachment,
    capitalAccountForm,
    futuresParamsForm,
    stampedFileForm,
    recipientInfoForm,
  },
  props: {
    sup_this: {
      type: Object,
      required: true,
    },
    allForms: {
      type: Object,
      required: true,
      default() {
        return {}
      },
    },
    //is-add 当前操作的模式
    isEdit: {
      type: Boolean,
      required: true,
      default: false,
    },
    isExhibit: {
      type: Boolean,
      required: true,
      default: false,
    },
    signTypeForForm: {
      type: String,
      required: true,
      default: 'sign',
    },
  },
  data() {
    return {
      // isEdit: false,
      // 当前dialog显隐控制
      associateDialog: false,
      // 当前form的this
      form_sup_this: this,

      signTypeSpecific: '',
      // 默认起始是从0开始
      formFlag: 0,
      totalSteps: [
        '申请',
        '初审',
        '上传用印',
        '复审',
        '三方关联',
        '完成',
      ],
      // 保存之后当前业务的id
      processRecordId: '',
      // 流程实例id
      processInstanceId: '',

      // 资金账户告知函url
      stampedOriginalFileUrl: threePartiesAssociatedStampedOriginalFileUrl,

      attachmentPDFSrc:'',
      prefixFileViewPDFApiUrl: threePartiesAssociatedViewPdfUrl,
      prefixFileDownloadApiUrl: threePartiesAssociatedExportPdfUrl,

      // 三方关联驳回时, 全局只读字段
      globalReadOnly: false,
      // 资金账户表单元素
      capitalAccountForm: {
        //产品代码
        productCode: '',
        // 签约类型
        singedType: '',
        // 托管账户名称
        escrowAccountId: '',
        // 托管账户账号

        //资金账号
        capitalAccount: '',
        //资金密码
        capitalPassword: '',
        //证件类型
        certificateType: '',
        //证件号码
        certificateNumber: '',
        // 签约券商/期货商
        signedContractId: '',
        //营业部
        salesDepartment: '',
        //  紧急联系人
        contactName: '',
        //  联系电话
        contactMobile: '',
      },
      // 默认不展示期货交易信息
      showFutureParamsForm: false,
      // 银期签约form决定是否使用表单
      futuresParamsForm: {
        // 联动的公司名称
        futuresCompanyCodeName: '',
        //中金所会员号
        cffexMid: '',
        //上期所会员号
        shfeMid: '',
        //大商所会员号
        dceMid: '',
        //郑商所会员号
        zceMid: '',
        //能源中心会员号
        ineMid: '',
        //开户行名称
        openAccountBank: '',
        //开户行网点
        bankOutlets: '',
        //期货公司保证金账号
        securityDepositAccount: '',
        //期货公司代码
        futuresCompanyCode: '',
        //内部资金账号
        internalCapitalAccount: '',
        // 中金所交易编码_投机
        cffexSpeculationCode: '',
        // 中金所交易编码_套保
        cffexHedgingCode: '',
        // 中金所交易编码_套利
        cffexSpreadingCode: '',
        // 上期所交易编码_投机
        shfeSpeculationCode: '',
        // 郑期所交易编码_投机
        zceSpeculationCode: '',
        // 大商所交易编码_投机
        dceSpeculationCode: '',
        // 期货监控中心_账户
        futuresMonitorAccount: '',
        // 期货监控中心_密码
        futuresMonitorPassword: '',
      },

      recipientInfoForm: {
        // 是否展示联系人
        visible: '',
        //快递单号
        expressNumber: '',
        //收件人
        recipients: '',
        //收件人附件列表
        attachment: [],
      },

      // upload 单独的id
      frameName: 'stampedUploadBox',
      // 控制文件状态,决定提交流程
      uploadSuccessful: false,
      // 用印文件上传组件必传参数
      uploadFile: {
        title: '',
        // 文件类型, 项目中基本为pdf
        acceptFileType: 'pdf',
        //单位大小为M
        fileMaxSize: 10,
        warningText: '文件最大为10M, 请注意!',
        // 上传文件地址
        uploadUrlPath: threePartiesAssociatedUploadUrl,
        // 文件下载地址
        exportFileUrlPath: threePartiesAssociatedExportPdfUrl,
        // 文件浏览地址
        viewFileUrlPath: threePartiesAssociatedViewPdfUrl,
        // 后台返回
        applyId: '',
        // 用印文件类型,  资金账户告知函
        fileType: '0601',
      },
      // 用印文件展示的附件
      stampedAttachment: [],

      //****************************************************
      // 控制文件状态,决定提交流程
      recipientUploadSuccessful: false,
      // 用印文件上传组件必传参数
      recipientUploadFile: {
        title: '附件:',
        // 文件类型, 项目中基本为pdf
        acceptFileType: 'pdf',
        //单位大小为M
        fileMaxSize: 10,
        //提示信息
        warningText: '如果在非本公司开户，请上传经纪服务协议。如果银期签约，请另外上传期货备忘录',
        // 上传文件地址
        uploadUrlPath: threePartiesAssociatedUploadUrl,
        // 文件下载地址
        exportFileUrlPath: threePartiesAssociatedExportPdfUrl,
        // 文件浏览地址
        viewFileUrlPath: threePartiesAssociatedViewPdfUrl,
        // 后台返回
        applyId: '',
        // 收件人文件类型, 经济服务协议或期货备忘录
        fileType: '01ZZ',
      },
      recipientAttachment: [],
    }
  },
  computed: {
    computedVisibleForExpress() {
      if (this.recipientInfoForm.visible === '1') {
        // 第三步编辑不显示, 展示状态显示
        if (this.formFlag === 2 && !this.computedExhibitMode) {
          return false
        } else {
          return true
        }
      } else {
        return false
      }
    },
    computedUploadFrameVisible() {
      // 如果是展示模式, 不展示上传组件
      return !this.computedExhibitMode
    },
    computedEditModeByOnlyRead() {
      // 驳回之后的编辑能力
      if (this.globalReadOnly === '1') {
        return true
      } else {
        if (this.computedExhibitMode) {
          return true
        } else {
          return false
        }
      }
    },
    computedUploadUrl() {
      return this.uploadFile.uploadUrlPath + '?' +
        'creatorId=' + store.getters.user.userId +
        '&fileType=' + this.uploadFile.fileType +
        '&applyId=' + this.uploadFile.applyId
    },
    computedExportFileUrl() {
      return this.uploadFile.exportFileUrlPath
    },
    computedViewFileUrl() {
      return this.uploadFile.viewFileUrlPath
    },
    computedExhibitMode() {
      return this.isExhibit
    },
    computedEditMode() {
      return this.isEdit
    },
    computedTitle() {
      if (this.isExhibit) {
        return '查看'
      } else {
        if (this.isEdit) {
          return '编辑'
        } else {
          return '新增'
        }
      }
    },
    computedVisibleByStep() {
      // 如果是第一步, 同时不是展示模式
      if (this.isExhibit) {
        return true
      } else {
        if (this.formFlag === 0) {
          return true
        } else {
          return false
        }
      }
    },
    computedStampedAttachment() {
      return this.stampedAttachment
    },
    computedRecipientAttachment() {
      return this.recipientAttachment
    },
  },
  watch: {},
  mounted() {
    this.$nextTick(() => {
    })
  },
  methods: {
    initEvent() {
      // 如果是新增
      if (!this.computedExhibitMode && !this.computedEditMode) {
        this.resetForm()
        this.$nextTick(() => {
          this.$refs['capitalAccountForm'].initEvent()
        })
      } else {
        // 更新数据, 消息队列
        // 在下次 DOM 更新循环结束之后执行延迟回调。
        // 在修改数据之后立即使用这个方法，获取更新后的 DOM
        this.$nextTick(() => {
          // 直接填补数据
          this.echoFormData()
        })
      }
    },
    // 展示还是隐藏银期签约表单
    handleFuturesParamsForm(data) {
      this.showFutureParamsForm = data.paramsFlag

      this.signTypeSpecific = data.signType
    },
    downloadStampedOriginalFile() {
      const url = this.stampedOriginalFileUrl + '?' + 'id=' + this.processRecordId
      window.open(url)
    },
    updateUploadingInfo(info) {
      // 更新文件的状态, 决定03状态时是否能够提交
      // console.log('uploadSuccessful',info)
      this.uploadSuccessful = info.success
      // 如果是删除文件, 清空存储集合
      if (!info.success) {
        if (this.stampedAttachment.length > 0) {
          this.stampedAttachment.splice(0, 1)
        }
      }
    },
    updateRecipientUploadingInfo(info) {
      // 更新文件的状态, 用户管理
      // console.log('recipientUploadSuccessful', info)

      // 文件增加或者删除
      if (info.success === true) {
        //增加文件
        this.recipientAttachment.push(info.fileInfo)
      } else {
        // 删除文件
        // console.log(info.attachmentId.id)

        let deleteIndex = 0
        let exist = false
        this.recipientAttachment.forEach((value, index) => {
          if (value.attachmentId === info.attachmentId.id) {
            exist = true
            deleteIndex = index
          }
        })
        if (exist) {
          this.recipientAttachment.splice(deleteIndex, 1)
        }
      }
      // 通过剩余文件数量决定上传成功于否的状态, 而不是每次上传,因为还有删除
      // console.log('recipientAttachment', this.recipientAttachment.length)
      this.recipientUploadSuccessful = this.recipientAttachment.length > 0

    },
    updateCapitalAccount(value) {
      this.futuresParamsForm.internalCapitalAccount = value
    },
    updateTrusteeshipInfo(item) {
      this.futuresParamsForm.futuresCompanyCodeName = item.signedContractName
      this.futuresParamsForm.cffexMid = item.cffexMid
      this.futuresParamsForm.dceMid = item.dceMid
      this.futuresParamsForm.ineMid = item.ineMid
      this.futuresParamsForm.shfeMid = item.shfeMid
      this.futuresParamsForm.zceMid = item.zceMid
      this.futuresParamsForm.signedContractId = item.signedContractId
    },
    getParametersForSave() {
      let capitalFormData = this.capitalAccountForm
      let futureFormData = this.futuresParamsForm
      let recipientFormData = this.recipientInfoForm

      let id = this.processRecordId
      // 后台的冗余字段,没什么作用
      let step = this.formFlag
      let data = {
        'id': id,
        'signedType': this.signTypeSpecific,
        'step': step,
        'creatorId': store.getters.user.userId,

        'capitalAccount': capitalFormData.capitalAccount,
        'capitalPassword': capitalFormData.capitalPassword,
        'certificateNumber': capitalFormData.certificateNumber,
        'certificateType': capitalFormData.certificateType,
        'contactMobile': capitalFormData.contactMobile,
        'contactName': capitalFormData.contactName,

        'escrowAccountId': capitalFormData.escrowAccountId,
        'escrowAccountName': capitalFormData.escrowAccountName,
        'escrowAccount': capitalFormData.escrowAccount,

        'productCode': capitalFormData.productCode,
        'signedContractId': capitalFormData.signedContractId,
        'salesDepartment': capitalFormData.salesDepartment,

        'cffexSpeculationCode': futureFormData.cffexSpeculationCode,
        'cffexHedgingCode': futureFormData.cffexHedgingCode,
        'cffexSpreadingCode': futureFormData.cffexSpreadingCode,

        'shfeSpeculationCode': futureFormData.shfeSpeculationCode,
        'zceSpeculationCode': futureFormData.zceSpeculationCode,
        'dceSpeculationCode': futureFormData.dceSpeculationCode,

        'cffexMid': futureFormData.cffexMid,
        'dceMid': futureFormData.dceMid,
        'ineMid': futureFormData.ineMid,
        'shfeMid': futureFormData.shfeMid,
        'zceMid': futureFormData.zceMid,

        'futuresMonitorPassword': futureFormData.futuresMonitorPassword,
        'futuresMonitorAccount': futureFormData.futuresMonitorAccount,

        'bankOutlets': futureFormData.bankOutlets,
        'openAccountBank': futureFormData.openAccountBank,
        'internalCapitalAccount': futureFormData.internalCapitalAccount,
        'futuresCompanyCode': futureFormData.futuresCompanyCode,

        'securityDepositAccount': futureFormData.securityDepositAccount,

        'recipients': recipientFormData.recipients,
        'expressNumber': recipientFormData.expressNumber,

      }
      // console.log(JSON.stringify(data))
      return data
    },
    saveFlowInfo(hideNotification, submitFunction) {
      const p1 = new Promise((resolve, reject) => {
        let form = this.$refs['capitalAccountForm'].$refs['capitalAccountForm']
        form.validate(valid => {
          if (valid) {
            resolve('good capital')
          } else {
            reject('capital error')
          }
        })
      })

      let value = []
      if (this.showFutureParamsForm) {
        const p2 = new Promise((resolve, reject) => {
          this.$refs['futuresParamsForm'].$refs['futuresParamsForm'].validate(valid => {
            if (valid) {
              resolve('good future')
            } else {
              reject('future error')
            }
          })
        })

        value = [p1, p2]
      } else {
        value = [p1]
      }

      // 如果展示收件信息, 需要表单校验
      if (this.computedVisibleForExpress) {
        const p3 = new Promise((resolve, reject) => {
          // 第一个是本层级ref名字,第二个是组件内的ref名字
          this.$refs['recipientInfoForm'].$refs['recipientInfoForm'].validate(valid => {
            if (valid) {
              resolve('good express')
            } else {
              reject('bad express')
            }
          })
        })
        value.push(p3)
      }

      Promise.all(value).then((res) => {
        let data = this.getParametersForSave()
        saveThreePartiesAssociateInfo(data).then((res) => {
          if (res.status === 'success') {
            // 如果不是提交按钮, 直接刷新
            if (!hideNotification) {
              this.toast('保存成功')

              this.associateDialog = false

              this.refreshTableDataList()

              this.resetDefaultData()
            } else {
              // 保存之后, 使用id继续提交
              if (res.data.id && submitFunction) {
                submitFunction(res.data.id)
              }
            }
          } else {
            this.toast(res.msg, 'warning')
          }
        }).catch((error) => {
          console.error(error)
        })

      }).catch((error) => {
        console.error(error)
      })
    },
    // 重填, 第三步存在
    refillFlowData() {
      let id = this.allForms.id
      if (id) {
        this.submitFunction(id, '1', '0', '重填成功', true)
      }

    },
    // 存在流程id的时候可以调用
    terminateFlow() {
      let id = this.allForms.id
      if (id) {
        this.submitFunction(id, '0', '1', '终止成功')
      }
    },
    doSubmit() {
      // 如果在第一步
      if (this.formFlag === 0) {
        // 普通状态直接提交
        if (this.recipientInfoForm.visible !== '1') {
          this.saveFlowInfo(true, this.submitFunction)
        } else {
          // 此时必须上传文件才能提交
          if (this.recipientUploadSuccessful) {
            this.saveFlowInfo(true, this.submitFunction)
          } else {
            this.toast('请上传文件')
          }
        }
      } else if (this.formFlag === 2) {
        // 隐藏通知, 表示在第三步
        if (this.uploadSuccessful) {
          let id = this.allForms.id
          if (id) {
            this.submitFunction(id)
          } else {
            console.log('id 为空')
          }
        } else {
          this.toast('请上传用印文件')
        }
      }
    },
    submitFunction(id, isRefilled = '0', isTerminated = '0', toastText = '提交成功', isReFillForm = false) {
      let data = {
        id: id,
        creatorId: store.getters.user.userId,
        isRefilled: isRefilled,
        isTerminated: isTerminated,
      }
      handleApplyThreePartiesAssociateInfo(data).then((res) => {
        if (res.status === 'success') {
          this.toast(toastText)

          // 重填标志, 其他状态isRefillForm为False值
          if (!isReFillForm) {
            this.associateDialog = false
            this.resetDefaultData()
          } else {
            // 重填的话, 回到第一步
            this.formFlag = 0
          }

          this.refreshTableDataList()
        } else {
          this.toast(res.msg, 'warning')
        }
      }).catch((error) => {
        console.error(error)
      })

    },
    /**
     *
     * @param attachments
     * @param code  // 收件人文件类型, 经济服务协议或期货备忘录   fileType: '01ZZ',
     *              // 用印文件类型,  资金账户告知函     fileType: '0601',
     * @returns {*[]}
     */
    filterAttachments(attachments, code) {
      // const stampedCode = '0601'
      // const protocol = '01ZZ'
      let array = []
      if (attachments && attachments.length > 0) {
        array = attachments.filter((value) => {
          return value.attachmentType === code
        })
        return array
      }
      return array
    },
    refreshTableDataList() {
      this.$parent.init()
    },
    resetDefaultData() {
      this.cancelSubmit()
    },
    cancelSubmit() {
      this.associateDialog = false
      this.showFutureParamsForm = false
      setTimeout(() => {
        this.$parent.isExhibitMode = false
        this.$parent.isEditMode = false
        this.formFlag = 0
        this.processRecordId = ''
        this.processInstanceId = ''
        this.stampedAttachment = []
        this.recipientAttachment = []
      }, 500)

      this.resetForm()

      this.clearUploadFileInfo()
      // 首先修改rule规则
      this.$emit('clearAllFormsData')
      // 随后重置表单数据
      this.$nextTick(() => {
        this.$refs['capitalAccountForm'].$refs['capitalAccountForm'].resetFields()
        this.$refs['futuresParamsForm'].$refs['futuresParamsForm'].resetFields()
        this.$refs['recipientInfoForm'].$refs['recipientInfoForm'].resetFields()
      })
    },
    closeDialog() {
      this.cancelSubmit()
    },
    echoFormData() {
      const data = this.allForms

      this.processRecordId = data.id

      // 流程实例id
      this.processInstanceId = data.processInstanceId

      // 用于处理文件上传的applyNumber
      if (data.id) {
        this.uploadFile.applyId = data.id
        this.recipientUploadFile.applyId = data.id
      }

      this.signTypeSpecific = data.signedType

      // 依赖类型
      if (data.signedType === '03') {
        this.showFutureParamsForm = true
      }

      // 步骤, 从1 开始, 组件从0开始
      if (data.flowStatus) {
        this.formFlag = parseInt(data.flowStatus) - 1
      } else {
        this.formFlag = 0
      }

      // 全局只读标识
      this.globalReadOnly = data.onlyRead

      this.capitalAccountForm = {
        //产品代码
        productCode: data.productCode,
        // 签约类型
        singedType: data.signedType,
        // 托管账户id
        escrowAccountId: data.escrowAccountId,
        // 托管账户名称
        escrowAccountName: data.escrowAccountName,
        // 托管账户账号
        escrowAccount: data.escrowAccount,
        //资金账号
        capitalAccount: data.capitalAccount,
        //资金密码
        capitalPassword: data.capitalPassword,
        //证件类型
        certificateType: data.certificateType,
        //证件号码
        certificateNumber: data.certificateNumber,
        // 签约券商/期货商
        signedContractId: data.signedContractId,
        //营业部
        salesDepartment: data.salesDepartment,
        //  紧急联系人
        contactName: data.contactName,
        //  联系电话
        contactMobile: data.contactMobile,
      }

      this.futuresParamsForm = {
        // 联动的公司名称, 是signedContractName字段
        futuresCompanyCodeName: data.signedContractName,
        //中金所会员号
        cffexMid: data.cffexMid,
        //上期所会员号
        shfeMid: data.shfeMid,
        //大商所会员号
        dceMid: data.dceMid,
        //郑商所会员号
        zceMid: data.zceMid,
        //能源中心会员号
        ineMid: data.ineMid,
        //开户行名称
        openAccountBank: data.openAccountBank,
        //开户行网点
        bankOutlets: data.bankOutlets,
        //期货公司保证金账号
        securityDepositAccount: data.securityDepositAccount,
        //期货公司代码
        futuresCompanyCode: data.futuresCompanyCode,
        //内部资金账号,
        internalCapitalAccount: data.internalCapitalAccount,
        // 中金所交易编码_投机
        cffexSpeculationCode: data.cffexSpeculationCode,
        // 中金所交易编码_套保
        cffexHedgingCode: data.cffexHedgingCode,
        // 中金所交易编码_套利
        cffexSpreadingCode: data.cffexSpreadingCode,
        // 上期所交易编码_投机
        shfeSpeculationCode: data.shfeSpeculationCode,
        // 郑期所交易编码_投机
        zceSpeculationCode: data.zceSpeculationCode,
        // 大商所交易编码_投机
        dceSpeculationCode: data.dceSpeculationCode,
        // 期货监控中心_账户
        futuresMonitorAccount: data.futuresMonitorAccount,
        // 期货监控中心_密码
        futuresMonitorPassword: data.futuresMonitorPassword,
      }

      this.recipientInfoForm = {
        // 返回字段是这么写的, 显然不标准
        // visibleExpress: data.visibleExpress,
        // 是否展示快递 收件人对象
        visible: data.visible,
        recipients: data.recipients,
        expressNumber: data.expressNumber,
      }

      // 附件综合
      if (data.attachments) {
        this.recipientAttachment = this.filterAttachments(data.attachments, '01ZZ')
        if (this.recipientAttachment.length > 0) {
          this.recipientUploadSuccessful = true
        }

        // 理论上是一个文件
        this.stampedAttachment = this.filterAttachments(data.attachments, '0601')
        // 如果有用印文件,是直接可以提交的
        if (this.stampedAttachment.length > 0) {
          this.uploadSuccessful = true
        }

      } else {
        this.recipientAttachment = []
        this.recipientUploadSuccessful = false

        this.stampedAttachment = []
        this.uploadSuccessful = false
      }

    },
    previewCurrentFile(file) {
      this.attachmentPDFSrc  = this.prefixFileViewPDFApiUrl + '?' + 'filePath=' + file.attachmentUrl
    },
    downloadCurrentFile(file) {
      const url = this.prefixFileDownloadApiUrl + '?' + 'fileName=' + file.attachmentName + '&'
        + 'filePath=' + file.attachmentUrl
      window.open(url)
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
    clearUploadFileInfo() {
      // 控制文件状态,决定提交流程
      this.uploadSuccessful = false
      // 用印文件上传组件必传参数
      this.uploadFile = {
        title: '',
        // 文件类型, 项目中基本为pdf
        acceptFileType: 'pdf',
        //单位大小为M
        fileMaxSize: 10,
        warningText: '文件最大为10M, 请注意!',
        // 上传文件地址
        uploadUrlPath: threePartiesAssociatedUploadUrl,
        // 文件下载地址
        exportFileUrlPath: threePartiesAssociatedExportPdfUrl,
        // 文件浏览地址
        viewFileUrlPath: threePartiesAssociatedViewPdfUrl,
        // 后台返回
        applyId: '',
        // 用印文件类型,  资金账户告知函
        fileType: '0601',
      }

      // 控制文件状态,决定提交流程
      this.recipientUploadSuccessful = false
      // 用印文件上传组件必传参数
      this.recipientUploadFile = {
        title: '附件:',
        // 文件类型, 项目中基本为pdf
        acceptFileType: 'pdf',
        //单位大小为M
        fileMaxSize: 10,
        warningText: '如果在非本公司开户，请上传经纪服务协议。如果银期签约，请另外上传期货备忘录。',
        // 上传文件地址
        uploadUrlPath: threePartiesAssociatedUploadUrl,
        // 文件下载地址
        exportFileUrlPath: threePartiesAssociatedExportPdfUrl,
        // 文件浏览地址
        viewFileUrlPath: threePartiesAssociatedViewPdfUrl,
        // 后台返回
        applyId: '',
        // 收件人文件类型, 经济服务协议或期货备忘录
        fileType: '01ZZ',
      }

      this.$nextTick(() => {
        this.$refs['fileUploadRef'].clearUploadInfo()
        this.$refs['recipientInfoForm'].clearUploadInfo()
      })
    },
    resetForm() {
      this.globalReadOnly = false
      // 清除本地元素
      this.capitalAccountForm = {
        //产品代码
        productCode: '',
        // 签约类型
        singedType: '',
        // 托管账户名称
        escrowAccountId: '',
        // 托管账户账号

        //资金账号
        capitalAccount: '',
        //资金密码
        capitalPassword: '',
        //证件类型
        certificateType: '',
        //证件号码
        certificateNumber: '',
        // 签约券商/期货商
        signedContractId: '',
        //营业部
        salesDepartment: '',
        //  紧急联系人
        contactName: '',
        //  联系电话
        contactMobile: '',
      }

      this.futuresParamsForm = {
        // 联动的公司名称
        futuresCompanyCodeName: '',
        //中金所会员号
        cffexMid: '',
        //上期所会员号
        shfeMid: '',
        //大商所会员号
        dceMid: '',
        //郑商所会员号
        zceMid: '',
        //能源中心会员号
        ineMid: '',
        //开户行名称
        openAccountBank: '',
        //开户行网点
        bankOutlets: '',
        //期货公司保证金账号
        securityDepositAccount: '',
        //期货公司代码
        futuresCompanyCode: '',
        //内部资金账号
        internalCapitalAccount: '',
        // 中金所交易编码_投机
        cffexSpeculationCode: '',
        // 中金所交易编码_套保
        cffexHedgingCode: '',
        // 中金所交易编码_套利
        cffexSpreadingCode: '',
        // 上期所交易编码_投机
        shfeSpeculationCode: '',
        // 郑期所交易编码_投机
        zceSpeculationCode: '',
        // 大商所交易编码_投机
        dceSpeculationCode: '',
        // 期货监控中心_账户
        futuresMonitorAccount: '',
        // 期货监控中心_密码
        futuresMonitorPassword: '',

      }
      this.recipientInfoForm = {
        // 是否展示联系人
        visible: '',
        //快递单号
        expressNumber: '',
        //收件人
        recipients: '',
        //收件人附件列表
        attachment: [],
      }
    },
  },
}
</script>

<style scoped lang="scss">

  .form {
    background: white;
    padding: 10px;
    width: 100%;
    font-family: Source Han Sans SC;
  }

  .form_footer {
    text-align: center;
    margin-top: 15px;

    .el-button {
      width: 120px;
      height: 42px;
    }
  }

  /deep/ .tip-size-intro{
    width: 290px !important;
  }

  /deep/ .el-form-item__label {
    font-size: 14px;
    font-weight: 400;
    color: rgba(100, 100, 100, 1);
  }

  /*输入框大小*/
  /deep/ .el-input__inner {
    width: 360px;
    height: 36px !important;
  }

  /deep/ .upload-file-box form.el-form .el-form-item.el-form-item--small .el-form-item__content{
    margin-left: 30% !important;
  }
  /deep/ form.el-form .el-form-item.el-form-item--small .el-form-item__label{
    width: 6% !important;
    padding: 0 40px 0 0;
  }
  /deep/ form.el-form .el-form-item.el-form-item--small .el-form-item__content{
    margin-left: 11% !important;
  }


  .exhibit-stamped-file-box, .exhibit-recipient-file-box {
    padding: 10px;

    .title.parameter {
      height: 42px;
      line-height: 42px;
      background: rgba(245, 247, 250, 1);
      border: 1px solid rgba(224, 229, 233, 1);
      font-size: 16px;
      text-align: center;
      font-weight: normal;
      color: rgba(50, 50, 50, 1);
    }
  }
</style>
<style lang="scss">
  .el-dialog__wrapper.el-dialog-three-parties-associated {
    top: 5vh;
    bottom: 5vh;

    .el-dialog {
      margin-top: 0 !important;
    }
  }

  .el-dialog-three-parties-associated{
    .el-dialog{
      min-width:1360px;
    }

  }

</style>
