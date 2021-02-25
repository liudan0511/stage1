<template>
  <div class="index">
    <template>
      <el-tabs v-model="activeName">
        <el-tab-pane label="备份系统要素" name="backUp">
          <keep-alive>
            <back-up :sup_this="sup_this" @getFundCode="getFundCode" ref="backupRef"></back-up>
          </keep-alive>
        </el-tab-pane>
        <el-tab-pane label="管理人报告" name="manager">
          <keep-alive>
            <manager :sup_this="sup_this" ref="managerRef"></manager>
          </keep-alive>
        </el-tab-pane>
        <el-tab-pane label="终版报告" name="final">
          <keep-alive>
            <final :sup_this="sup_this" ref="finalRef"></final>
          </keep-alive>

        </el-tab-pane>
      </el-tabs>
    </template>
  </div>
</template>

<script>

  import backUp from './backUp'
  import manager from './manager'
  import final from './final'

  export default {
    name: "",
    components: {
      backUp, manager, final,
    },
    data() {
      return {
        activeName: 'backUp',
        flag: true,
        sup_this: this,
        fundCode: ''
      }
    },
    methods: {
      isUnfold() {
        this.flag = !this.flag
      },
      getFundCode(fundCode) {
        this.fundCode = fundCode
        // this.$refs.managerRef.fundCode = fundCode;
      }
    }
  }
</script>
<style scoped>
  .index {
    width: 100%;
    padding: 0 20px;
  }

  /deep/ .el-tabs .el-row {
    margin-bottom: 0;
  }

  /*添加滚动条*/
  /deep/ .el-tabs__content {
    height: 620px;
    overflow-y: scroll !important;
    margin-top: 40px;
  }

  /deep/ .el-tabs__content::-webkit-scrollbar {
    width: 8px;
    /*height: 4px;*/
  }
  /deep/ .el-tabs__content::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
    background: rgba(0,0,0,0.2);
  }
  /deep/ .el-tabs__content::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
    border-radius: 10px;
    background: rgba(0,0,0,0.1);

  }

  /deep/ .el-tabs__nav-scroll{
    position: fixed;
    background: white;
    width: 100%;
    z-index: 100;
  }
</style>

