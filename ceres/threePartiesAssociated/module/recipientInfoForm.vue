<template>
  <div>
    <div class="part-recipient-box">
      <!--      <div class="future-part-top-line"></div>-->
      <div class="title fund">
        收件信息
      </div>
      <div class="info">
        <el-form ref="recipientInfoForm"
                 :model="recipientInfoForm"
                 :rules="computedFormRules"
                 :inline="true"
                 size="mini"
                 @submit.native.prevent
                 label-position="left">
          <el-row>
            <el-form-item label="快递单号:" label-width="156px" prop="expressNumber">
              <el-input
                v-model="recipientInfoForm.expressNumber"
                placeholder=""
                maxlength="20"
                class="filter-item"
                :disabled="computedGlobalReadOnly || exhibitMode"
                :readonly="computedGlobalReadOnly || exhibitMode"/>
            </el-form-item>

            <el-form-item label="收件人:" label-width="156px" prop="recipients" class="ml">
              <el-input
                v-model="recipientInfoForm.recipients"
                placeholder=""
                class="filter-item"
                maxlength="50"
                :disabled="computedGlobalReadOnly || exhibitMode"
                :readonly="computedGlobalReadOnly || exhibitMode"/>
            </el-form-item>

          </el-row>
        </el-form>
      </div>
      <!--  收件人上传文件-->
      <div class="recipient-upload-box">
        <upload-file
          ref="uploadFileRef"
          v-show="computedUploadFrameVisible"
          :upload-frame-id="frameName"
          :sup_this="form_sup_this"
          :upload-title="recipientUploadFileInfo.title"
          :upload-url="computedRecipientUploadUrl"
          :export-file-url="computedExportFileUrl"
          :view-file-url="computedViewFileUrl"
          :if-multiple-file="true"
          :file-number-limit="fileNumberLimit"
          attachment-title="附件:"
          :file-max-size="recipientUploadFileInfo.fileMaxSize"
          :warning-text="recipientUploadFileInfo.warningText"
          :uploaded-file-info="uploadedFileInfo"
          :preview-pdf-src="uploadPDFSrc"
          @handleFilePreview="handleFilePreview"
          @deleteUploadedFile="deleteUploadedFile"
          @updateUploadingInfo="updateUploadingInfo"
          :accept-file-type="recipientUploadFileInfo.acceptFileType">
        </upload-file>
      </div>

      <!-- 展示用用印文件附件列表-->
      <attachment
        v-if="!computedUploadFrameVisible && uploadedFileInfo.length > 0"
        attachment-title="附件"
        :delete-forbidden="true"
        :pdf-src="attachmentPDFSrc"
        @previewCurrentFile="previewCurrentFile"
        @downloadCurrentFile="downloadCurrentFile"
        :attachment="uploadedFileInfo">
      </attachment>
    </div>


  </div>
</template>

<script>
import Attachment from '@/views/components/Attachment'
import UploadFile from '@/views/components/UploadFile'
import store from '@/store'
import {
  threePartiesAssociatedExportPdfUrl,
  threePartiesAssociatedViewPdfUrl,
} from '../../../../api/ceres/threePartiesAssociated'
import { removeUploadedPdf } from 'api/ceres/threePartiesAssociated'

export default {
  name: 'RecipientInfoForm',
  components: { Attachment, UploadFile },
  props: {
    //  form
    recipientInfoForm: {
      type: Object,
      required: true,
    },
    globalReadOnly: {
      type: Boolean,
      required: true,
    },
    // upload 组件各种信息
    recipientUploadFileInfo: {
      type: Object,
      required: true,
    },
    // 已上传文件列表
    uploadedFileInfo: {
      type: Array,
      required: true,
    },
    // 是否展示上传插件
    uploadFrameVisible: {
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
    computedUploadFrameVisible() {
      if (this.computedGlobalReadOnly) {
        return false
      } else {
        return this.uploadFrameVisible
      }

    },
    computedGlobalReadOnly() {
      return this.globalReadOnly
    },
    computedRecipientUploadUrl() {
      return this.recipientUploadFileInfo.uploadUrlPath + '?' +
        'creatorId=' + store.getters.user.userId +
        '&fileType=' + this.recipientUploadFileInfo.fileType +
        '&applyId=' + this.recipientUploadFileInfo.applyId
    },
    computedExportFileUrl() {
      return this.recipientUploadFileInfo.exportFileUrlPath
    },
    computedViewFileUrl() {
      return this.recipientUploadFileInfo.viewFileUrlPath
    },
    computedFormRules() {
      if (this.exhibitMode || this.computedGlobalReadOnly) {
        return this.notRequiredRules
      } else {
        return this.rules
      }
    },
  },
  watch: {
    computedRecipientAttachment(val) {
      console.log(JSON.stringify(val))
    },
  },
  data() {
    const validateNumber = (rule, value, callback) => {
      if (!(/^[0-9a-zA-Z]+$/.test(value))) {
        callback(new Error('请输入数字或者英文字母!'))
      }
      callback()
    }
    return {
      fileNumberLimit: 200,
      form_sup_this: this,
      frameName: 'recipientFrameBox',

      // 子组件中文件预览src
      uploadPDFSrc: '',
      // 附件中预览的src
      attachmentPDFSrc: '',
      // 预览url前缀
      prefixFileViewPDFApiUrl: threePartiesAssociatedViewPdfUrl,
      // 下载前缀
      prefixFileDownloadApiUrl: threePartiesAssociatedExportPdfUrl,
      rules: {
        expressNumber: [
          { required: true, message: '请输入快递单号', trigger: ['blur', 'change'] },
          { validator: validateNumber, trigger: ['blur', 'change'] },
        ],
        recipients: [
          { required: true, message: '请输入收件人', trigger: ['blur', 'change'] },
        ],
      },
      notRequiredRules: {
        expressNumber: [
          { required: false, message: '', trigger: [] },
        ],
        recipients: [
          { required: false, message: '', trigger: [] },
        ],
      },
    }
  },
  mounted() {
    this.$nextTick(() => {

    })
  },
  methods: {
    initEvent() {

    },
    handleFilePreview(info) {
      this.uploadPDFSrc = this.viewFileUrl + '?' + 'filePath=' + info.attachmentUrl
    },
    deleteUploadedFile(params) {
      removeUploadedPdf(params)
      .then(res => {
        if (res.status === 'success') {
          this.$notify({
            title: '删除文件成功!',
            type: 'success',
            duration: 2500,
          })
          this.$refs['uploadFileRef'].deleteFileCallback(params, 'success')
        } else {
          this.$notify({
            title: '删除文件失败!',
            type: 'success',
            duration: 2500,
          })
          this.$refs['uploadFileRef'].deleteFileCallback('', 'failed')
        }
      })
      .catch(err => {
        this.$refs['uploadFileRef'].deleteFileCallback('', 'error')
        console.error(err.message)
      })
    },
    updateUploadingInfo(info) {
      this.$emit('updateUploadingInfo', info)
    },
    clearUploadInfo() {
      this.$nextTick(() => {
        this.$refs['uploadFileRef'].clearUploadChildInfo()
      })
    },
    previewCurrentFile(file) {
      this.attachmentPDFSrc = this.prefixFileViewPDFApiUrl + '?' + 'filePath=' + file.attachmentUrl
    },
    downloadCurrentFile(file) {
      const url = this.prefixFileDownloadApiUrl + '?' + 'fileName=' + file.attachmentName + '&'
        + 'filePath=' + file.attachmentUrl
      window.open(url)
    },
  },
}
</script>

<style scoped lang="scss">
.info {
  width: 1360px;
  margin: 0 auto;
  padding: 20px 0;
}

.ml {
  margin-left: 275px;
}

.title {
  background: rgba(245, 247, 250, 1);
  border: 1px solid rgba(224, 229, 233, 1);
  font-size: 16px;
  text-align: center;
  font-weight: normal;
  color: rgba(50, 50, 50, 1);
  margin-bottom: 15px;
}

.title.fund {
  height: 42px;
  line-height: 42px;
}

.el-col-10 {
  width: 40%;
  margin-left: 9%;
}

.part-recipient-box {
  padding-top: 20px;
}

.partTitle {
  font-size: 16px;
  font-weight: 400;
  color: rgba(50, 50, 50, 1);
  border-left: 3px solid #ff0000;
  margin-left: 5%;
  margin-bottom: 10px;
  padding: 0 10px;
}

/*输入框大小*/
/deep/ .el-input__inner {
  width: 360px;
  height: 36px;
}

.future-part-top-line {
  margin: 0 auto;
  height: 1px;
  width: 100%;
  background: rgba(224, 229, 233, 1);
  margin-bottom: 10px;
}

.recipient-upload-box {
  width: 1360px;
  margin: 0 auto;
  /*padding-left: 145px;*/
  /*padding-right: 5%;*/
}
</style>
