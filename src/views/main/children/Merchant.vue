<template>
  <div class="content" v-loading.fullscreen.lock="loading">
    <header class="header">
      <!-- <h3>商户账户管理</h3> -->
      <el-form inline :model="searchForm">
        <el-form-item label="商户编码">
          <el-input v-model="searchForm.merchant_code" clearable placeholder="请输入商户编码"></el-input>
        </el-form-item>
        <el-form-item label="商户名称">
          <el-input v-model="searchForm.merchant_name" clearable placeholder="请输入商户名称"></el-input>
        </el-form-item>
        <el-form-item label="联系人">
          <el-input v-model="searchForm.link_person" clearable placeholder="请输入联系人"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">查询</el-button>
        </el-form-item>
      </el-form>
      <el-button-group>
        <el-button type="primary" icon="el-icon-plus" @click="newAction()">新增</el-button>
        <el-button type="primary" icon="el-icon-delete" @click="delAction()">删除</el-button>
      </el-button-group>
    </header>
    <!-- 表格部分 -->
    <el-table :data="tableData" border style="width: 100%" @selection-change="selectionChange">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column prop="merchant_code" label="商户号" width="120">
      </el-table-column>
      <el-table-column prop="merchant_name" label="商户名称" width="160">
      </el-table-column>
      <el-table-column prop="link_person" label="联系人" width="120">
      </el-table-column>
      <el-table-column prop="link_phone" label="联系电话" width="160">
      </el-table-column>
      <!-- <el-table-column prop="address" label="地址">
      </el-table-column> -->
      <el-table-column prop="remark" label="备注说明">
      </el-table-column>
      <el-table-column prop="user_create" label="创建人" width="120">
      </el-table-column>
      <el-table-column prop="time_update" label="更新日期" width="200">
      </el-table-column>
      <el-table-column prop="user_create" label="更新操作人" width="120">
      </el-table-column>
      <el-table-column prop="is_enabled" label="是否启用" width="120">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.is_enabled" active-color="#13ce66" inactive-color="#ff4949"
            @change="changeSwich(scope.row)">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="160">
        <template slot-scope="scope">
          <el-button type="text" @click="editAction(scope.row.merchant_code)">编辑</el-button>
          <el-button type="text" @click="keyAction(scope.row)">有效信息</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="footer">
      <el-pagination @size-change="sizeChange" @current-change="currentChange" :current-page.sync="searchForm.page"
        :page-sizes="[15, 30, 50, 100]" :page-size="searchForm.rows" layout="sizes, prev, pager, next"
        :total="searchForm.total">
      </el-pagination>
    </div>
    <!-- 编辑部分 -->
    <el-dialog :title="editTxt" :visible.sync="showEdit" :close-on-click-modal="false">
      <el-form :model="form" label-width="80px" ref="editForm" :rules="rules" hide-required-asterisk>
        <el-form-item label="商户号" prop="merchant_code">
          <el-input v-model="form.merchant_code" autocomplete="off" readonly>
            <el-button slot="append" type="primary" v-if="isNew" @click="reloadCode">重新获取</el-button>
          </el-input>
        </el-form-item>
        <el-form-item label="商户名称" prop="merchant_name">
          <el-input v-model="form.merchant_name" autocomplete="off" maxlength="20" placeholder="请输入商户名称"></el-input>
        </el-form-item>
        <el-form-item label="联系人" prop="link_person">
          <el-input v-model="form.link_person" autocomplete="off" maxlength="20" placeholder="请输入联系人"></el-input>
        </el-form-item>
        <el-form-item label="联系电话" prop="link_phone">
          <el-input v-model="form.link_phone" autocomplete="off" maxlength="20" placeholder="请输入联系电话"></el-input>
        </el-form-item>
        <el-form-item label="地址" prop="address">
          <el-input v-model="form.address" autocomplete="off" maxlength="100" placeholder="请输入联系地址"></el-input>
        </el-form-item>
        <el-form-item label="有效日期">
          <el-col :span="11">
            <el-form-item prop="valid_st">
              <el-date-picker type="date" placeholder="请选择生效日期" :readonly="!isNew" format="yyyy-MM-dd"
                v-model="form.valid_st" value-format="yyyy-MM-dd" style="width: 100%;">
              </el-date-picker>
            </el-form-item>
          </el-col>
          <el-col style="text-align:center" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item prop="valid_et">
              <el-date-picker type="date" placeholder="请选择失效日期" :readonly="!isNew" format="yyyy-MM-dd"
                v-model="form.valid_et" value-format="yyyy-MM-dd" style="width: 100%;">
              </el-date-picker>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="备注">
          <el-input type="textarea" autocomplete="off" :rows="2" maxlength="100" placeholder="请输入说明内容"
            v-model="form.remark">
          </el-input>
        </el-form-item>
        <el-form-item label="是否启用">
          <el-radio-group v-model="form.is_enabled">
            <el-radio :label="true" :disabled="!isNew">启用</el-radio>
            <el-radio :label="false" :disabled="!isNew">禁用</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancalAction('editForm')">取 消</el-button>
        <el-button type="primary" @click="confrimAction('editForm')">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 密码编辑部分 -->
    <el-dialog title="有效信息" :visible.sync="showKey" :close-on-click-modal="false" width="500px">
      <el-form :model="keyForm" label-width="80px" label-position="left" ref="keyForm" hide-required-asterisk>
        <el-form-item label="秘钥" prop="secret_key">
          <el-input v-model="keyForm.secret_key" autocomplete="off" readonly>
            <el-button slot="append" type="primary" @click="reloadKey">密钥变更</el-button>
          </el-input>
        </el-form-item>
        <el-form-item label="有效日期">
          <el-col :span="11">
            <el-form-item prop="valid_st">
              <el-date-picker type="date" placeholder="请选择生效日期" format="yyyy-MM-dd" v-model="keyForm.valid_st"
                value-format="yyyy-MM-dd" style="width: 100%;">
              </el-date-picker>
            </el-form-item>
          </el-col>
          <el-col style="text-align:center" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item prop="valid_et">
              <el-date-picker type="date" placeholder="请选择失效日期" format="yyyy-MM-dd" v-model="keyForm.valid_et"
                value-format="yyyy-MM-dd" style="width: 100%;">
              </el-date-picker>
            </el-form-item>
          </el-col>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="showKey=false">取 消</el-button>
        <el-button type="primary" @click="confrimKeyAction('keyForm')">提 交</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "merchant",
  data() {
    return {
      loading: false,
      searchForm: {
        merchant_code: "",
        merchant_name: "",
        link_person: "",
        page: 1,
        rows: 15,
        total: 30
      },
      tableData: [],
      multipleSelection: [],
      showEdit: false,
      rules: {
        merchant_name: [
          { required: true, message: "请输入商户名称", trigger: "blur" }
        ],
        link_person: [
          { required: true, message: "请输入联系人", trigger: "blur" }
        ],
        link_phone: [
          { required: true, message: "请输入联系电话", trigger: "blur" },
          {
            pattern: /^[1][3,4,5,7,8,9][0-9]{9}$/,
            message: "联系电话格式不正确",
            trigger: "blur"
          }
        ],
        address: [{ required: true, message: "请输入地址", trigger: "blur" }],
        valid_st: [
          {
            required: true,
            message: "请选择生效日期",
            trigger: "change"
          }
        ],
        valid_et: [
          {
            required: true,
            message: "请选择失效日期",
            trigger: "change"
          }
        ]
      },
      form: {
        type: 1,
        merchant_code: "",
        merchant_name: "",
        link_person: "",
        link_phone: "",
        address: "",
        remark: "",
        valid_st: "",
        valid_et: "",
        is_enabled: true
      },
      editTxt: "",
      isNew: true,
      showKey: false,
      keyForm: {
        merchant_code: "",
        secret_key: "",
        valid_st: "",
        valid_et: ""
      }
    };
  },
  created() {
    this.getlist();
  },
  methods: {
    // 获取信息
    getlist() {
      this.loading = true;
      this.$ajax
        .get("/api/merchant/", this.searchForm)
        .then(res => {
          this.loading = false;
          this.searchForm.total = res.rsp_data.total || 0;
          this.tableData = res.rsp_data.data_detail || [];
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 查询
    onSubmit() {
      this.searchForm.page = 1;
      this.getlist();
    },
    // 新增操作
    newAction() {
      this.editTxt = "新增商户";
      this.showEdit = true;
      this.isNew = true;
      this.form.type = 1;
      this.getCode();
    },
    // 新增获取商户号
    getCode() {
      this.loading = true;
      this.$ajax
        .get("/api/merchant/option/1")
        .then(res => {
          this.loading = false;
          this.form.merchant_code = res.rsp_data;
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 重新获取商户号
    reloadCode() {
      this.getCode();
    },

    // 查看密钥
    keyAction(row) {
      this.keyForm.secret_key = row.secret_key;
      this.keyForm.valid_st = row.valid_st;
      this.keyForm.valid_et = row.valid_et;
      this.keyForm.merchant_code = row.merchant_code;
      this.showKey = true;
    },
    // 重新获取密钥
    reloadKey() {
      this.loading = true;
      this.$ajax
        .post("/api/merchant/option/2", {
          merchant_code: this.keyForm.merchant_code
        })
        .then(res => {
          this.loading = false;
          if (res.rsp_code === 200) {
            this.$message({ type: "success", message: res.rsp_msg });
            this.keyForm.secret_key = res.rsp_data;
            this.getlist();
          } else {
            this.$message({ type: "error", message: res.rsp_msg });
          }
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 确认信息修改提交
    confrimKeyAction(formName) {
      this.loading = true;
      this.$ajax
        .post("/api/merchant/option/3", {
          merchant_code: this.keyForm.merchant_code,
          valid_st: this.keyForm.valid_st,
          valid_et: this.keyForm.valid_et
        })
        .then(res => {
          this.loading = false;
          if (res.rsp_code === 200) {
            this.$message({ type: "success", message: res.rsp_msg });
            this.showKey = false;
            this.getlist();
          } else {
            this.$message({ type: "error", message: res.rsp_msg });
          }
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 编辑操作---弹窗赋值
    editAction(merchant_code) {
      this.editTxt = "编辑商户";
      this.showEdit = true;
      this.isNew = false;
      this.loading = true;
      this.$ajax
        .get(`/api/merchant/${merchant_code}`)
        .then(res => {
          this.loading = false;
          if (res.rsp_code === 200) {
            this.form = {
              type: 2,
              merchant_code: res.rsp_data.merchant_code,
              merchant_name: res.rsp_data.merchant_name,
              link_person: res.rsp_data.link_person,
              link_phone: res.rsp_data.link_phone,
              address: res.rsp_data.address,
              remark: res.rsp_data.remark,
              valid_st: res.rsp_data.valid_st,
              valid_et: res.rsp_data.valid_et,
              is_enabled: res.rsp_data.is_enabled
            };
          } else {
            this.$message({ type: "error", message: res.rsp_msg });
          }
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 更改单行状态
    changeSwich(row) {
      this.loading = true;
      this.$ajax
        .patch("/api/merchant/", {
          update_type: row.is_enabled ? 1 : 2,
          code_list: row.merchant_code
        })
        .then(res => {
          this.loading = false;
          if (res.rsp_code === 200) {
            this.$message({ type: "success", message: res.rsp_msg });
          } else {
            this.$message({ type: "error", message: res.rsp_msg });
          }
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 删除
    delAction() {
      if (this.multipleSelection.length < 1) {
        this.$message({ type: "warning", message: "请选择需要操作的数据" });
      } else {
        let ids = this.multipleSelection
          .map(el => {
            return el.merchant_code;
          })
          .join("|");
        this.loading = true;
        this.$ajax
          .patch("/api/merchant/", { update_type: 3, code_list: ids })
          .then(res => {
            this.loading = false;
            if (res.rsp_code === 200) {
              this.getlist();
              this.$message({ type: "success", message: res.rsp_msg });
            } else {
              this.$message({ type: "error", message: res.rsp_msg });
            }
          })
          .catch(err => {
            this.loading = false;
          });
      }
    },
    // 取消提交---重置参数
    cancalAction(formName) {
      this.$refs[formName].resetFields();
      this.showEdit = false;
      Object.keys(this.form).forEach(el => {
        this.form[el] = "";
      });
      this.form.is_enabled = true;
    },

    // 确认提交操作执行
    confrimAction(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          if (this.form.type === 2) {
            this.editPost(formName);
          } else {
            this.newPost(formName);
          }
        } else {
          return false;
        }
      });
    },
    // 新增提交方法
    newPost(formName) {
      this.loading = true;
      this.$ajax
        .post("/api/merchant/", this.form)
        .then(res => {
          this.loading = false;
          if (res.rsp_code === 200) {
            this.$message({ type: "success", message: res.rsp_msg });
            this.$refs[formName].resetFields();
            this.showEdit = false;
            this.getlist();
          } else {
            this.$message({ type: "error", message: res.rsp_msg });
          }
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 编辑提交方法
    editPost(formName) {
      this.loading = true;
      this.$ajax
        .put(`/api/merchant/${this.form.merchant_code}`, this.form)
        .then(res => {
          this.loading = false;
          if (res.rsp_code === 200) {
            this.$message({ type: "success", message: res.rsp_msg });
            this.$refs[formName].resetFields();
            this.showEdit = false;
            this.getlist();
          } else {
            this.$message({ type: "error", message: res.rsp_msg });
          }
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 页码大小
    sizeChange(val) {
      this.searchForm.rows = val;
      this.searchForm.page = 1;
      this.getlist();
    },
    // 翻页
    currentChange(val) {
      this.searchForm.page = val;
      this.getlist();
    },
    // 选择表格数据
    selectionChange(val) {
      this.multipleSelection = val;
    }
  }
};
</script>
<style lang="less" scoped>
.header {
  height: 80px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  .el-form-item {
    margin-bottom: 0;
  }
}
.el-table {
  height: calc(100% - 160px);
  overflow-y: auto;
}
.el-pagination {
  margin-top: 10px;
}
</style>
