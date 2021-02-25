<template>
  <div class="upload-file-box">
    <div class="title parameter">
      用印文件
    </div>
    <div class="stamped-file-download-box">
      <span>用印文件:</span>
      <el-button icon="el-icon-download"
              class="capital-account-button"
              @click="downloadStampedOriginalFile()">资金账户信息告知函
      </el-button>
    </div>
    <upload-file
      ref="uploadFileRef"
      :upload-frame-id="uploadFrameId"
      :sup_this="form_sup_this"
      :upload-title="uploadTitle"
      :upload-url="uploadUrl"
      :export-file-url="exportFileUrl"
      :view-file-url="viewFileUrl"
      :file-number-limit="1"
      :if-multiple-file="ifMultipleFile"
      :file-max-size="fileMaxSize"
      :warning-text="warningText"
      :uploaded-file-info="uploadedFileInfo"
      :preview-pdf-src="uploadPDFSrc"
      @deleteUploadedFile="deleteUploadedFile"
      @handleFilePreview="handleFilePreview"
      @updateUploadingInfo="updateUploadingInfo"
      :accept-file-type="acceptFileType">
    </upload-file>
  </div>
</template>

<script>
import UploadFile from '@/views/components/UploadFile'
import {  removeUploadedPdf } from 'api/ceres/threePartiesAssociated'

export default {
  components: { UploadFile },
  props: {
    sup_this: {
      type: Object,
      required: false,
    },
    uploadedFileInfo: {
      type: Array,
      required: true,
    },
    uploadFrameId: {
      type: String,
      required: true,
    },
    //是否是多文件上传
    ifMultipleFile: {
      type: Boolean,
      required: true,
    },
    // 如果是多文件, 默认两个
    fileNumberLimit: {
      type: Number,
      required: false,
    },
    uploadTitle: {
      type: String,
      required: true,
    },
    // 上传地址
    uploadUrl: {
      type: String,
      required: true,
    },
    // 下载查看地址
    exportFileUrl: {
      type: String,
      required: true,
    },
    viewFileUrl: {
      type: String,
      required: true,
    },
    // 文件类型, 项目中基本为pdf
    acceptFileType: {
      type: String,
      required: true,
    },
    //单位大小为M
    fileMaxSize: {
      type: Number,
      required: true,
    },
    warningText: {
      type: String,
      required: true,
    },
    // 后台返回
    applyId: {
      type: String,
      required: false,
    },
  },
  data() {
    return {
      form_sup_this: this,
      uploadPDFSrc: ''
    }
  },
  watch: {
  },
  mounted() {
  },
  methods: {
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
    handleFilePreview(info){
      this.uploadPDFSrc = this.viewFileUrl + '?' + 'filePath=' + info.attachmentUrl
    },
    updateUploadingInfo(info) {
      this.$emit('updateUploadingInfo', info)
    },
    clearUploadInfo() {
      this.$nextTick(() => {
        this.$refs['uploadFileRef'].clearUploadChildInfo()
      })
    },
    // 下载资金账户告知函
    downloadStampedOriginalFile() {
      this.$emit('downloadStampedOriginalFile')
    },
  },
}
</script>

<style scoped lang="scss">
  .upload-file-box {
    padding: 10px;
  }

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

  .capital-account-button {
    cursor: pointer;
    display: inline-block;
    vertical-align: middle;
    padding-left: 10px;
    padding-right: 10px;
    padding-top: 5px;
    padding-bottom: 7px;
    border: 1px solid rgba(255, 130, 75, 1);
    border-radius: 2px;
    font-size: 13px;
    font-family: Source Han Sans SC;
    color: rgba(255, 130, 75, 1);
  }

  .stamped-file-download-box {
    padding-left: 28.75%;
    padding-top: 10px;
  }
</style>
