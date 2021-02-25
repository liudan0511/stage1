<template>
  <div class="index">
    <eHeader
      ref="header"
      :sup_this="sup_this"
      :query="query"
    />
    <el-button
      v-if="permitted"
      class="filter-item"
      size="mini"
      style="padding: 4px 10px;margin-bottom: 10px"
      type="primary"
      icon="el-icon-plus"
      @click="add"
    >新增
    </el-button>
    <!--表格渲染-->
    <el-table
      row-key="key"
      ref="tableDom"
      :data="rootList"
      v-show="queryContactName.length === 0"
      stripe
      border
      lazy
      size="small"
      highlight-current-row
      style="width: 100%;"
      :load="load"
      :tree-props="{children: 'children', hasChildren: 'hasChildren'}">
      <el-table-column
        :show-overflow-tooltip="true"
        prop="productName"
        label="产品名称"
        class="demo-table-expand"
        style="width: 200px;"
        align="left"
      />

      <el-table-column
        :show-overflow-tooltip="true"
        prop="newCategory"
        label="业务分类"
      />

      <el-table-column
        :show-overflow-tooltip="true"
        prop="contactName"
        label="姓名"
      />

      <el-table-column
        :show-overflow-tooltip="true"
        prop="contactMobile"
        label="手机"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        prop="contactMail"
        label="邮箱信息"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        prop="contactPhone"
        label="固定电话"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        label="操作"
        width="150px"
        align="center"
        v-if="permitted"
      >
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.id"
            @click="handleEdit(scope.$index,scope.row)"
            size="mini"
            type="success">
            编辑
          </el-button>
          <el-button v-if="scope.row.id" @click="handleDelete(scope.$index,scope.row)" type="danger" size="mini">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--          搜索表格渲染-->
    <el-table
      row-key="key"
      :data="queryContactName"
      v-show="queryContactName.length > 0"
      stripe
      border
      lazy
      size="small"
      highlight-current-row
      style="width: 100%;"
      :load="load"
    >
      <el-table-column
        :show-overflow-tooltip="true"
        prop="productName"
        label="产品名称"
      />

      <el-table-column
        :show-overflow-tooltip="true"
        prop="newCategory"
        label="业务分类"
      />

      <el-table-column
        :show-overflow-tooltip="true"
        prop="contactName"
        label="姓名"
      />

      <el-table-column
        :show-overflow-tooltip="true"
        prop="contactMobile"
        label="手机"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        prop="contactMail"
        label="邮箱信息"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        prop="contactPhone"
        label="固定电话"
      />
      <el-table-column
        :show-overflow-tooltip="true"
        label="操作"
        width="150px"
        align="center"
        v-if="permitted"
      >
        <template slot-scope="scope">
          <!-- 编辑-->

          <el-button
            v-if="scope.row.id"
            @click="handleEdit(scope.$index,scope.row)"
            size="mini"
            type="success">
            编辑
          </el-button>
          <el-button
            v-if="scope.row.id"
            @click="handleDelete(scope.$index,scope.row)"
            type="danger"
            size="mini">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      :total="total"
      background
      :current-page="page + 1"
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
      :data="editData"
      :title="title"
      :status="status"
      @change="change"
    />
  </div>
</template>

<script>
  import eHeader from './module/header'
  import eForm from './module/form'
  import { getRoot, getContactInfo, del } from '@/api/ceres/businessContact'
  import store from '@/store/index'
  import initData from '@/mixins/initData'

  export default {
    name: 'Index',
    mixins: [initData],
    components: { eHeader, eForm },
    data() {
      return {
        title: '添加',
        status: 'add',
        //删除时用到
        maps: new Map(),
        currentPage: 1,
        tableHeight: 0,
        columns: [
          {
            text: '名称',
            value: 'name'
          }
        ],
        sup_this: this,
        rootList: [],
        //二级列表数据
        queryContactName: [],
        //判断是一级展示还是二级展示
        isParentFlag: false,

        //是否显示弹框
        dialogFormVisible: false,
        // 修改
        editData: {},
        id: '',
      }
    },
    created() {
      this.$nextTick(() => {
        this.getRoot()
      })
    },
    computed: {
      // 获取权限
      permitted() {
        if (store.getters.user.roles[0] === 'ADMIN') {
          return store.getters.user.roles[0]
        }
      }
    },
    methods: {


      //获取一级路由
      getRoot() {
        let params = {
          page: this.page,
          size: this.size,
          creatorId: store.getters.user.userId
        }
        getRoot(params).then(res => {
          if (res.status === 'success') {
            let tableData = res.data.content
            this.total = res.data.totalElements;
            tableData.forEach(function (item) {
              delete item.children;
            });
            let map = []; //构建map
            tableData.forEach((i, index) => {
              let item = i
              item.id = ''
              // item.key= i.productCode + index
              item.key = i.productCode
              item.hasChildren = true
              item.children = []
              map.push(item)
            });
            this.rootList = [];
            this.$nextTick(() => {
              this.rootList = map;
            })
          } else {
            this.$notify({
              title: res.msg,
              type: 'warning',
              duration: 2500
            })
          }
        })
      },

      pageChange(e) {
        this.page = e - 1
        if (this.isParentFlag) {
          this.searchContactName()
        } else {
          this.getRoot()
        }
        this.page = 0
      },
      sizeChange(e) {
        this.page = 0
        this.size = e
        if (this.isParentFlag) {
          this.searchContactName()
        } else {
          this.getRoot()
        }
      },

      //树形结构
      load(tree, treeNode, resolve) {
        const productCode = tree.productCode
        this.maps.set(productCode, { tree, treeNode, resolve })
        getContactInfo({
          productCode: productCode
        }).then(res => {
          if (res.status === 'success') {
            let data = res.data.content
            data.forEach(item => {
              item.hasChildren = false
              item.key = item.id
              if (item.category === '01') {
                item.newCategory = '业务联系协调人'
              } else if (item.category === '02') {
                item.newCategory = '估值对接人'
              } else {
                item.newCategory = '数据对接人'
              }
            })
            this.$nextTick(() => {
              resolve(data)
            })
          } else {
            this.$notify({
              title: res.msg,
              type: 'warning',
              duration: 2500
            })
          }
        })
      },

      //搜索
      searchContactName() {
        let value = this.query.value
        let category = this.query.category
        let params = {
          creatorId: store.getters.user.userId,
          category: category,
          contactName: value,
          page: this.page,
          size: this.size,
        }
        getContactInfo(params).then(res => {
          if (res.status === 'success') {
            if (res.data && res.data.content.length > 0) {
              this.queryContactName = res.data.content
              this.queryContactName.forEach(item => {
                if (item.category === '01') {
                  item.newCategory = '业务联系协调人'
                } else if (item.category === '02') {
                  item.newCategory = '估值对接人'
                } else {
                  item.newCategory = '数据对接人'
                }
              })
              this.total = res.data.totalElements;
              this.isParentFlag = true
            } else {
              this.total = 0
              this.rootList = [];
              this.queryContactName = []
            }
          } else {
            this.$notify({
              title: res.msg,
              type: 'warning',
              duration: 2500
            })
            this.queryContactName = []
          }
        })

      },

      //批量删除触发
      handleSelectionChange(val) {
        val.map(item => {
          if (this.ids.indexOf(item.id) >= 0) {
            this.ids = this.ids.splice(this.ids.indexOf(item.id), 1)
          } else {
            this.ids = this.ids.concat(item.id)
          }
        })
      },

      //单个删除
      handleDelete(index, row) {
        this.$confirm("是否要删除该数据", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(() => {
          del(row.id).then(res => {
            if (res.status === 'success') {
              this.$notify({
                title: '删除成功',
                type: 'success',
                duration: 2500,
              });
              if (this.isParentFlag) {
                this.searchContactName()
              } else {
                this.getRoot()
                const { productCode } = row
                const { tree, treeNode, resolve } = this.maps.get(productCode)
                this.$set(this.$refs['tableDom'].store.states.lazyTreeNodeMap, productCode, [])
                // console.log('res1',this.$refs['tableDom'].store.states.lazyTreeNodeMap)
                this.load(tree, treeNode, resolve);
              }
            } else {
              this.$notify({
                title: res.msg,
                type: 'warning',
                duration: 2500
              })
            }
          }).catch(err => {
            this.$notify({
              title: '删除失败',
              type: 'error',
              duration: 2500,
            });
          })
        });

      },
      add() {
        this.dialogFormVisible = true
        this.status = 'add'
        this.title = '新增'
        this.$refs.formAlert.rules.productCode = [
          { required: true, message: "请选择产品", trigger: "change" }
        ]
        this.$refs.formAlert.rules.category = [
          { required: true, message: "请选择分类", trigger: ["blur", "change"] }
        ]
        this.$refs.formAlert.rules.contactMobile = [
          { required: true, message: "请输入手机号", trigger: "blur" },
          {
            pattern: /^1[3456789]\d{9}$/,
            message: "请输入正确的11位手机号码!",
            trigger: ["blur", "change"]
          }
        ]
        this.$refs.formAlert.rules.contactMail = [
          { required: true, message: "请输入邮箱地址", trigger: "blur" },
          {
            type: "email",
            message: "请输入正确的邮箱地址",
            trigger: ["blur", "change"]
          }
        ]
      },
      handleEdit(index, row) {
        this.dialogFormVisible = true
        this.editData = row
        this.status = 'edit'
        this.title = '编辑'
        this.$nextTick(() => {
          this.$refs.formAlert.edit()
        })
      },
      //子组件传到父组件的值通过这个方法改变
      change(flag) {
        this.dialogFormVisible = flag
      }
    }

  }
</script>

<style scoped>
  * {
    font-size: 14px;
  }

  .index {
    padding: 20px;
    width: 98%;
  }

  .el-col-md-10 {
    width: 100%;
  }

  /deep/ .el-table .el-table__header th.is-left {
    text-align: center;
  }

  /deep/ .el-button--danger {
    background: white;
    color: red;
  }

  /*表格箭头样式*/
  /deep/ .el-table .el-table__expand-icon{
    width: 14px;
    height: 14px;
    line-height: 13px;
    border: 1px solid grey;
    border-radius: 2px;
  }
  /deep/ .el-table .el-table__expand-icon {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }

  /deep/ .el-table__expand-icon .el-icon-arrow-right:before{
    content: "+";
  }
  /deep/ .el-table__expand-icon.el-table__expand-icon--expanded .el-icon-arrow-right:before{
    content: "-";
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


