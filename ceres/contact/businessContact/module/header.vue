<template>
  <div class="head-container">
    <div class="el-button-spli header-search-margin"></div>
    <el-form :inline="true" size="mini">
      <span class="searchline">&nbsp;</span>
      <el-form-item>
        <!-- 搜索 -->
        姓名：
        <el-input
          v-model="value"
          style="width: 200px;"
          class="filter-item"
          @keyup.enter.native="toQuery"/>
      </el-form-item>
      业务分类：
      <el-select v-model="category" multiple collapse-tags placeholder="请选择" :popper-append-to-body="false">
        <el-option
          style="height: 28px"
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <el-form-item>
        <el-button
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-search"
          @click="toQuery">搜索
        </el-button>
      </el-form-item>
      <el-form-item>
        <el-button
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-refresh-right"
          @click.stop="reset">
          重置
        </el-button>
      </el-form-item>
<!--      <div style="display: inline-block;margin: 0px 2px;">-->
<!--        <eForm ref="form" :is-add="true" :sup_this="sup_this"/>-->
<!--      </div>-->
    </el-form>
  </div>
</template>

<script>
  import checkPermission from '@/utils/permission'; // 权限判断函数
  import eForm from './form';

  export default {
    components: { eForm },
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
        value: '',
        options: [{
          value: '01',
          label: '业务联系协调人'
        }, {
          value: '02',
          label: '估值对接人'
        }, {
          value: '03',
          label: '数据对接人'
        }],
        category: []
      }

    },
    methods: {
      checkPermission,
      toQuery() {
        this.sup_this.query.value = this.value
        if (this.category) {
          this.sup_this.query.category = this.category.join()
        }
        this.sup_this.searchContactName();
      },

      reset() {
        this.value = ''
        this.category = ''
        //重置父界面的查询条件
        this.sup_this.page = 0;
        this.sup_this.size = 10;
        // 搜索查询结果置空
        this.sup_this.isParentFlag = false
        this.sup_this.queryContactName = []
        this.sup_this.getRoot();
      },
      // delCheck(ids){
      //   console.log('ids',ids)
      //   del({ids}).then(res => {
      //     this.init()
      //     // console.log(res)
      //     this.$notify({
      //       title: '删除成功',
      //       type: 'success',
      //       duration: 2500,
      //     });
      //     // this.getList()
      //   }).catch(err => {
      //   });
      // }
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

  /*下拉框多选之后隐藏选中的选项*/
  /deep/ .el-select__tags {
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    overflow: hidden;
  }
</style>
