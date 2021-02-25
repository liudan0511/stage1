<template>
  <div class="index">
    <el-form :inline="true" size="mini" @submit.native.prevent>
      <div class="el-button-spli"></div>
      <span class="searchline">&nbsp;</span>
      <el-form-item label="产品名称:">
        <!--        filterable代表可输入-->
        <el-select filterable v-model="fundCode" placeholder="请选择" style="width: 400px;">
          <el-option
            v-for="item in options"
            :key="item.fundCode"
            :label="item.showFundName"
            :value="item.fundCode"
            style="width: 400px">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="报表类型:">
        <el-select multiple collapse-tags v-model="reportName" placeholder="请选择" style="width: 185px;">
          <el-option
            v-for="item in typeOption"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="报表频率:">
        <el-select multiple collapse-tags v-model="reportPeriod" @change="onSelectedDrug($event)" placeholder="请选择"
                   style="width: 120px;">
          <el-option
            v-for="item in periodOption"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="报告期：">
        <el-select multiple collapse-tags v-model="reportYear" placeholder="请选择" style="width: 125px;">
          <el-option
            v-for="item in yearOption"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="月份：" v-if="monFlag">
        <el-select multiple collapse-tags v-model="reportMonth " placeholder="请选择" style="width: 110px;">
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
        <el-button class="filter-item"
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
      style="margin-bottom: 20px"
      @click="batchDownLoad">批量下载
    </el-button>
    <!--      表格渲染-->
    <el-table
      :data="tableData"
      ref="tableRef"
      border
      stripe
      @selection-change="handleSelectionChange"
      size="small"
      style="width: 100%">
      <el-table-column
        type="selection"
        width="100"
      >
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
        width="300"
        :show-overflow-tooltip="true"
      >
      </el-table-column>
      <el-table-column
        prop="fileName"
        label="文件名称"
        :show-overflow-tooltip="true"
      >
      </el-table-column>
      <el-table-column
        prop="reportPeriod"
        label="报告频率"
        width="100"
      >
      </el-table-column>
      <el-table-column
        prop="reportDate"
        label="报告期"
        width="120">
      </el-table-column>
      <el-table-column label="操作" class="btn" width="100">
        <template slot-scope="scope">
          <el-button
            slot="reference"
            type="danger"
            size="mini"
            @click="downLoad(scope.row.fileId)"
          >下载
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      :total="total"
      :current-page="page"
      style="margin-top: 8px;"
      layout="prev, pager, next,jumper,total, sizes"
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
  import {getFunds, toQuery, financialUrl} from '@/api/ceres/finacialReport'

  export default {
    name: "Final",
    mixins: [initData],
    data() {
      return {
        //是否展示月份
        monFlag: false,
        fundCode: '',
        reportName: '',
        reportPeriod: '',
        reportYear: '',
        reportMonth: '',
        total: 0,
        page: 1,
        years: '',
        reportTypes: '',
        months: '',
        options: [],
        yearOption: [],
        tableData: [],
        typeOption: [
          {
            value: '01',
            label: '资产负债表'
          },
          {
            value: '02',
            label: '损益表'
          },
          {
            value: '03',
            label: '所有者权益变动表'
          }
        ],
        periodOption: [
          {
            value: '01',
            label: '月报'
          },
          {
            value: '02',
            label: '年报'
          }
        ],
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
        //批量下载id
        ids: [],
      }
    },
    created() {
      this.getProInfo()
      this.initYearBox()
      // this.reportPeriod = this.periodOption[0].value
      // this.reportYear = this.yearOption[0].value
    },
    methods: {
      //分页
      pageChange(e) {
        this.page = e
        this.toQuery()
      },
      sizeChange(e) {
        this.page = 1
        this.size = e
        this.toQuery()
      },
      //获取年
      initYearBox() {
        let thisYear = new Date().getUTCFullYear();
        let lastYear = thisYear - 1;
        let startYear = thisYear - 2;
        let year = [];
        year.push(thisYear, lastYear, startYear);

        for (var i = 0; i < 3; i++) {
          let params = {
            label: year[i] + '年',
            value: year[i]
          }
          this.yearOption.push(params);
        }
      },

      //  点击报告类型是否显示月份
      onSelectedDrug(event) {
        if (!event.includes('02') && !event.includes('01')) {
          this.monFlag = false
          this.reportMonth = ''
        } else if (event.includes('02')) {
          this.monFlag = false
          this.reportMonth = ''
        } else {
          this.monFlag = true
        }
      },

      //获取所有产品名称
      getProInfo() {
        getFunds(
          // 'CS0TG131'
          store.getters.user.userId
        ).then(res => {
          if (res.status === 'success') {
            if (res.data) {
              this.options = res.data
              this.fundCode = this.options[0].fundCode
              this.toQuery()
            }
          } else {
            this.$notify({
              title: res.msg,
              type: 'error',
              duration: 2500
            })
          }
        })
      },

      //搜索展示
      toQuery() {
        let months
        if (this.reportMonth) {
          months = this.reportMonth.join()
        } else {
          months = ''
        }
        let reportName
        if (this.reportName) {
          reportName = this.reportName.join()
        } else {
          reportName = ''
        }
        let reportYear
        if (this.reportYear) {
          reportYear = this.reportYear.join()
        } else {
          reportYear = ''
        }
        let reportPeriod
        if (this.reportPeriod) {
          reportPeriod = this.reportPeriod.join()
        } else {
          reportPeriod = ''
        }

        let param = {
          page: this.page,
          size: this.size,
          fundCode: this.fundCode,
          reportName: reportName,
          reportPeriod: reportPeriod,
          reportYear: reportYear,
          reportMonth: months
        }
        toQuery(param).then(res => {
          if (res.status === 'success') {
            this.tableData = res.data.result
            this.total = res.data.total
            if (this.tableData) {
              this.tableData.forEach(item => {
                if (item.reportPeriod === '01') {
                  item.reportPeriod = '月报'
                } else {
                  item.reportPeriod = '年报'
                }
              })
            }
          } else {
            this.$notify({
              title: res.msg,
              type: 'error',
              duration: 2500
            })
          }
        })
      },

      //重置
      reset() {
        this.fundCode = ''
        this.reportName = ''
        this.reportPeriod = ''
        this.reportYear = ''
        this.reportMonth = ''
        // this.monFlag = false
        this.fundCode = this.options[0].fundCode
        this.page = 1
        this.size = 10
        this.toQuery()
      },

      //批量下载
      handleSelectionChange(val) {
        val.map(item => {
          if (this.ids.indexOf(item.fileId) >= 0) {
            this.ids = this.ids.splice(this.ids.indexOf(item.fileId), 1)
          } else {
            this.ids = this.ids.concat(item.fileId)
          }
        })
      },
      //
      batchDownLoad() {
        let ids = this.ids.join()
        if (ids) {
          let url = financialUrl + 'userId=' + store.getters.user.userId + '&fileId=' + ids + '&typeCode=03'
          window.open(url)
        } else {
          this.$notify({
            title: '请选择',
            type: 'error',
            duration: 2500
          })
        }

      },

      //单个下载
      downLoad(id) {
        let url = financialUrl + 'userId=' + store.getters.user.userId + '&fileId=' + id + '&typeCode=03'
        window.open(url)
      }

    }
  }
</script>

<style scoped>
  .index {
    padding: 20px;
    width: 98%;
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

  /*修改下拉框内容展示*/
  /*/deep/ .el-select__tags {*/
  /*  display: -webkit-box;*/
  /*  -webkit-box-orient: vertical;*/
  /*  -webkit-line-clamp: 1;*/
  /*  overflow: hidden;*/
  /*}*/

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
