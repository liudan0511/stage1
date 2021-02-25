<template>
  <div class="index">
    <eHeader
      ref="header"
      :sup_this="sup_this"
      :query="query"
    />
    <el-button
      class="filter-item"
      size="mini"
      type="primary"
      style="margin-bottom: 20px"
      :disabled="statusFlag"
      @click="batchEditStatus">置为已读
    </el-button>
    <!--      表格渲染-->
    <el-table
      :data="data"
      ref="tableRef"
      border
      stripe
      @selection-change="handleSelectionChange"
      size="small"
      style="width: 100%">
      <el-table-column
        type="selection"
        width="80"
      >
      </el-table-column>
      <el-table-column
        prop="productCode"
        label="产品代码"
        width="100"
      >
      </el-table-column>
      <el-table-column
        prop="productName"
        label="产品名称"
        :show-overflow-tooltip="true"
      >
      </el-table-column>
      <el-table-column
        prop="monitorDate"
        label="监控日期"
        :show-overflow-tooltip="true"
        width="150"
      >
        <template slot-scope="scope">
          {{timestampToTime(scope.row.monitorDate)}}
        </template>
      </el-table-column>
      <el-table-column
        prop="violationContent"
        label="违规事项"
        :show-overflow-tooltip="true"
      >
      </el-table-column>
      <el-table-column
        prop="realValue"
        label="实际值"
        width="150"
      >
      </el-table-column>
      <el-table-column
        prop="exceedingDays"
        label="超标天数"
        width="100"
      >
      </el-table-column>
      <el-table-column
        prop="sendLetterDate"
        label="发函日期"
        width="150"
      >
        <template slot-scope="scope">
          {{timestampToTime(scope.row.sendLetterDate)}}
        </template>
      </el-table-column>
      <el-table-column
        prop="readStatus"
        label="阅读状态"
        width="100"
      >
        <template slot-scope="scope">
          {{scope.row.readStatus ==='0'?'未读':'已读'}}
        </template>
      </el-table-column>
      <el-table-column
        prop="readTime"
        label="阅读时间"
        width="150"
      >
        <template slot-scope="scope">
          {{scope.row.readTime?timestampToTime(scope.row.readTime):''}}
        </template>
      </el-table-column>
      <el-table-column label="操作" class="btn" width="200px">
        <template slot-scope="scope">
          <el-button
            slot="reference"
            type="danger"
            size="mini"
            @click="preview(scope.$index,scope.row)"
          >查看
          </el-button>
          <el-button
            slot="reference"
            type="danger"
            size="mini"
            @click="downLoad(scope.row.id)"
          >下载
          </el-button>
          <el-button
            slot="reference"
            type="danger"
            size="mini"
            @click="reply(scope.$index,scope.row)"
          >回复
          </el-button>
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
    <eForm
      ref="formAlert"
      :dialogFormVisible="dialogFormVisible"
      :formAlertData="formAlertData"
      :title="title"
      @change="change"
      :sup_this="sup_this"
    />
    <div class="preview-file-box">
      <el-dialog
        v-el-drag-dialog
        @close="closeDialog()"
        :visible.sync="pdfShow"
        :append-to-body="true"
        custom-class="dialogStyle"
        :title="archiveName">
        <iframe :src="pdfSrc" width="100%" height="650px" frameborder="0"/>
      </el-dialog>
    </div>
  </div>
</template>

<script>
  import eHeader from './module/header'
  import eForm from './module/form'
  import initData from '@/mixins/initData'
  import store from "@/store/index"
  import {baseUrl, editStatus, viewPdfUrl, downloadPdfUrl} from '@/api/ceres/riskWarning'

  export default {
    name: "index",
    components: {eHeader, eForm},
    mixins: [initData],
    data() {
      return {
        // total: 0,
        tableData: [],
        sup_this: this,
        title: '回复',
        //置为已读id
        ids: [],
        statusFlag: false,
        //显示弹框
        dialogFormVisible: false,
        formAlertData: {},
        // PDF名字
        archiveName: '',
        // 文件预览部分
        pdfShow: false,
        pdfSrc: '',
      }
    },
    mounted() {
      this.$nextTick(() => {
        this.init()
      })
    },
    methods: {
      change(flag) {
        this.dialogFormVisible = flag
      },

      timestampToTime(timestamp) {

        let date = new Date(timestamp);//时间戳为10位需*1000，时间戳为13位的话不需乘1000

        let Y = date.getFullYear() + '-';

        let M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-';

        let D = (date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + ' ';

        // let h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':';
        //
        // let m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) + ':';
        //
        // let s = (date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds());

        // return Y + M + D + h + m + s;
        return Y + M + D

      },


      beforeInit() {
        this.url = baseUrl
        this.params = {
          page: this.page,
          size: this.size,
        }

        if (this.query.productCode) {
          this.params.productCode = this.query.productCode
        }

        if (this.query.readStatus) {
          this.params.readStatus = this.query.readStatus
        }

        if (this.query.monitorDateStart) {
          this.params.monitorDateStart = this.query.monitorDateStart
        }
        if (this.query.monitorDateEnd) {
          this.params.monitorDateEnd = this.query.monitorDateEnd
        }

        if (store.getters.user.userId) {
          this.params.creatorId = store.getters.user.userId
        }
        return true

      },


      //置为已读
      handleSelectionChange(val) {
        if (JSON.stringify(val) === '[]') {
          this.statusFlag = false
        }
        let status = []
        val.map(item => {
          status.push(item.readStatus)
          if (status.includes('1')) {
            this.statusFlag = true
            this.$notify({
              title: '请选择状态为未读的产品',
              type: 'warning',
              duration: 2500
            })
          } else {
            this.statusFlag = false
          }
          if (this.ids.indexOf(item.id) >= 0) {
            this.ids = this.ids.splice(this.ids.indexOf(item.id), 1)
          } else {
            this.ids = this.ids.concat(item.id)
          }
        })

      },

      batchEditStatus() {
        let ids = this.ids.join()
        if (ids) {
          let params = {
            ids: ids,
            creatorId: store.getters.user.userId,
          }
          editStatus(params).then(res => {
            if (res.status === 'success') {
              this.page = 0
              this.size = 10
              this.init()
            } else {
              this.$notify({
                title: res.msg,
                type: 'warning',
                duration: 2500
              })
            }
          }).catch(err => {
            this.$notify({
              title: '为未读状态才可修改',
              type: 'warning',
              duration: 2500
            })
          })
        } else {
          this.$notify({
            title: '请选择',
            type: 'error',
            duration: 2500
          })
        }
      },
      //查看
      preview(index, row) {
        this.pdfShow = true
        this.archiveName = row.attachmentName
        this.pdfSrc = viewPdfUrl + '/' + row.attachmentName + '?id=' + row.id + '&creatorId=' + store.getters.user.userId
        this.page = 0
        this.size = 10
        this.init()
      },

      // 默认列表下载
      downLoad(id) {
        let url = downloadPdfUrl + 'id=' + id + '&creatorId=' + store.getters.user.userId
        window.open(url)
      },

      //  回复
      reply(index, row) {
        this.dialogFormVisible = true
        this.formAlertData = row
        this.$refs['formAlert'].publicId = row.id
        this.$refs['formAlert'].init()
      },
      //关闭弹框
      closeDialog() {
        this.archiveName = ''
      },
    }
  }
</script>

<style scoped>
  .index {
    padding: 20px;
    width: 98%;
  }

  /deep/ .el-table .el-button {
    padding: 0 12px;
  }

  .el-button-spli {
    width: 3px;
    background-color: red;
    height: 24px;
    color: red;
    margin-right: 2px;
    float: left;
  }

  /*输入框中样式*/
  /deep/ .el-select .el-tag {
    max-width: 150px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  /deep/ .el-select__tags {
    max-width: 250px;
  }

  /deep/ .el-select__tags-text {
    display: block;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  /deep/ .el-input--mini .el-input__icon {
    line-height: 33px;
  }

  /*表格样式*/
  /deep/ .el-table .el-table__header th {
    font-size: 16px;
    height: 37px;
  }

  /deep/ .el-table .el-table__body td {
    font-size: 14px;
    height: 47px;
  }

  /*分页样式*/

  /*修改背景色*/
  /deep/ .el-pagination.is-background .el-pager li, /deep/ .el-pagination.is-background .btn-prev, /deep/ .el-pagination.is-background .btn-next {
    background-color: #eee;
  }

  /deep/ .el-pagination__jump .el-input .el-input__inner {
    width: 50px;
    height: 30px;
  }

  /deep/ .el-pagination__total {
    margin-left: 15px;
  }
</style>
