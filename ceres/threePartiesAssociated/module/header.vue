<template>
  <div class="head-container">
    <el-form :inline="true" size="mini" @submit.native.prevent>
      <span class="searchLine">&nbsp;</span>
      <el-form-item label="产品名称:">
        <el-select
          collapse-tags
          multiple
          v-model="productCode"
          placeholder="请选择"
          style="width: 370px;height: 32px;">
          <el-option
            v-for="item in productCodeOptions"
            :key="item.productCode"
            :label="item.productFullName"
            :value="item.productCode"
            style="width: 370px">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="签约类型：">
        <el-select
          collapse-tags
          multiple
          v-model="signType"
          placeholder="请选择"
          style="width: 180px;">
          <el-option
            v-for="item in signTypes"
            :key="item.key"
            :label="item.value"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <!--状态选择-->
      <el-form-item label="状态:">
        <el-select
          collapse-tags
          multiple
          v-model="flowStatus"
          placeholder="请选择"
          style="width: 180px;">
          <el-option
            v-for="item in statusOptions"
            :key="item.key"
            :label="item.value"
            :value="item.key">
          </el-option>
        </el-select>
      </el-form-item>
      <!--日期部分-->
      <el-form-item label="申请日期:" ref="dateRef">
        <el-date-picker
          type="date"
          placeholder="开始日期"
          v-model="startDate"
          format="yyyy-MM-dd"
          value-format="yyyy-MM-dd HH:mm:ss"
          :picker-options="pickerOptionsStart"
          style="width: 140px;"
        ></el-date-picker>
        <span style="vertical-align: top">-</span>
        <el-date-picker
          type="date"
          placeholder="结束日期"
          format="yyyy-MM-dd"
          value-format="yyyy-MM-dd HH:mm:ss"
          v-model="endDate"
          :picker-options="pickerOptionsEnd"
          style="width: 140px;"
        ></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-search"
          @click="toQuery">
          <span>查询</span>
        </el-button>
        <el-button
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-refresh"
          @click="reset">
          <span>重置</span>
        </el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>

import { getProductInfos } from 'api/ceres/threePartiesAssociated'
import store from '@/store'

export default {
  components: {},
  props: {
    // 父组件对象
    sup_this: {
      type: Object,
      required: true,
    },
    // 父组件中查询条件对象
    query: {
      type: Object,
      required: true,
    },
    pageType: {
      type: String,
      required: true,
    },
  },
  watch: {
    pageType(val) {
      this.changeSignType()
    },
  },
  data() {
    return {
      // 表格数据
      tableData: [],

      // 基金名称
      productCode: '',
      // 基金可选项
      productCodeOptions: [],
      // 签约类型
      signType: '',
      // 类型集合
      signTypes: [],
      // 签约类型可选项,
      signTypesOptionsObj: {
        sign: [
          {
            key: '01',
            value: '三方存管',
          },
          {
            key: '02',
            value: '融资融券',
          },
          {
            key: '03',
            value: '银期签约',
          },
          {
            key: '04',
            value: '银衍签约',
          },
        ],
        cancel: [
          {
            key: '05',
            value: '解除银期签约',
          },
          {
            key: '06',
            value: '解除银衍签约',
          },
        ],
      },

      // 选中状态
      status: '',
      statusOptions: [
        {
          key: '01',
          value: '申请',
        },
        {
          key: '02',
          value: '初审',
        },
        {
          key: '03',
          value: '上传用印文件',
        },
        {
          key: '04',
          value: '复审',
        },
        {
          key: '05',
          value: '三方关联',
        },
        {
          key: '06',
          value: '完成',
        },
        {
          key: '07',
          value: '终止',
        },

      ],
      // 时间对象
      startDate: '',
      endDate: '',
      // 时间限制条件
      pickerOptions: {
        // 禁用当前日之前的时间 86400000 ms
        disabledDate(time) {
          return time.getTime() < Date.now() - 8.64e7
        },
      },
      pickerOptionsStart: {
        disabledDate: (time) => {
          if (this.endDate) {
            return time.getTime() > Date.now() || time.getTime() > new Date(this.endDate).getTime()
          } else {
            return time.getTime() > Date.now()
          }
        },
      },
      pickerOptionsEnd: {
        disabledDate: (time) => {
          return time.getTime() < new Date(this.startDate).getTime() || time.getTime() > Date.now()
        },
      },
      flowStatus: '',
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.queryProductInfo()
      this.changeSignType()
    })
  },
  methods: {
    queryProductInfo() {
      let params = {
        creatorId: store.getters.user.userId,
      }
      getProductInfos(params).then((res) => {
        if (res.status === 'success') {
          this.productCodeOptions = res.data
        } else {
          this.$notify({
            title: res.msg,
            type: 'warning',
            duration: 2500,
          })
        }
      }).catch((error) => {
        console.error(error)
      })
    },
    getDataTypeByPath() {
      let path = this.$route.path
      if (path.indexOf('sign') > -1) {
        return 'sign'
      } else {
        return 'cancel'
      }
    },
    changeSignType() {
      let type = this.pageType
      if (!type) {
        type = this.getDataTypeByPath()
      }
      // 通过页面类型展示签约选择项
      if (type === 'sign') {
        this.signTypes = this.signTypesOptionsObj.sign
      } else if (type === 'cancel') {
        this.signTypes = this.signTypesOptionsObj.cancel
      } else {
        console.error('pageType is missed')
      }
    },
    toQuery() {
      // header组件内变量部分
      if (this.productCode) {
        this.query.productCode = this.productCode.join()
      } else {
        this.query.productCode = ''
      }

      if (this.signType) {
        this.query.signType = this.signType.join()
      } else {
        this.query.signType = ''
      }

      if (this.flowStatus) {
        this.query.flowStatus = this.flowStatus.join()
      } else {
        this.query.flowStatus = ''
      }
      this.query.startDate = this.startDate
      this.query.endDate = this.endDate

      // 分页重新初始化
      this.$parent.page = 0
      this.$parent.size = 10
      // 重新初始化
      this.$parent.init()
    },
    reset() {
      // header组件内变量部分
      this.productCode = ''
      this.signType = ''
      this.flowStatus = ''
      this.startDate = ''
      this.endDate = ''

      this.query.productCode = this.productCode
      this.query.signType = this.signType
      this.query.flowStatus = this.flowStatus
      this.query.startDate = this.startDate
      this.query.endDate = this.endDate

      this.$parent.page = 0
      this.$parent.size = 10
      this.$parent.init()
    },

  },
}
</script>
<style lang="scss" scoped>
  .head-container {
    display: flex;
    flex-flow: row wrap;
  }

  /*输入框中样式*/
  /deep/ .el-select .el-tag {
    max-width: 230px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  /*/deep/ .el-select__tags {*/
  /*  max-width: 300px;*/
  /*}*/

  /*/deep/ .el-select__tags-text {*/
  /*  display: block;*/
  /*  !*max-width: 200px;*!*/
  /*  overflow: hidden;*/
  /*  text-overflow: ellipsis;*/
  /*  white-space: nowrap;*/
  /*}*/

  /*/deep/ .el-select__input {*/
  /*  !*width: 100%;*!*/
  /*  min-width: 100px !important;*/
  /*  position: relative;*/
  /*  !*left: -75px;*!*/
  /*}*/

  /deep/ .el-select .el-input .el-input__inner {
    height: 32px;
    line-height: 32px;
  }

  /deep/ .el-form-item--mini, .el-form-item__label {
    height: 32px;
    line-height: 32px;
  }

  /deep/ .el-date-editor .el-input__inner {
    height: 32px;
    line-height: 32px;
  }

  .searchLine {
    display: inline-block;
    width: 3px;
    height: 24px;
    background: red;
    position: relative;
    vertical-align: sub;
    margin-right: 6px;
  }

  /deep/ .el-input--mini .el-input__icon {
    line-height: 34px;
  }

</style>
