<template>
  <el-dialog
    v-el-drag-dialog
    :append-to-body="true"
    :visible.sync="dialogFormVisible"
    :title="title"
    width="500px"
    :before-close="closeDialog"
  >
    <el-form ref="form" :model="form" :rules="rules" size="middle" label-width="83px">
      <!--        添加时展示产品名称-->
      <el-form-item label="产品名称:" prop="productCode" v-if="!form.id">
        <el-select
          filterable
          multiple
          v-model="form.productCode"
          placeholder="请选择产品名称"
          style="width: 370px;"
        >
          <el-option
            v-for="item in options"
            :key="item.fundCode"
            :label="item.fundName"
            style="width: 370px"
            :value="item.fundCode"
          ></el-option>
        </el-select>
      </el-form-item>

      <!--      修改时展示产品名称-->
      <el-form-item label="产品名称:" prop="productCode" v-else>
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input v-model="form.productName" :disabled="true" style="width: 370px;"/>
          </el-col>
        </el-row>
      </el-form-item>

      <el-form-item label="业务分类:" prop="category" v-if="!form.id">
        <el-select v-model="form.category" placeholder="请选择业务分类" style="width: 370px;">
          <el-option
            v-for="item in categories"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <!--      修改时展示业务分类-->
      <el-form-item label="业务分类:" prop="category" v-else>
        <el-select v-model="form.category" disabled placeholder="请选择" style="width: 370px;">
          <el-option
            v-for="item in categories"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="姓名:" prop="contactName">
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-autocomplete
              style="width: 370px;"
              class="inline-input"
              v-model="form.contactName"
              value-key="contactName"
              :fetch-suggestions="querySearch"
              placeholder="请输入姓名"
              :trigger-on-focus="true"
              @select="handleSelect"
              @input="inputChange"
            ></el-autocomplete>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="手机:" prop="contactMobile" v-if="!form.contactId&&!form.id">
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              class="inline-input"
              style="width: 370px;"
              maxlength="11"
              :disabled="flag"
              v-model="form.contactMobile"
              placeholder="请输入手机号"
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="手机:" prop="contactMobile" v-else-if="!form.contactId">
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              class="inline-input"
              style="width: 370px;"
              maxlength="11"
              :disabled="flag"
              v-model="form.contactMobile"
              placeholder="请输入手机号"
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="手机:" prop="contactMobile" v-else>
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              class="inline-input"
              style="width: 370px;"
              maxlength="11"
              :disabled="flag"
              v-model="form.contactMobile"
              placeholder=""
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="邮箱信息:" prop="contactMail" v-if="!form.contactId&&!form.id">
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              class="inline-input"
              style="width: 370px;"
              :disabled="flag"
              maxlength="25"
              v-model="form.contactMail"
              placeholder="请输入邮箱信息"
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="邮箱信息:" prop="contactMail" v-else-if="!form.contactId">
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              class="inline-input"
              style="width: 370px;"
              :disabled="flag"
              maxlength="25"
              v-model="form.contactMail"
              placeholder="请输入邮箱信息"
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="邮箱信息:" prop="contactMail" v-else>
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              class="inline-input"
              style="width: 370px;"
              :disabled="flag"
              maxlength="25"
              v-model="form.contactMail"
              placeholder=""
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>

      <el-form-item label="固定电话:" prop="contactPhone" v-if="!form.contactId&&!form.id">
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              type="text"
              class="inline-input"
              style="width: 370px;"
              :disabled="flag"
              v-model="form.contactPhone"
              placeholder="请输入固定电话"
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="固定电话:" prop="contactPhone" v-else-if="!form.contactId">
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              type="text"
              class="inline-input"
              style="width: 370px;"
              :disabled="flag"
              v-model="form.contactPhone"
              placeholder="请输入固定电话"
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="固定电话:" prop="contactPhone" v-else>
        <el-row class="demo-autocomplete">
          <el-col :span="12">
            <el-input
              type="text"
              class="inline-input"
              style="width: 370px;"
              :disabled="flag"
              v-model="form.contactPhone"
              placeholder=""
            ></el-input>
          </el-col>
        </el-row>
      </el-form-item>


    </el-form>
    <div slot="footer" class="dialog-footer" style="text-align:center;">
      <el-button @click="cancel">取消</el-button>
      <el-button type="primary" @click="doSubmit">确认</el-button>
    </div>
  </el-dialog>
</template>

<script>
  import { add, edit, getPro, getContacts } from "@/api/ceres/businessContact";
  import store from "@/store/index";

  export default {
    // props: ["title", "status", "dialogFormVisible", "data"],
    props: {
      title: {
        type: String,
        required: true
      },
      status: {
        type: String,
        required: true
      },
      dialogFormVisible: {
        type: Boolean,
        required: true
      },
      data: {
        type: Object,
      },
    },
    data() {
      var accountCharacter = (rule, value, callback) => {
        if (value) {
          if (value.length > 30) {
            callback(new Error("最长输入30个字符!"));
          } else {
            callback();
          }
        }
        callback();
      };

      return {
        //根据姓名输入与否来判断是否展示电话等内容
        flag: false,
        //姓名数据
        restaurants: [],
        //姓名结果
        results: '',
        //产品数据
        options: [],
        // 一下两个 全选产品时用到
        oldOptions: [],
        all: [],
        placeHolderFlag: false,
        form: {
          id: "",
          contactId: "",
          productCode: "",
          productName: "",
          category: "",
          contactName: "",
          contactPhone: "",
          contactMobile: "",
          contactMail: "",
          search: ""
        },
        categories: [
          {
            value: "01",
            label: "业务联系协调人"
          },
          {
            value: "02",
            label: "估值对接人"
          },
          {
            value: "03",
            label: "数据对接人"
          }
        ],
        rules: {
          productCode: [
            // {required: true, message: "请选择产品", trigger: "change"}
          ],
          category: [
            // {required: true, message: "请选择分类", trigger: ["blur", "change"]}
          ],
          contactName: [
            { required: true, message: "请输入姓名", trigger: ["blur", "change"] },
          ],
          contactPhone: [
            { validator: accountCharacter, trigger: ["blur", "change"] }
          ],
          contactMobile: [
            { required: true, message: "请输入手机号", trigger: "blur" },
            {
              pattern: /^1[3456789]\d{9}$/,
              message: "请输入正确的11位手机号码!",
              trigger: ["blur", "change"]
            }
            ],
          contactMail: [
            { required: true, message: "请输入邮箱地址", trigger: "blur" },
            {
              type: "email",
              message: "请输入正确的邮箱地址",
              trigger: ["blur", "change"]
            }
          ]
        }
      };
    },
    created() {
      this.$nextTick(() => {
        this.getPro()
        this.getContacts()
      })
    },
    methods: {
      // //全选
      // selChange(valArr) {
      //   let allIdArr = this.all;
      //   let oldVal;
      //   if (this.oldOptions.length === 1) {
      //     oldVal = this.oldOptions[0]
      //   } else {
      //     oldVal = []
      //   }
      //   // 当前选中的有'全选'
      //   if (valArr.includes("allSelect")) {
      //     this.form.productCode = allIdArr;
      //   }
      //   // // 旧数据包含'全选'，当前选中数据不包含全选
      //   if (oldVal.includes("allSelect") && !valArr.includes("allSelect")) {
      //     this.form.productCode = [];
      //   }
      //   // // 旧数据包含'全选'，当前选中数据包含全选
      //   if (oldVal.includes("allSelect") && valArr.includes("allSelect")) {
      //     const index = valArr.indexOf("allSelect");
      //     valArr.splice(index, 1); // 排除全选选项
      //     this.form.productCode = valArr;
      //   }
      //   // 旧数据不包含'全选'，当前选中数据不包含'全选'
      //   if (!oldVal.includes("allSelect") && !valArr.includes("allSelect")) {
      //     // 除了全选外 其他全部选中时
      //     if (valArr.length === allIdArr.length - 1) {
      //       this.form.productCode = ["allSelect"].concat(valArr);
      //     }
      //   }
      //   // 数据发生变化时保存数据，作为下次对比的旧数据
      //   this.oldOptions[0] = this.form.productCode;
      // },
      //获取产品
      getPro() {
        getPro(
          store.getters.user.userId
        ).then(res => {
         if (res.status === 'success') {
           this.options = res.data;
         } else {
           this.$notify({
             title: res.msg,
             type: 'warning',
             duration: 2500
           })
         }
          // 全选功能暂时不要
          // const allSelect = {
          //   fundName: "全选",
          //   fundCode: "allSelect"
          // };
          // this.options.unshift(allSelect);
          // this.options.map(item => {
          //   this.all.push(item.fundCode);
          // });
        });
      },
      //获取表格内容提供输入内容显示数据
      getContacts() {
        let params = {
          // currentUserId: 'CSBY01',
          currentUserId: store.getters.user.userId,
          contactName: ""
        };
        getContacts(params).then(res => {
          if (res.status === 'success') {
            this.restaurants = res.data;
          } else {
            this.$notify({
              title: res.msg,
              type: 'warning',
              duration: 2500
            })
          }
        });
      },
      //点击姓名下拉，存在则后面的电话等赋值
      handleSelect(item) {
        this.flag = true;
        this.placeHolderFlag = true
        this.form.contactId = item.contactId;
        this.form.contactMail = item.contactMail;
        this.form.contactMobile = item.contactMobile;
        this.form.contactPhone = item.contactPhone;
        this.rules.contactMobile = [
          { required: false },
        ];
        this.rules.contactMail = [
          { required: false },
        ];
      },
      querySearch(queryString, cb) {
        // let results;
        if (queryString) {
          this.results = this.restaurants.filter(
            v => v.contactName.indexOf(queryString) !== -1
          );
        } else {
          this.results = this.restaurants;
        }
        // 调用 callback 返回建议列表的数据
        cb(this.results);
      },
      //若是姓名是输入的，后面的电话等信息不赋值
      inputChange() {
        var contactName = (rule, value, callback) => {
          if (value.length > 15) {
            callback(new Error("最多输入15个字!"));
          } else {
            callback()
          }
        }
        this.flag = false;
        this.form.contactId = ""
        this.form.contactMail = "";
        this.form.contactMobile = "";
        this.form.contactPhone = "";
        this.rules.contactMobile = [
          { required: true, message: "请输入手机号", trigger: "blur" },
          {
            pattern: /^1[3456789]\d{9}$/,
            message: "请输入正确的11位手机号码!",
            trigger: ["blur", "change"]
          }
        ];
        this.rules.contactName = [
          { required: true, message: "请输入姓名", trigger: ["blur", "change"] },
          { validator: contactName, trigger: "change" }
        ];
        this.rules.contactMail = [
          { required: true, message: "请输入邮箱地址", trigger: "blur" },
          {
            type: "email",
            message: "请输入正确的邮箱地址",
            trigger: ["blur", "change"]
          }
        ]

      },
      cancel() {
        this.$emit('change', false)
        this.resetForm();
      },
      doSubmit() {
        this.$refs["form"].validate(valid => {
          if (valid) {
            if (this.status === "add") {
              this.doAdd();
            } else {
              this.doEdit();
            }
          }
        });
      },
      doAdd() {
        if (this.form.productCode.includes("allSelect")) {
          this.form.productCode.shift();
          this.form.productCode = this.form.productCode.toString();
        } else {
          this.form.productCode = this.form.productCode.toString();
        }
        // this.form.creatorId = 'CSBY01'
        let creatorIdFlag = false
        if (!creatorIdFlag) {
          this.form.creatorId = store.getters.user.userId;
        }
        add(this.form)
          .then(res => {
            if (res.status === 'success') {
              this.$notify({
                title: res.msg,
                type: "success",
                duration: 2500
              });
              this.resetForm();
              this.$emit('change', false)
              if (this.$parent.isParentFlag) {
                this.$parent.searchContactName()
              } else {
                this.$parent.getRoot();
              }
            } else {
              this.$notify({
                title: res.msg,
                type: 'warning',
                duration: 2500
              })
            }
          })
          .catch(err => {
            this.$notify({
              title: "添加失败",
              type: "error",
              duration: 2500
            });
          });
      },

      // 修改展示
      edit() {
        this.form.id = this.data.id
        this.form.contactId = this.data.contactId
        this.form.productCode = this.data.productCode
        this.form.productName = this.data.productName
        this.form.category = this.data.category
        this.form.contactName = this.data.contactName
        this.form.contactPhone = this.data.contactPhone
        this.form.contactMobile = this.data.contactMobile
        this.form.contactMail = this.data.contactMail
        this.rules.productCode = [
          { required: false },
        ];
        this.rules.category = [
          { required: false },
        ];
        if (this.data.contactId !== null) {
          this.flag = true
          this.rules.contactMobile = [
            { required: false },
          ];
          this.rules.contactMail = [
            { required: false },
          ]
        } else {
          this.rules.contactMobile = [
            { required: true, message: "请输入手机号", trigger: "blur" },
            {
              pattern: /^1[3456789]\d{9}$/,
              message: "请输入正确的11位手机号码!",
              trigger: ["blur", "change"]
            }
          ];
          this.rules.contactMail = [
            { required: true, message: "请输入邮箱地址", trigger: "blur" },
            {
              type: "email",
              message: "请输入正确的邮箱地址",
              trigger: ["blur", "change"]
            }
          ]
        }
      },

      //修改保存
      doEdit() {
        // this.form.creatorId = 'CSBY01'
        this.form.creatorId = store.getters.user.userId;
        edit(this.form)
          .then(res => {
            if (res.status === 'success') {
              this.resetForm();
              this.$notify({
                title: res.msg,
                type: "success",
                duration: 2500
              });
              this.$emit('change', false)
              if (this.$parent.isParentFlag) {
                this.$parent.searchContactName();
              } else {
                this.$parent.getRoot();
              }
            } else {
              this.$notify({
                title: res.msg,
                type: 'warning',
                duration: 2500
              })
            }
          })
          .catch(err => {
            this.$notify({
              title: "修改失败",
              type: "error",
              duration: 2500
            });
          });
      },
      resetForm() {
        this.$refs["form"].resetFields();
        this.form.id = "";
        this.form.productCode = "";
        this.form.contactName = "";
        this.form.contactPhone = "";
        this.form.contactMobile = "";
        this.form.contactMail = "";
        this.flag = false;
      },
      closeDialog() {
        this.$emit('change', false)
        this.resetForm()
      }
    }

  };
</script>

<style scoped>
  .demo-autocomplete .inline-input {
    width: 310px;
  }

  /deep/ .el-select__tags {
    width: 400px;
  }

  .el-form-item__label {
    width: 83px;
  }
  /*下拉框多选之后隐藏选中的选项*/
  /deep/ .el-select__tags {
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
    overflow-y: auto;
    overflow-x: hidden;
    max-height: 65px;
    white-space: pre-wrap;
  }

  /*/deep/ .el-tag:nth-child(2) {*/
  /*  display: none;*/
  /*}*/

  /deep/ .el-select__input {
    /*width: 100%;*/
    min-width:200px !important;
    position: relative;
    /*left: -75px;*/
  }

  /deep/ .el-notification {
    width: 500px;
  }

</style>
