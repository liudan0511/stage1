<template>
  <div class="index">
    <el-table
      :data="tableData"
      :span-method="objectSpanMethod"
      :cell-class-name="tableRowClassName"
      border
      class="tableCls"
      style="width: 100%;">
      <el-table-column
        prop="businessFlow"
        label="业务流程">
      </el-table-column>

      <el-table-column label="服务内容" width="370" align="center">
        <template slot-scope="scope">
          <div v-html="scope.row.serviceContent">
            {{ scope.row.serviceContent }}
          </div>
        </template>
      </el-table-column>
      <el-table-column
        prop="contactName"
        label="姓名">
      </el-table-column>
      <el-table-column
        prop="contactPhone"
        label="办公电话">
      </el-table-column>
      <el-table-column
        prop="contactMobile"
        label="手机">
      </el-table-column>
      <el-table-column
        prop="contactMail"
        label="邮箱">
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
  import { getZtsList } from '@/api/ceres/ztsContact';
  import initData from '@/mixins/initData';

  export default {
    name: 'Index',
    mixins: [initData],
    data() {
      return {
        // total: 0,
        // size:10,
        tableData: [],
        // 流程集合
        flowIndexArr: [],
        conArr: [],
        // 服务内容
        flowObj: {},
        // 内容集合
        contentObj: {},
        // 处理样式数据集合
        flowStyleArray: [],
      };
    },
    mounted() {
      this.$nextTick(() => {
        this.getZtsList();
      })


    },
    methods: {
      getZtsList() {
        getZtsList().then(res => {
          this.tableData = res.data;
          // this.total = res.data.totalElements
          this.tableData.forEach((item, index) => {
            //合并业务流程
            if (this.flowObj[item.businessFlow]) {
              let nextItem = this.tableData[index].businessFlow;
              let prevItem = this.tableData[index - 1].businessFlow;
              if (item.businessFlow === nextItem) {
                this.flowObj[item.businessFlow].push(index);
                this.contentObj[item.serviceContent].push(index);
              } else if (item.businessFlow === prevItem) {
                this.flowObj[item.businessFlow].push(index);
                this.contentObj[item.serviceContent].push(index);
              }
            } else {
              this.flowObj[item.businessFlow] = [];
              this.flowObj[item.businessFlow].push(index);
              this.contentObj[item.serviceContent] = [];
              this.contentObj[item.serviceContent].push(index);
            }
          });

          for (let k in this.flowObj) {
            if (this.flowObj[k].length > 1) {
              this.flowIndexArr.push(this.flowObj[k]);
            }
          }

          for (let i in this.contentObj) {
            if (this.contentObj[i].length > 1) {
              this.conArr.push(this.contentObj[i]);
            }
          }

          // 构造样式集合
          for (const flowObjKey in this.flowObj) {
            let item = {
              flowName: flowObjKey,
              list: this.flowObj[flowObjKey],
            };
            this.flowStyleArray.push(item);
          }
          //去除鼠标悬停效果
          this.$nextTick(() => {
            document.getElementsByClassName('tableCls')[0].classList.remove('el-table--enable-row-hover');
            // this.$forceUpdate();
          })
        });
      },
      // 合并的方法
      objectSpanMethod({ row, rowIndex, column, columnIndex }) {
        // 第一列
        if (columnIndex === 0) {
          for (let i = 0; i < this.flowIndexArr.length; i++) {
            let element = this.flowIndexArr[i];
            for (let j = 0; j < element.length; j++) {
              let item = element[j];
              if (rowIndex === item) {
                if (j === 0) {
                  return {
                    rowspan: element.length,
                    colspan: 1,
                  };
                } else {
                  return {
                    rowspan: 0,
                    colspan: 0,
                  };
                }
              }
            }
          }
        }
        //因为需求是前两列中有相同的就合并，所以需要再次判断出来第一列中哪些行需要合并；
        // 服务内容合并
        if (columnIndex === 1) {
          for (let j = 0; j < this.conArr.length; j++) {
            let element = this.conArr[j];
            for (let k = 0; k < element.length; k++) {
              let item = element[k];
              if (rowIndex === item) {
                if (k === 0) {
                  return {
                    rowspan: element.length,
                    colspan: 1,
                  };
                } else {
                  return {
                    rowspan: 0,
                    colspan: 0,
                  };
                }
              }
            }
          }
        }
      },

      /**
       *
       * @param row
       * @param column
       * @param rowIndex
       * @param columnIndex
       * @returns {string}
       */
      tableRowClassName({ row, column, rowIndex, columnIndex }) {
        // 每个单元格都会进入, 带入rowIndex和columnIndex
        // 判断rowIndex是否在偶数行数组的list集合之中
        // 如果在偶数行, 则返回success
        // 如果在奇数行, 则返回warning
        let length = this.flowStyleArray.length;
        for (let i = 0; i < length; i++) {
          let line = this.flowStyleArray[i]
          if (line.list.includes(rowIndex)) {
            // console.log("***********"+line.index)
            if (i % 2 === 0) {
              return 'success-row';
            } else {
              return 'warning-row';
            }
          }
        }

      },

    },
  };
</script>

<style scoped>
  /deep/ body{
    background: white;
  }
  .index{
    padding: 20px;
    width: 98%;
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

  /deep/ .el-table .warning-row {
    background: #EFF2F7
  }

  /deep/ .el-table .success-row {
    background: white;
  }
</style>
