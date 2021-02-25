<template>
  <div class="head-container">
    <div class="el-button-spli header-search-margin"></div>
    <el-form :inline="true" size="mini" @submit.native.prevent>
      <span class="searchline">&nbsp;</span>
      <el-form-item label="产品名称:">
        <!--        filterable代表可输入-->
        <el-select filterable multiple collapse-tags v-model="productCode" placeholder="请选择" style="width: 350px;">
          <el-option
            v-for="item in productOption"
            :key="item.productCode"
            :label="item.productCodeAndName"
            :value="item.productCode"
            style="width: 400px">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="阅读状态：">
        <el-select v-model="readStatus" placeholder="请选择" style="width: 125px;">
          <el-option
            v-for="item in typeOption"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="监控日期：">
        <el-date-picker
          type="date"
          placeholder="开始日期"
          v-model="monitorDateStart"
          format="yyyy-MM-dd"
          value-format="yyyy-MM-dd"
          style="width: 140px;"
          :picker-options="pickerOptionsStart"
        ></el-date-picker>
        <span style="vertical-align: top">-</span>
        <el-date-picker
          type="date"
          placeholder="结束日期"
          format="yyyy-MM-dd"
          value-format="yyyy-MM-dd"
          v-model="monitorDateEnd"
          style="width: 140px;"
          :picker-options="pickerOptionsEnd"
        ></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-search"
          @click="search">搜索
        </el-button>
        <el-button class="filter-item"
                   size="mini"
                   type="primary"
                   icon="el-icon-refresh-right"
                   @click="reset">重置
        </el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import store from "@/store/index"
  import checkPermission from '@/utils/permission'; // 权限判断函数
  import {getProducts} from '@/api/ceres/riskWarning'

  export default {
    props: {
      query: {
        type: Object,
        required: true,
      },
      sup_this: {
        type: Object,
        required: true,
      },
      ids: {
        type: Array,
        required: false,
      }
    },
    data() {
      return {
        productCode: '',
        //阅读状态
        readStatus: '',
        // 监控开始日期
        monitorDateStart: '',
        // 监控结束日期
        monitorDateEnd: '',
        productOption: [],
        typeOption: [{
          value: '0',
          label: '未读'
        }, {
          value: '1',
          label: '已读'
        }],
        pickerOptionsStart: {
          disabledDate: time => {
            let endDateVal = this.monitorDateEnd;
            if (endDateVal) {
              return time.getTime() > new Date(endDateVal).getTime();
            }
          }
        },
        pickerOptionsEnd: {
          disabledDate: time => {
            let beginDateVal = this.monitorDateStart;
            if (beginDateVal) {
              return (
                time.getTime() <
                new Date(beginDateVal).getTime() - 1 * 24 * 60 * 60 * 1000
              );
            }
          }
        },
      }
    },
    mounted() {
      this.getProductInfo()
    },
    methods: {
      checkPermission,

      //获取所有产品名称
      getProductInfo() {
        let params = {
          creatorId: store.getters.user.userId
        }
        getProducts(params).then(res => {
          if (res.status === 'success') {
            this.productOption = res.data
          } else {
            this.$notify({
              title: res.msg,
              type: 'error',
              duration: 2500
            })
          }
        })
      },
      search() {
        if (this.productCode) {
          this.sup_this.query.productCode = this.productCode.join()
        } else {
          this.sup_this.query.productCode = ''
        }
        this.sup_this.query.readStatus = this.readStatus
        this.sup_this.query.monitorDateStart = this.monitorDateStart
        this.sup_this.query.monitorDateEnd = this.monitorDateEnd
        this.$parent.page = 0
        this.$parent.size = 10
        this.$parent.init()
      },

      reset() {
        this.productCode = ''
        this.readStatus = ''
        this.monitorDateStart = ''
        this.monitorDateEnd = ''
        this.$parent.page = 0
        this.$parent.size = 10
        this.search()
      },
    },
  };
</script>
<style lang="scss" scoped>
  .head-container {
    display: flex;
  }

  .el-button-spli {
    width: 3px;
    background-color: red;
    height: 24px;
    color: red;
    margin-right: 2px;
  }

  .header-search-margin {
    margin-top: 3px;
  }

  .el-input__inner {
    height: 28px;
  }

  .el-select__caret {
    margin-top: 7px;
  }

  /deep/ .el-input__inner {
    height: 32px;
  }

  /deep/ .el-select .el-input .el-select__caret {
    line-height: 32px;
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
</style>
