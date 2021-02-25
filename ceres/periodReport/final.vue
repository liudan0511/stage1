<template>
  <div>
    <div class="el-button-spli"></div>
    <el-form :inline="true" size="mini" @submit.native.prevent>
      <el-form-item label="产品名称:">
        <!--        filterable代表可输入-->
        <el-select
          filterable
          collapse-tags
          multiple
          v-model="fundCodes"
          placeholder="请输入产品代码或产品名称"
          style="width: 340px;height: 32px">
          <el-option
            v-for="item in options"
            :key="item.code"
            :label="item.name"
            :value="item.code"
            style="width:340px">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="年份：" prop="year">
        <el-date-picker
          v-model="years"
          type="year"
          popper-class="yearCls"
          style="width: 220px"
          placeholder="选择年">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="报告类型：">
        <el-select
          multiple
          collapse-tags
          v-model="reportTypes"
          placeholder="请选择"
          style="width: 180px;"
          @change="onSelectedDrug($event)">
          <el-option
            v-for="item in types"
            :key="item.code"
            :label="item.name"
            :value="item.code">
          </el-option>
        </el-select>
        <el-select multiple collapse-tags v-model="months" placeholder="请选择" style="width: 130px;" v-if="monFlag">
          <el-option
            v-for="item in monOption"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-search"
          @click="toQuery">搜索
        </el-button>
        <el-button
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-refresh-right"
          @click="reset">重置
        </el-button>
      </el-form-item>
    </el-form>
    <el-button
      class="filter-item"
      size="mini"
      type="primary"
      icon="el-icon-download"
      style="margin-bottom: 5px"
      @click="batchDownLoad">批量下载
    </el-button>
    <!--      表格渲染-->
    <el-table
      :data="tableData"
      border
      :stripe="true"
      ref="tableRef"
      @selection-change="handleSelectionChange"
      style="width: 100%">
      <el-table-column
        type="selection"
        width="50">
      </el-table-column>
      <el-table-column
        prop="fundCode"
        label="产品代码"
        width="100"
      >
      </el-table-column>
      <el-table-column
        prop="fundName"
        label="产品名称"
        min-width="200"
        :show-overflow-tooltip="true"
      >
      </el-table-column>

      <el-table-column
        prop="reportTypeName"
        label="报告类型"
        width="150"
      >
      </el-table-column>
      <el-table-column
        prop="reportPeroid"
        label="报告期"
        width="200"
      >
      </el-table-column>
      <el-table-column label="管理人编辑" class="download" width="180">
        <template slot-scope="scope">
          <span v-if="scope.row.managerReportStatus==='0'"><img
            src="@/assets/periodReport/0.png"
            alt=""
            @click="turnManager">{{scope.row.managerReportStatus}}%</span>
          <span v-if="scope.row.managerReportStatus==='50'">
            <img
              src="@/assets/periodReport/50.png"
              alt=""
              @click="turnManager">{{scope.row.managerReportStatus}}%
          </span>
          <span v-if="scope.row.managerReportStatus==='100'">
            <img
              src="@/assets/periodReport/100.png"
              alt=""
              @click="turnManager">{{scope.row.managerReportStatus}}%
          </span>
          <span v-if="!scope.row.managerReportStatus">
            <img
              src=""
              alt=""
              @click="turnManager">
          </span>

        </template>
      </el-table-column>
      <el-table-column label="财务数据核对" class="download" width="180">
        <template slot-scope="scope">
          <span v-if="scope.row.disclosureProgressStatus==='0'"><img
            src="@/assets/periodReport/0.png"
            alt=""
            @click="turnManager">{{scope.row.disclosureProgressStatus}}%</span>
          <span v-if="scope.row.disclosureProgressStatus==='50'">
            <img
              src="@/assets/periodReport/50.png"
              alt=""
              @click="turnManager">{{scope.row.disclosureProgressStatus}}%
          </span>
          <span v-if="scope.row.disclosureProgressStatus==='100'">
            <img
              src="@/assets/periodReport/100.png"
              alt=""
              @click="turnManager">{{scope.row.disclosureProgressStatus}}%
          </span>
          <span v-if="!scope.row.disclosureProgressStatus">
            <img
              src=""
              alt=""
              @click="turnManager">
          </span>
        </template>
      </el-table-column>
      <el-table-column label="报告下载" class="download" width="180">
        <template slot-scope="scope" width="200">
          <el-button
            class="default"
            v-if="scope.row.wordFileId!==''"
            @click="downLoad(scope.row.wordFileId)"
            icon="el-icon-w"></el-button>
          <el-button
            v-if="scope.row.excelFileId!==''"
            @click="downLoad(scope.row.excelFileId)"
            icon="el-icon-x"></el-button>
          <el-button
            v-if="scope.row.wordFileId!=='' && scope.row.excelFileId!==''"
            @click="downLoadPdf(scope.row.id)"
            icon="el-icon-p"></el-button>
          <el-button
            v-if="scope.row.xbFileId!==''"
            @click="downLoad(scope.row.xbFileId)"
            icon="el-icon-xb"></el-button>
        </template>
      </el-table-column>
      <el-table-column label="操作" class="btn" width="150">
        <template slot-scope="scope">
          <el-tooltip class="item" effect="dark" content="预览" placement="top-start">
            <el-button
              v-if="scope.row.wordFileId!==''"
              @click="preview(scope.row.id)"
              icon="el-icon-preview">
            </el-button>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="申请用印版" placement="top-start">
            <el-button
              v-if="scope.row.wordFileId!==''"
              @click="apply(scope.row.id)"
              icon="el-icon-apply"></el-button>
          </el-tooltip>


        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      :total="total"
      :current-page="page"
      style="margin-top: 8px;"
      background
      layout="prev, pager, next,jumper,total,sizes"
      :page-size="size"
      :page-sizes="[10, 50, 100]"
      @size-change="sizeChange"
      @current-change="pageChange"
    />
  </div>
</template>

<script>
  import initData from '@/mixins/initData'
  import store from "@/store/index"
  import { getProTable, getProInfo, getTypes, periodUrl } from '@/api/ceres/periodReport'


  export default {
    name: "Final",
    mixins: [initData],
    props: {
      sup_this: {
        type: Object,
        default: null
      }
    },
    data() {
      return {
        total: 0,
        fundCodes: '',
        types: [],
        years: '',
        reportTypes: '',
        months: '',
        options: [],
        yearOption: [],
        //是否展示月份下拉框
        monFlag: false,
        tableData: [],
        monOption: [{
          value: '01',
          label: '1月'
        }, {
          value: '02',
          label: '2月'
        }, {
          value: '03',
          label: '3月'
        }, {
          value: '04',
          label: '4月'
        }, {
          value: '05',
          label: '5月'
        }, {
          value: '06',
          label: '6月'
        }, {
          value: '07',
          label: '7月'
        }, {
          value: '08',
          label: '8月'
        }, {
          value: '09',
          label: '9月'
        }, {
          value: '10',
          label: '10月'
        }, {
          value: '11',
          label: '11月'
        }, {
          value: '12',
          label: '12月'
        }],
        wordFileId: '',
        excelFileId: '',
        pdfFileId: '',
        xbFileId: '',
        signPdfFileId: '',
        //批量下载id
        ids: [],
      }
    },
    created() {
      this.getProInfo()
      this.getType()
      this.initYearBox()
      this.toQuery()
    },
    methods: {

      //分页
      pageChange(e) {
        this.page = e
        this.toQuery()
        this.page = 0
      },
      sizeChange(e) {
        this.page = 0
        this.size = e
        this.toQuery()
      },
      //获取年
      initYearBox() {
        let thisYear = new Date().getUTCFullYear();
        let lastYear = thisYear - 1;
        let startYear = thisYear - 2;
        let year = [];
        year.push(startYear, lastYear, thisYear);

        for (var i = 0; i < 3; i++) {
          let params = {
            label: year[i] + '年',
            value: year[i]
          }
          this.yearOption.push(params);
        }
      },

      //获取所有产品名称
      getProInfo() {
        getProInfo(
          store.getters.user.userId
        ).then(res => {
          if (res.status === 'success') {
            this.options = res.data
          } else {
            this.$notify({
              title: res.msg,
              type: "error",
              duration: 2500
            });
          }

        })
      },

      //获取报告类型
      getType() {
        getTypes().then(res => {
          if (res.status === 'success') {
            this.types = res.data
          } else {
            this.$notify({
              title: res.msg,
              type: "error",
              duration: 2500
            });
          }

        })
      },
      //  点击报告类型是否显示月份
      onSelectedDrug(event) {
        if (event.includes("0201")) {
          this.monFlag = true
        } else {
          this.monFlag = false
        }
      },

      //搜索
      toQuery() {
        let fundCodes;
        if (this.fundCodes) {
          fundCodes = this.fundCodes.join()
        } else {
          fundCodes = ''
        }
        let years;
        if (this.years) {
          years = this.years.toString().substr(11, 4)
        } else {
          years = ''
        }
        let reportTypes;
        if (this.reportTypes) {
          reportTypes = this.reportTypes.join()
        } else {
          reportTypes = ''
        }
        let months;
        if (this.monFlag) {
          months = this.months.join()
        } else {
          months = ''
        }
        let params = {
          fundCodes: fundCodes,
          years: years,
          reportTypes: reportTypes,
          months: months,
          page: this.page,
          size: this.size,
          userId: store.getters.user.userId,
        }
        getProTable(params).then(res => {
          if (res.status === 'success') {
            this.tableData = res.data
            this.total = res.total
            if (this.tableData) {
              this.total = res.total
              this.tableData.map(item => {
                if (item.managerWordFileId || item.releaseWordFileId) {
                  item.wordFileId = item.managerWordFileId || item.releaseWordFileId
                } else {
                  item.wordFileId = ''
                }

                if (item.managerExcelFileId || item.releaseExcelFileId) {
                  item.excelFileId = item.managerExcelFileId || item.releaseExcelFileId
                } else {
                  item.excelFileId = ''
                }

                if (item.managerPdfFileId || item.releasePdfFileId) {
                  item.pdfFileId = item.managerPdfFileId || item.releasePdfFileId
                } else {
                  item.pdfFileId = ''
                }

                if (item.managerXbFileId || item.releaseXbFileId) {
                  item.xbFileId = item.managerXbFileId || item.releaseXbFileId
                } else {
                  item.xbFileId = ''
                }
              })
            }
          } else {
            this.$notify({
              title: res.msg,
              type: "error",
              duration: 2500
            });
          }
        })
      },
      //点击报告进度跳转到管理人报告
      turnManager() {
        this.sup_this.activeName = 'manager'
      },
      //重置
      reset() {
        this.page = 0
        this.size = 10
        this.fundCodes = ''
        this.years = ''
        this.reportTypes = ''
        this.months = ''
        this.monFlag = false
        this.toQuery()
      },
      //获取ids
      handleSelectionChange(val) {
        val.forEach(item => {
          if (this.ids.indexOf(item.id) >= 0) {
            this.ids = this.ids.splice(this.ids.indexOf(item.id), 1)
          } else {
            this.ids = this.ids.concat(item.id)
          }
        })
      },

      //批量下载---传每一行的id
      batchDownLoad() {
        let idsArr = this.ids.join(',');
        if (idsArr) {
          let url = periodUrl + '/bulkdownload/' + idsArr + '/' + store.getters.user.userId
          window.open(url)
        } else {
          if (!idsArr) {
            this.$notify({
              title: '无可下载内容',
              type: 'error',
              duration: 2500
            })
          }
        }
      },
      //下载word,excel,xb
      downLoad(id) {
        let url = periodUrl + '/download/' + id + '/' + store.getters.user.userId
        window.open(url)
      },
      //下载pdf
      downLoadPdf(id) {
        console.log(id)
        if (id) {
          let url = periodUrl + '/downpdf/' + id + '/' + store.getters.user.userId
          window.open(url)
        }
      },
      //预览
      preview(id) {
        let url = periodUrl + '/viewpdf/' + id + '/' + store.getters.user.userId
        window.open(url)
      },
      //申请应用版
      apply(id) {
        let url = periodUrl + '/applysign/' + id + '/' + store.getters.user.userId
        window.open(url)
      },

      // 批量下载---传文件id
      notEmpty(p) {
        if (Object.prototype.toString.call(p) === '[object Array]') {
          if (p.length > 0) return true;
        } else if (Object.prototype.toString.call(p) === '[object Object]') {
          if (JSON.stringify(p) !== '{}') return true;
        } else if (p != "" && p !== undefined && p != null && p !== "undefined" && p !== "null") return true;
        else return false;
      },
      // madeIDS(val) {
      //   let IDS = [];
      //   if (this.notEmpty(val)) {
      //     val.map(item => {
      //       let arr = ['wordFileId', 'excelFileId', 'pdfFileId', 'xbFileId']
      //       arr.forEach(v => {
      //         let it = item[v];
      //         if (this.notEmpty(it) && IDS.indexOf(it) === -1) {
      //           IDS.push(it);
      //         }
      //       })
      //     })
      //   }
      //   return IDS;
      // },
      //批量下载---传文件id
      // batchDownLoad() {
      //   let IDS = this.$refs.tableRef.selection;
      //   // console.log(idsArr)
      //   // debugger
      //   let idsArr = this.madeIDS(IDS).join(',');
      //
      //   if (idsArr) {
      //     let url = periodUrl + '/downloadzip/' + idsArr + '/' + store.getters.user.userId
      //     window.open(url)
      //   } else {
      //     if (!idsArr) {
      //       this.$notify({
      //         title: '无可下载内容',
      //         type: 'error',
      //         duration: 2500
      //       })
      //     }
      //   }
      // },

    }
  }
</script>
<style>

</style>
<style scoped>
  p {
    padding: 0;
  }

  .el-button-spli {
    width: 3px;
    background-color: red;
    height: 24px;
    color: red;
    margin-right: 10px;
    float: left;
  }

  /deep/ .el-input__inner {
    height: 32px !important;
  }

  /*修改下拉框内容展示*/
  /*/deep/ .el-select__tags{*/
  /*  width: 650px;*/
  /*  max-width: 370px;*/
  /*}*/

  /*/deep/ .el-select__tags {*/
  /*  display: -webkit-box;*/
  /*  -webkit-box-orient: vertical;*/
  /*  -webkit-line-clamp: 1;*/
  /*  overflow: hidden;*/
  /*  max-height: 50px;*/
  /*}*/
  /deep/ .el-form {
    height: 35px;
    min-width: 1105px;
  }

  /deep/ .el-tag .el-tag--info {
    position: absolute;
    left: 0;
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
    /*max-width: 200px;*/
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  /deep/ .el-select__input {
    /*width: 100%;*/
    min-width: 100px !important;
    position: relative;
    /*left: -75px;*/
  }

  .btn .el-button {
    width: 22px;
    height: 24px;
  }

  .progress .cell .el-button {
    width: 100%;
  }

  .cell img {
    /*width: 100%;*/
    /*height: 20px;*/
    margin-top: 5px;
  }

  .cell span img {
    vertical-align: top;
    margin-right: 5px;
  }

  .cell .el-button {
    width: 22px;
  }

  body .el-table .el-button {
    padding: 0 12px;
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

  /deep/ .el-icon-w {
    background: url('../../../assets/periodReport/W.png') center no-repeat;
    background-size: cover;
    margin-left: -11px;
  }

  /deep/ .el-icon-x {
    background: url('../../../assets/periodReport/X.png') center no-repeat;
    background-size: cover;
    margin-left: -11px;
  }

  /deep/ .el-icon-p {
    background: url('../../../assets/periodReport/P.png') center no-repeat;
    background-size: cover;
    margin-left: -11px;
  }

  /deep/ .el-icon-xb {
    background: url('../../../assets/periodReport/XB.png') center no-repeat;
    background-size: cover;
    margin-left: -11px;
  }

  /deep/ .el-icon-preview {
    background: url('../../../assets/periodReport/preview.png') center no-repeat;
    background-size: cover;
    margin-left: -11px;
  }

  /deep/ .el-icon-apply {
    background: url('../../../assets/periodReport/apply.png') center no-repeat;
    background-size: cover;
    margin-left: -11px;
  }

  /deep/ .el-icon-w:before, .el-icon-x:before, .el-icon-p:before, .el-icon-xb:before {
    content: "替";
    font-size: 16px;
    visibility: hidden;
  }

  /deep/ .el-icon-x:before {
    content: "替";
    font-size: 16px;
    visibility: hidden;
  }

  /deep/ .el-icon-p:before {
    content: "替";
    font-size: 16px;
    visibility: hidden;
  }

  /deep/ .el-icon-xb:before {
    content: "替";
    font-size: 16px;
    visibility: hidden;
  }

  /deep/ .el-icon-preview:before {
    content: "替";
    font-size: 16px;
    visibility: hidden;
  }

  /deep/ .el-icon-apply:before {
    content: "替";
    font-size: 16px;
    visibility: hidden;
  }

  /deep/ .el-icon-w, /deep/ .el-icon-x, /deep/ .el-icon-p, /deep/ .el-icon-xb, /deep/ .el-icon-preview, /deep/ .el-icon-apply {
    width: 22px;
    height: 24px;
  }

</style>
