<template>
  <el-dialog
    v-el-drag-dialog
    :append-to-body="true"
    :title="title"
    :visible.sync="dialogFormVisible"
    width="80%"
    :before-close="closeDialog"
  >
    <el-form ref="form" :model="form" :rules="rules" size="middle">
      <el-row class="content">
        <el-form-item label="内容:" prop="replyContent" label-width="60px">
          <el-input
            type="textarea"
            autosize
            :row="5"
            placeholder="请按照基金合同约定填写"
            v-model="form.replyContent"
            maxlength="1000"
          ></el-input>
        </el-form-item>
      </el-row>
      <el-row class="file">
        <upload-file
          ref="uploadFileRef"
          :upload-frame-id="frameName"
          :sup_this="form_sup_this"
          :upload-title="UploadFile.title"
          :upload-url="uploadUrl"
          :export-file-url="UploadFile.exportFileUrlPath"
          :view-file-url="UploadFile.viewFileUrlPath"
          :if-multiple-file="true"
          :file-max-size="UploadFile.fileMaxSize"
          :warning-text="UploadFile.warningText"
          @updateUploadingInfo="updateUploadingInfo"
          :accept-file-type="UploadFile.acceptFileType"
          :fileNumberLimit="fileNumberLimit"
          :preview-pdf-src="attachmentPDFSrc"
          @handleFilePreview="handleFilePreview"
          @updateDeleteFileInfo="updateDeleteFileInfo"
          @downloadCurrentFile="downloadCurrentFile"
          v-show="computedUploadSuccess"
        >
        </upload-file>
        <!--        展示上传的附件-->
        <attachment
          ref="attachment"
          attachment-title="附件"
          :pdf-src="attachmentPDFSrc"
          @updateDeleteFileInfo="updateDeleteFileInfo"
          @downloadCurrentFile="downloadCurrentFile"
          :attachment="recipientAttachment"
        >
        </attachment>
      </el-row>
    </el-form>
    <div style="text-align:center;margin-bottom: 10px">
      <el-button type="primary" @click="doSubmit">提交</el-button>
      <el-button @click="cancel">取消</el-button>
    </div>
    <el-table
      :data="data"
      ref="tableRef"
      border
      stripe
      size="small"
      style="width: 100%">
      <el-table-column
        prop="createTime"
        label="回复时间"
        width="150"
      >
      </el-table-column>
      <el-table-column
        label="回复人"
        :show-overflow-tooltip="true"
        width="150"
      >
        <template slot-scope="scope">
          {{scope.row.creatorName+'('+getType(scope.row.replierType)+')'}}
        </template>
      </el-table-column>
      <el-table-column
        prop="replyContent"
        label="回复内容"
        :show-overflow-tooltip="true"
      >
      </el-table-column>
      <el-table-column
        prop="attachmentId"
        label="附件"
      >
        <template slot-scope="scope">
          <attachment
            ref="attachment"
            attachment-title="附件"
            :delete-forbidden="true"
            :pdf-src="attachmentPDFSrc"
            @previewCurrentFile="previewCurrentFile"
            :attachment="disposeAttachments(scope.row.attachments)">
          </attachment>

        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      :total="total"
      :current-page="page+1"
      style="margin-top: 8px;"
      layout="prev, pager, next,jumper,total, sizes"
      :page-size="size"
      :page-sizes="[10, 50, 100]"
      @size-change="sizeChange"
      @current-change="pageChange"
    />
  </el-dialog>
</template>

<script>
  import store from "@/store/index"
  import initData from '@/mixins/initData'
  import UploadFile from '@/views/components/UploadFile'
  import Attachment from '@/views/components/Attachment'
  import {
    riskWarningExportPdfUrl,
    riskWarningViewPdfUrl,
    riskWarningUploadUrl,
    submitReply,
    removeReplyPdf,
    replyBaseUrl,
  } from 'api/ceres/riskWarning'

  export default {
    mixins: [initData],
    components: {UploadFile, Attachment},
    props: {
      sup_this: {
        type: Object,
        required: true,
      },
      title: {
        type: String,
        required: true
      },
      dialogFormVisible: {
        type: Boolean,
        required: true
      },
      formAlertData: {
        type: Object,
      },
    },
    data() {
      return {
        //index传给form的id
        publicId: '',
        fileNumberLimit: 2,
        total: 0,
        form: {
          //回复意见
          replyContent: ''
        },
        rules: {
          replyContent: [
            {required: true, message: "请输入内容(最长为1000字)", trigger: ["blur"]},
          ],
        },
        form_sup_this: this,
        attachmentPDFSrc: '',
        frameName: 'form',
        // 用于展示的附件
        recipientAttachment: [],
        UploadFile: {
          title: '附件:',
          // 文件类型, 项目中基本为pdf
          acceptFileType: 'pdf',
          //单位大小为M
          fileMaxSize: 10,
          //提示信息
          warningText: '',
          // // 上传文件地址
          uploadUrlPath: riskWarningUploadUrl,
          // // 文件下载地址
          exportFileUrlPath: riskWarningExportPdfUrl,
          // // 文件浏览地址
          viewFileUrlPath: riskWarningViewPdfUrl,
          // 后台返回
          applyId: '',
          // 收件人文件类型
          fileType: '03ZZ',
        },
        attachmentIds: [],
        replierType: {
          '01': '管理人',
          '02': '托管人'
        }
      }
    },
    computed: {
      computedAcceptFileType() {
        let dot = '.'
        if (!this.acceptFileType) {
          return dot + 'pdf'
        } else {
          return dot + this.acceptFileType
        }
      },
      uploadUrl() {
        return this.UploadFile.uploadUrlPath + '?' +
          'creatorId=' + store.getters.user.userId +
          '&fileType=' + this.UploadFile.fileType +
          '&productCode=' + this.formAlertData.productCode
      },
      computedExportFileUrl() {
        return this.uploadFile.exportFileUrlPath
      },
      computedViewFileUrl() {
        return this.uploadFile.viewFileUrlPath
      },
      computedUploadSuccess() {
        if (this.recipientAttachment.length === 0) {
          return true
        } else if (this.recipientAttachment.length >= this.fileNumberLimit) {
          return false
        } else {
          return true
        }
      }

    },
    mounted() {
      this.handleFilePreview()
    },
    methods: {
      getType(type) {
        return this.replierType[type]
      },
      disposeAttachments(data) {
        data.forEach(item => {
          item.attachmentName = item.archiveName
        })
        return data
      },
      //上传成功之后获取附件的所有id
      handleFilePreview(fileInfo) {
        if (fileInfo) {
          this.attachmentIds.push(fileInfo.attachmentId)
          this.attachmentPDFSrc = riskWarningViewPdfUrl + '/' + fileInfo.archiveName + '?replyAttachId=' + fileInfo.attachmentId + '&creatorId=' + store.getters.user.userId
        }
      },
      beforeInit() {
        this.url = replyBaseUrl
        this.params = {
          page: this.page,
          size: this.size,
          investmentRiskId: this.publicId,
        }
        return true
      },
      doSubmit() {
        this.form.investmentRiskId = this.formAlertData.id
        this.form.creatorId = store.getters.user.userId
        this.form.attachmentids = this.attachmentIds.join()
        this.$refs["form"].validate(valid => {
          if (valid) {
            submitReply(this.form).then(res => {
              if (res.status === 'success') {
                this.init()
                this.form.replyContent = ''
                this.recipientAttachment = []
                this.attachmentIds = []
              } else {
                this.$notify({
                  title: res.msg,
                  type: 'warning',
                  duration: 2500
                })
              }
            })
          }

        })
      },
      updateUploadingInfo(info) {
        // 文件增加或者删除
        if (info.success === true) {
          //增加文件
          this.recipientAttachment.push(info.fileInfo)
          this.recipientAttachment.map(item => {
            item.attachmentName = item.archiveName
          })
        } else {
          // 删除文件
          let deleteIndex = 0
          let exist = false
          this.recipientAttachment.forEach((value, index) => {
            if (value.attachmentId === info.attachmentId) {
              exist = true
              deleteIndex = index
            }
          })

          if (exist) {
            this.recipientAttachment.splice(deleteIndex, 1)
            this.attachmentIds.splice(deleteIndex, 1)
          }
        }
      },

      //删除附件
      updateDeleteFileInfo(file) {
        let params = {
          replyAttachId: file.attachmentId,
          creatorId: store.getters.user.userId
        }
        removeReplyPdf(params)
          .then(res => {
            if (res.status === 'success') {
              this.$notify({
                title: '删除文件成功!',
                type: 'success',
                duration: 2500,
              })
              this.$refs['uploadFileRef'].deleteFileCallback(params.replyAttachId, 'success')
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

      //下载
      downloadCurrentFile(file) {
        const url = this.UploadFile.exportFileUrlPath + 'replyAttachId=' + file.attachmentId + '&creatorId=' + store.getters.user.userId
        window.open(url)
      },

      //预览
      previewCurrentFile(file) {
        this.attachmentPDFSrc = this.UploadFile.viewFileUrlPath + '/' + file.archiveName + '?replyAttachId=' + file.attachmentId + '&creatorId=' + store.getters.user.userId
      },
      cancel() {
        this.form.replyContent = ''
        this.recipientAttachment = []
        this.attachmentIds = []
        this.$emit('change', false)
      },
      closeDialog() {
        this.cancel()
      }
    }
  }
</script>

<style scoped>
  .content /deep/ .el-form-item__label {
    /*padding-right: 20px;*/
  }

  .file /deep/ .el-form-item__label {
    width: 5% !important;
    /*padding-left: 15px !important;*/
    text-align: left;
  }

  .file /deep/ .el-form-item__content {
    margin-left: 50px !important;
  }

  /deep/ .cell .left-box, /deep/ .cell .download_name_box {
    display: none;
  }

  /deep/ .file_name_box {
    max-width: 100% !important;
  }

  /deep/ .cell {
    white-space: pre-wrap !important;
  }

  /deep/ .cell .right-box {
    width: 95%;
  }

  /deep/ .right-box .file_list_box {
    line-height: 5px;
    /*margin-bottom: 10px;*/
    margin-top: 10px;
  }

  /deep/ .attachment-file-box {
    padding: 10px 0 10px 10px;
    min-width: 512px;
  }

  /deep/ .attachment_wrapper {
    /*display: none;*/
  }

  /deep/ .cell .attachment_wrapper {
    display: inline-block;
  }

  /deep/ .cell .right-box {
    text-align: left;
  }

  /deep/ .el-textarea {
    width: 95%;
  }

  /deep/ .el-form-item__error {
    left: 55px;
  }
</style>
