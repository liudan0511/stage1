<template>
  <div class="manager">
    <el-form ref="form" :model="form" :inline="true" size="mini" @submit.native.prevent :rules="rules">
      <el-form-item label="产品名称：" prop="fundCode">
        <el-select
          filterable
          v-model="fundCode"
          placeholder="请输入产品代码或产品名称"
          style="width: 320px"
          @change="onSelectedDrug($event)">
          <el-option
            v-for="item in options"
            :key="item.code"
            :label="item.name"
            :value="item.code"
            style="width: 320px">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="年份：" prop="year">
        <el-date-picker
          v-model="year"
          type="year"
          popper-class="yearCls"
          style="width: 220px"
          placeholder="选择年">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="报告类型：" prop="reportType">
        <el-select v-model="reportType" placeholder="请选择">
          <el-option
            v-for="item in  types"
            :key="item.code"
            :label="item.name"
            :value="item.code">
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
      </el-form-item>

      <div class="explain" v-if="obj.type==='1'">
        填写说明：如报告期内高管、基金经理及其关联基金经验、基金运作遵规守信情况、基金投资策略和业绩表现、对宏观经济、证券市场及其行业走势展望、内部基金监察稽核工作、基金估值程序、基金运作情况和运用杠杆情况、投资收益分配和损失承担情况、会计师事务所出具非标准审计报告所涉相关事项、对本基金持有人数或基金资产净值预警情形、可能存在的利益冲突等
      </div>
      <!--      股权类-->
      <div class="explain" v-if="obj.type==='2'">
        填写说明：如报告期内高管及其关联基金情况、高管变动情况、基金运作遵规守信情况、基金投资策略和业绩表现、对宏观经济及其行业走势展望、内部基金监察稽核工作、基金估值程序、基金运作情况、投资收益分配和损失承担情况、关联交易等可能存在的利益冲突、关于基金负债以及潜在负债或担保的简要说明等
      </div>
      <el-row>
        <el-form-item label="" prop="content" class="contentCls">
          <el-input
            class="textarea"
            type="textarea"
            :rows="13"
            placeholder="请按照基金合同约定填写"
            v-model="form.content">
          </el-input>
        </el-form-item>
      </el-row>
    </el-form>

    <div
      class="footer"
    >
      <el-button
        type="primary"
        @click="doSubmit"
        style="margin-top: 10px"
      >提交
      </el-button>
    </div>
  </div>
</template>

<script>
  import store from "@/store/index"
  import { getProInfo, getType, toQuery, doSubmitManager, noHalfYearType } from '@/api/ceres/periodReport'
  import Bus from '@/utils/eventBus'

  export default {
    name: "Manager",
    props: {
      sup_this: {
        type: Object,
        default: null
      },
    },
    data() {
      return {
        fundCode: '',
        reportType: '',
        year: '',
        form: {
          content: ''
        },
        options: [],
        types: [],
        noHalfYearTypes: [],
        years: [],
        //展示类型 1为证券，2为股权
        obj: {},
        restaurant: [],
        rules: {
          content: [],
        },
        //  展示填写说明
        hintFlag: false
      }
    },

    mounted() {
      this.getProInfo()
      this.noHalfYear()
      this.initYearBox()
      this.getQuarter()
      Bus.$on('getFundCode', (data) => {
        this.fundCode = data
        this.options.map(val => {
          if (data === val.code) {
            this.obj = val;
          }
        })
        let year = this.year.toString().substr(11, 4)
        let params = {
          fundCode: this.fundCode,
          year: year,
          reportType: this.reportType,
          // userId: "CS02955",
          userId: store.getters.user.userId
        }
        toQuery(params).then(res => {
          if (res.status === 'success') {
            if (!res.data.content) {
              this.$confirm("是否复用上一期的管理人报告", "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
              }).then(() => {
                let reportType
                // 如果搜索出来的结果是空的，则调用上一季度的内容
                if (this.reportType === '020201') {
                  year = year - 1
                  reportType = '020204'
                } else if (this.reportType === '020202') {
                  reportType = '020201'
                } else if (this.reportType === '020203') {
                  reportType = '020202'
                } else if (this.reportType === '020204') {
                  reportType = '020203'
                }
                let params = {
                  fundCode: this.fundCode,
                  year: year,
                  reportType: reportType,
                  userId: store.getters.user.userId
                }
                toQuery(params).then(res => {
                  if (res.status === 'success') {
                    this.form.content = res.data.content
                  } else {
                    this.$notify({
                      title: res.msg,
                      type: 'error',
                      duration: 2500
                    })
                  }
                })
              }).catch(e => e)
            } else {
              this.form.content = res.data.content
            }
          } else {
            this.$notify({
              title: res.msg,
              type: 'error',
              duration: 2500
            })
          }
        }).catch(err => {
          this.$notify({
            title: '未查询到内容',
            type: 'error',
            duration: 2500
          })
        })
      })
    },

    methods: {
      //获取季度
      getQuarter() {
        var today = new Date();//获取当前时间
        var y = today.getFullYear();
        var m = today.getMonth() + 1;
        var d = today.getDate();
        let date1, date2
        var getYearWeek = function (a, b, c) {
          date1 = new Date(a, parseInt(b) - 1, c)
          date2 = new Date(a, 0, 1)
          d = Math.round((date1.valueOf() - date2.valueOf()) / 86400000);
          return Math.ceil((d + ((date2.getDay() + 1) - 1)) / 7);
        };
        var quarter = "";
        var result = getYearWeek(y, m, d);
        let week, day
        if (m < 4) {
          quarter = 1;
          week = result;
        } else if (m < 7) {
          quarter = 2;
          week = result - getYearWeek(y, 4, 1);
          day = new Date(y, 4, 1);
          if (day.getDay() > 1) {
            week += 1;
          }
        } else if (m < 10) {
          quarter = 3;
          week = result - getYearWeek(y, 7, 1);
          day = new Date(y, 7, 1);
          if (day.getDay() > 1) {
            week += 1;
          }
        } else {
          quarter = 4;
          week = result - getYearWeek(y, 10, 1);
          day = new Date(y, 10, 1);
          if (day.getDay() > 1) {
            week += 1;
          }
        }
        if (quarter === 1) {
          this.reportType = '020204'
        } else if (quarter === 2) {
          this.reportType = '020201'
        } else if (quarter === 3) {
          this.reportType = '020202'
        } else if (quarter === 4) {
          this.reportType = '020203'
        }
        this.year = new Date(Date.UTC(y))
      },
      //年份
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
          this.years.push(params);
        }
      },

      //  触发下拉框
      onSelectedDrug(event) {
        this.options.map(val => {
          if (event === val.code) {
            if (val.type === '2') {
              this.getType()
            } else {
              this.noHalfYear()
            }
            this.obj = val;
          }
        })
      },
      //获取所有产品
      getProInfo() {
        getProInfo(
          store.getters.user.userId
        ).then(res => {
          if (res.status === 'success') {
            this.options = res.data
          } else {
            this.$notify({
              title: res.msg,
              type: 'error',
              duration: 2500
            })
          }
        })
      },


      //获取报告类型
      getType() {
        getType().then(res => {
          if (res.status === 'success') {
            this.types = res.data
          } else {
            this.$notify({
              title: res.msg,
              type: 'error',
              duration: 2500
            })
          }
        })
      },

      // 无半年报类型
      noHalfYear() {
        noHalfYearType().then(res => {
          if (res.status === 'success') {
            this.types = res.data
          } else {
            this.$notify({
              title: res.msg,
              type: 'error',
              duration: 2500
            })
          }
        })
      },

      //搜索
      toQuery() {
        let year
        if (this.year) {
          year = this.year.toString().substr(11, 4)
        } else {
          year = ''
        }
        this.rules.content = [
          { required: false }
        ]
        if (this.fundCode !== '' && this.reportType !== '' && year !== '') {
          let params = {
            fundCode: this.fundCode,
            year: year,
            reportType: this.reportType,
            userId: store.getters.user.userId
          }
          toQuery(params).then(res => {
            if (res.status === 'success') {
              if (res.data) {
                this.form.content = res.data.content
              }
            } else {
              this.$notify({
                title: res.msg,
                type: 'error',
                duration: 2500
              })
            }
          }).catch(err => {
            this.$notify({
              title: '未查询到内容',
              type: 'error',
              duration: 2500
            })
          })
        } else {
          this.$notify({
            title: '产品名称、年份、报告类型为必选',
            type: 'error',
            duration: 2500
          })
        }

      },

      //提交
      doSubmit() {
        let year = this.year.toString().substr(11, 4)
        this.rules.content = [
          { required: true, message: "请输入内容", trigger: ["blur"] },
        ]
        if (this.fundCode !== '' && this.reportType !== '' && this.year !== '') {
          this.$refs["form"].validate(valid => {
            if (valid) {
              let data = {
                fundCode: this.fundCode,
                year: year,
                reportType: this.reportType,
                userId: store.getters.user.userId,
                content: this.form.content
              }
              doSubmitManager(data).then(res => {
                if (res.status === 'success') {
                  this.$notify({
                    title: '提交成功',
                    type: 'success',
                    duration: 2500
                  })
                } else {
                  this.$notify({
                    title: res.msg,
                    type: 'error',
                    duration: 2500
                  })
                }
                // this.sup_this.activeName = 'final'
              }).catch(err => {
                this.$notify({
                  title: '提交失败',
                  type: 'error',
                  duration: 2500
                })
              })
            }
          });
        } else {
          this.$notify({
            title: '请先搜索',
            type: 'error',
            duration: 2500
          })
        }


      }
    }
  }
</script>
<style lang="scss">
  .yearCls {
    &.el-date-picker .el-picker-panel__content {
      width: 185px !important;
    }

    &.el-date-picker {
      width: 220px !important;
    }
  }
</style>
<style scoped>
  * {
    font-size: 14px;
    font-family: Source Han Sans SC;
    font-weight: 400;
  }

  .el-tabs__nav-scroll {
    height: 50px;
  }

  .el-tabs__nav {
    height: 50px;
    line-height: 50px;
  }

  /*填写说明*/
  .explain {
    width: 100%;
    height: 60px;
    /*margin: 20px 0 20px 0;*/
    font-size: 14px;
    font-family: Source Han Sans SC;
    font-weight: 400;
    color: rgba(187, 187, 187, 1);
    line-height: 22px;
    margin-bottom: 10px;
  }

  .footer {
    text-align: center;
  }

  .footer .el-button {
    width: 120px;
    height: 42px;
  }

  /*.textarea {*/
  /*  width: 1200px;*/
  /*  height: 280px;*/
  /*  !*background:rgba(245,247,250,1);*!*/
  /*  !*border:1px solid rgba(224,229,233,1);*!*/
  /*}*/

  .manager {
    width: 99%;
  }
  /deep/ .el-input .el-input__inner {
    height: 32px;
  }

  /*文本框样式*/
  /deep/ .el-textarea__inner {
    resize: none;
    width: 100%;
    height: 285px;
  }

  /deep/ .el-scrollbar {
    width: 305px;
  }

  .contentCls {
    width: 100%;
  }

  .contentCls /deep/ .el-form-item__content {
    width: 100%;
  }

  .contentCls /deep/ .el-textarea {
    width: 100%;
  }
</style>



