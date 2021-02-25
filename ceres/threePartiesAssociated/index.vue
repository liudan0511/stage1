<template>
  <div class="three-parties-box content-frame-box" ref="calculateFrameRef">
    <eHeader
      :query="query"
      :sup_this="sup_this"
      :pageType="pageType"
      ref="headerRef"
    />
    <el-row class="function_operation_box" ref="functionBarRef">
      <el-button
        class="filter-item"
        size="mini"
        style="padding: 4px 10px;margin-bottom: 10px"
        type="primary"
        icon="el-icon-plus"
        @click="addItem"
      >新增
      </el-button>

    </el-row>
    <!--表格渲染-->
    <el-table
      ref="tableDom"
      :data="data"
      stripe
      border
      size="small"
      highlight-current-row
      style="width: 100%;"
      :key="Math.random()"
      :height="tableHeight"
    >
      <!--      <el-table-column type="selection" width="55"/>-->
      <el-table-column :show-overflow-tooltip="true" type="index" width="50px" label="序号"/>
      <el-table-column
        :show-overflow-tooltip="true"
        prop="applyNumber"
        label="申请编号"
        width="150"
        class="demo-table-expand"
        align="center"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        prop="productName"
        label="产品名称"
        class="demo-table-expand"
        width="300"
        align="left"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        prop="signedType"
        width="100"
        label="签约类型">
        <template slot-scope="scope">
          {{getTypeTextFromTypeId(scope.row.signedType)}}
        </template>
      </el-table-column>

      <el-table-column
        :show-overflow-tooltip="true"
        prop="flowStatus"
        width="100"
        label="状态">
        <template slot-scope="scope">
          {{getStatusTextFromStatusId(scope.row.flowStatus)}}
        </template>
      </el-table-column>

      <el-table-column
        :show-overflow-tooltip="true"
        prop="applyTime"
        width="200"
        label="申请时间"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        prop="rejectReason"
        width="auto"
        label="审核意见"
        align="left"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        label="操作"
        width="200px"
        align="center"
      >
        <template slot-scope="scope">
          <el-button size="mini"
                     type="success"
                     @click="lookOverItem(scope.row.id)">
            查看
          </el-button>
          <el-button v-if="showHandleButtonByStatus(scope.row)"
                     type="danger"
                     size="mini"
                     @click="editItem(scope.row.id)">
            处理
          </el-button>
          <el-button v-if="showDeleteButtonByStatus(scope.row)"
                     type="danger"
                     size="mini"
                     @click="deleteItem(scope.row.id)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      :total="total"
      background
      :current-page="page + 1"
      style="margin-top: 8px;margin-left: 0"
      layout="prev, pager, next,jumper,total, sizes"
      :page-size="size"
      :page-sizes="[10,50,100]"
      @size-change="sizeChange"
      @current-change="pageChange"
    />
    <!--    新增, 修改所用form表单-->
    <three-parties-associated-form
      ref="associatedForm"
      @clearAllFormsData="clearAllFormsData"
      :is-edit="isEditMode"
      :is-exhibit="isExhibitMode"
      :sign-type-for-form="pageType"
      :all-forms="allForms"
      :sup_this="sup_this">
    </three-parties-associated-form>
  </div>
</template>

<script>
import initData from '@/mixins/initData'
import eHeader from './module/header'
import store from '@/store'
import {
  batchDelete,
  threePartiesAssociatedTableUrl, viewApplyThreePartiesAssociateInfo,
} from 'api/ceres/threePartiesAssociated'
import threePartiesAssociatedForm from '@/views/ceres/threePartiesAssociated/module/form'

export default {
  name: 'Cancel',
  mixins: [initData],
  components: { eHeader, threePartiesAssociatedForm },
  data() {
    return {
      sup_this: this,
      isEditMode: false,
      isExhibitMode: false,
      tableHeight: 0,
      // 签约还是解约,默认default 为 sign, 解约通过传入的cancel字段判断
      pageType: '',
      // 资金类型form表单使用变量
      // signTypeForForm: 'sign',
      signTypesOptionsObj: {
        '01': '三方存管',
        '02': '融资融券',
        '03': '银期签约',
        '04': '银衍签约',
        '05': '解除银期签约',
        '06': '解除银衍签约',
      },
      statusOptionsObj: {
        '01': '申请',
        '02': '初审',
        '03': '上传用印文件',
        '04': '复审',
        '05': '三方关联',
        '06': '完成',
        '07': '终止',
      },
      // 所有edit模式或者查看模式下的form表单元素
      allForms: {},
    }
  },
  computed: {},
  created() {
    this.$nextTick(() => {
      // first analyze pagetype
      this.getDataTypeByPath()

      this.init()

      this.calculateTableHeight()

    })
  },
  methods: {
    beforeInit() {
      this.url = threePartiesAssociatedTableUrl
      this.params = {
        page: this.page,
        size: this.size,
        sort: 'applyTime,desc',
      }
      // 将initData.js中定义的query进行本地赋值

      if (this.query.productCode) {
        this.params.productCode = this.query.productCode
      } else {
        this.params.productCode = ''
      }

      if (this.query.signType) {
        this.params.signedType = this.query.signType
      } else {
        //00 代表 签约; 10 代表解约
        if (this.pageType === 'sign') {
          // 签约列表标记
          this.params.signedType = '00'
        } else {
          // 解约列表标记
          this.params.signedType = '10'
        }
      }

      if (this.query.startDate) {
        this.params.applyTimeStart = this.query.startDate
      }

      if (this.query.flowStatus) {
        this.params.flowStatus = this.query.flowStatus
      }

      if (this.query.endDate) {
        this.params.applyTimeEnd = this.query.endDate
      }

      if (store.getters.user.userId) {
        this.params.creatorId = store.getters.user.userId
      }

      return true
    },
    calculateTableHeight() {
      let calculatedFrame = this.$refs.calculateFrameRef
      let calculateHeader = this.$refs.headerRef
      if (this.total > 10) {
        this.tableHeight = `${calculatedFrame.clientHeight -
        calculateHeader.$el.clientHeight - 52}`
      } else {
        this.tableHeight = 'auto'
      }
    },
    getDataTypeByPath() {
      let path = this.$route.path
      if (path.indexOf('sign') > -1) {
        this.pageType = 'sign'
      } else {
        this.pageType = 'cancel'
      }
    },
    deleteItem(id) {
      this.confirmDeleteEntity(id)
    },
    showHandleButtonByStatus(row) {
      if (!row.flowStatus || row.flowStatus === '01' || row.flowStatus === '03') {
        return true
      } else {
        return false
      }
    },
    showDeleteButtonByStatus(row) {
      if (!row.flowStatus || (row.flowStatus === '01' && !row.processInstanceId)) {
        return true
      } else {
        return false
      }
    },
    batchDelete() {
      if (this.multipleSelection.length === 0) {
        this.$alert('请先勾选数据', '提示', {
          confirmButtonText: '确定',
          type: 'info',
        })
        return
      }
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
      .then(() => {
        let ids = ''
        if (this.multipleSelection.length > 0) {
          for (let i = 0; i < this.multipleSelection.length; i++) {
            ids = ids + this.multipleSelection[i].id + ','
          }
          ids = ids.substring(0, ids.length - 1)
        }
        // console.log("批量删除" + ids);
        console.log(ids)
      })
      .catch((err) => {
        console.log(err)
      })
    },
    confirmDeleteEntity(id) {
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
      .then(() => {
        batchDelete(id)
        .then(res => {
          if (res.status === 'success') {
            this.toast('删除成功')
            this.init()
          } else {
            this.toast(res.msg, 'error')
          }
        })
        .catch(err => {
          console.log(err.response.data.message)
          this.toast(err.response.data)
        })
      })
      .catch((err) => {
        console.log(err)
      })
    },
    addItem() {
      this.isEditMode = false
      this.isExhibitMode = false

      //00 代表 签约; 10 代表解约 pagetype

      // 传递不同的条件, 签约还是解除, 新增还是编辑
      this.$refs.associatedForm.associateDialog = true

      // 直接调用子组件事件
      this.$refs.associatedForm.initEvent()
    },
    lookOverItem(id) {
      this.isEditMode = false
      this.isExhibitMode = true
      this.queryFormDataById(id)
    },
    editItem(id) {
      this.isEditMode = true
      this.isExhibitMode = false
      this.queryFormDataById(id)
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
    getTypeTextFromTypeId(type) {
      return this.signTypesOptionsObj[type]
    },
    getStatusTextFromStatusId(status) {
      return this.statusOptionsObj[status]
    },
    queryFormDataById(id) {
      let params = {
        id: id,
      }
      viewApplyThreePartiesAssociateInfo(params).then((res) => {
        if (res.status === 'success') {
          let data = res.data

          if (data) {
            // 所有表单数据
            this.allForms = data

            this.$nextTick(() => {
              // 显隐dialog
              this.$refs.associatedForm.associateDialog = true
              // 直接调用子组件事件
              this.$refs.associatedForm.initEvent()
            })
          } else {
            this.toast(res.msg, 'warning')
          }
        } else {
          this.toast(res.msg, 'warning')
        }
      }).catch((error) => {
        console.error(error)
      })
    },
    clearAllFormsData() {
      this.resetMode()
      this.resetForm()
    },
    resetMode() {
      this.isEditMode = false
      this.isExhibitMode = false
    },
    // 避免表单resetFileds重置初始值引起的不能清空问题
    resetForm() {
      this.allForms = {
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
          // 是否展示联系人
          visibleExpress: '',

          recipientInfoForm: {
            //快递单号
            expressNumber: '',
            //收件人
            recipients: '',
            //收件人附件列表
            attachment: [],
          },
        },
      }
    },
  },
}

</script>

<style scoped lang="scss">

  .content-frame-box {
    height: calc(100vh - 285px);
    min-height: 580px;
  }

  .three-parties-box {
    background: #fff;
    border-radius: 3px;
    padding: 20px;
    width: 100%;
  }

  .function_operation_box {
    width: 100%;
    padding-bottom: 10px;
  }

  .el-col-md-10 {
    width: 100%;
  }

  /deep/ .el-table .el-button{
    padding: 0 12px;
  }

  /deep/ .el-table .el-table__header th.is-left {
    text-align: center;
  }

  /deep/ .el-table .el-table__header th {
    font-size: 16px;
    height: 37px;
  }

  /deep/ .el-table .el-table__body td {
    font-size: 14px;
    height: 47px;
  }

  /*操作列处理按钮样式*/
  /deep/ .el-button--danger {
    background: white;
    color: red;
  }

  /*分页样式*/
  /*修改背景色*/
  /deep/ .el-pagination.is-background .el-pager li,
  /deep/ .el-pagination.is-background .btn-prev,
  /deep/ .el-pagination.is-background .btn-next {
    background-color: #eee;
  }

  /deep/ .el-pagination__jump .el-input .el-input__inner {
    width: 50px;
    height: 32px;
    line-height: 32px;
  }

  /deep/ .el-pagination {
    padding-left: 0;
  }

  /deep/ .el-pagination__total {
    margin-left: 15px;
  }
  /deep/ .cell .el-button--mini{
    height: 28px;
  }
</style>


