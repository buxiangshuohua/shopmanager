<template>
  <el-card class="card">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>活动列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-row>
      <el-col>
        <el-input
          @clear="getAllUsers()"
          clearable
          placeholder="请输入用户名"
          v-model="query"
          class="input-with-select"
        >
          <el-button @click="searchUser()" slot="append" icon="el-icon-search"></el-button>
        </el-input>
        <el-button type="success" plain @click="showDiaAdd()">添加用户</el-button>
      </el-col>
    </el-row>
    <el-table class="table" height="300px" :data="tableData" style="width: 100%">
      <el-table-column prop="id" label="#" width="80"></el-table-column>
      <el-table-column prop="username" label="姓名" width="100"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
      <el-table-column prop="mobile" label="电话" width="120"></el-table-column>

      <el-table-column label="创建日期" width="180">
        <!-- 如果单元格的内容不是prop的值对应的值key对应的值
          此时需要给被显示的内容包裹容器template
          目的是：在里层使用外层tableData组件传值
          slot-scope作用是  承上启下 传递数据
          tableData会自动获取上层数据的值data="tableData"
          在里层通过tableData.row
        -->
        <template slot-scope="scope">{{scope.row.create_time|fmtDate}}</template>
      </el-table-column>

      <el-table-column label="用户状态" width="140">
        <template slot-scope="scope">
          <el-switch
            @change="changeState(scope.row)"
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180">
        <template slot-scope="scope">
          <el-button
            size="mini"
            plain
            type="primary"
            icon="el-icon-edit"
            circle
            @click="showDiaEdit(scope.row)"
          ></el-button>
          <el-button
            @click="showRoleDia(scope.row)"
            size="mini"
            plain
            type="success"
            icon="el-icon-check"
            circle
          ></el-button>
          <el-button
            size="mini"
            plain
            type="danger"
            icon="el-icon-delete"
            circle
            @click="showDeleConfig(scope.row)"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
    <!-- 弹出层 -->
    <!-- 添加 -->
    <el-dialog title="添加收货地址" :visible.sync="dialogFormVisibleAdd">
      <el-form :model="form">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="form.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" :label-width="formLabelWidth">
          <el-input v-model="form.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" :label-width="formLabelWidth">
          <el-input v-model="form.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
        <el-button type="primary" @click="addUser()">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 编辑用户 -->
    <el-dialog title="编辑收货地址" :visible.sync="dialogFormVisibleEdit">
      <el-form :model="form">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input disabled v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="form.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" :label-width="formLabelWidth">
          <el-input v-model="form.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleEdit = false">取 消</el-button>
        <el-button type="primary" @click="editUser()">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 操作用户弹出层 -->
    <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRole">
      <el-form>
        <el-form-item label="用户名" :label-width="formLabelWidth">
          {{currUsername}}
        </el-form-item>
        <el-form-item label="角色" :label-width="formLabelWidth">
          {{currRoleId}}
          <el-select v-model="currRoleId">
            <el-option disabled label="请选择" :value="-1"></el-option>
            <el-option v-for="(v,i) in roles" :key="i" :label="v.roleName" :value="v.id"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleRole = false">取 消</el-button>
        <el-button type="primary" @click="setRose()">确 定</el-button>
      </div>
    </el-dialog>
  </el-card>
</template>

<script>
export default {
  data() {
    return {
      // query: '',
      tableData: [],
      query: "",
      pagenum: 1,
      pagesize: 2,
      total: -1,
      // currentPage4: 2
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      dialogFormVisibleRole: false,  
      form: {
        username: "",
        password: "",
        email: "",
        mobile: ""
      },
      formLabelWidth: "100px",
      currRoleId: -1,
      // 角色数据
      roles: [],
      currUsername: ""
    };
  },
  created() {
    this.getTableData();
  },
  methods: {
    async setRose(){
      const res = await this.$http.put(`users/${this.currUserId}/role`,{
        rid: this.currRoleId
      })
      this.dialogFormVisibleRole= false 
    },
    async showRoleDia(user) {
      // console.log(user.id);
      this.currUsername = user.username
      this.currUserId = user.id 

      this.dialogFormVisibleRole= true
      // 获取所有角色的名字
     
      const res = await this.$http.get(`roles`)
      // console.log(res);
      
      const {
        meta: {msg, status},
        data
      } = res.data
      
      if(status===200) {
        // 获取data
        this.roles = data
      }
      // 获取所有用户id
      const res2 = await this.$http.get(`users/${user.id}`);
      // console.log(res2);
      this.currRoleId = res2.data.data.rid;
      
    },
    async changeState(user) {
      const res = await this.$http.put(
        `users/${user.id}/state/${user.mg_state}`
      );
      // console.log(res);
    },
    async editUser() {
      const res = await this.$http.put(`users/${this.form.id}`, this.form);
      // console.log(res);
      const {
        meta: { msg, status },
      } = res.data;
      if (status === 200) {
        this.dialogFormVisibleEdit = false;
        // 刷新表格
        this.getTableData();
      } else {
        this.$message.warning(msg);
      }
    },
    // 编辑用户功能实现
    async showDiaEdit(user) {
      this.dialogFormVisibleEdit = true;
      const res = await this.$http.get(`users/${user.id}`);
      // console.log(res)
      const {
        meta: { status },
        data
      } = res.data;
      if (status === 200) {
        this.form = data;
      }
    },
    // 删除用户功能功能
    showDeleConfig(user) {
      // console.log(user)

      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          const res = await this.$http.delete(`users/${user.id}`);
          // this.$message.success('删除成功')
          const {
            meta: { msg, status }
          } = res.data;

          if (status === 200) {
            this.$message.success("删除成功");
            this.pagenum = 1;
            this.getTableData();
          } else {
            this.$message.warning(msg);
          }
        })
        .catch(() => {
          this.$message.warning("取消删除");
        });
    },
    // 点击确定添加用户列表
    async addUser() {
      const res = await this.$http.post(`users`, this.form);
      const {
        meta: { msg, status }
      } = res.data;
      if (status === 201) {
        // 关闭对话框
        this.dialogFormVisibleAdd = false;
        // 刷新表格
        this.getTableData();
      } else {
        this.$message.warning(msg);
      }
    },
    // 打开对话框
    showDiaAdd() {
      this.form = {};
      this.dialogFormVisibleAdd = true;
    },
    // 点击X获取所有用户
    getAllUsers() {
      this.getTableData();
    },
    // 搜索功能功能
    searchUser() {
      this.pagenum = 1;
      this.getTableData();
    },
    // 分页相关方法
    handleSizeChange(val) {
      // console.log(`每页 ${val} 条`)
      this.pagenum = 1;
      this.pagesize = val;
      this.getTableData();
    },
    handleCurrentChange(val) {
      this.pagenum = val;
      this.getTableData();
      // console.log(`当前页: ${val}`)
    },
    async getTableData() {
      // 设置请求头
      // 原因是所有接口中 除了登录不需要授权，其他所有的请求都需要授权
      // 设置请求头
      const AUTH_TOKEN = localStorage.getItem("token");
      this.$http.defaults.headers.common["Authorization"] = AUTH_TOKEN;

      const res = await this.$http.get(
        `users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${
          this.pagesize
        }`
      );
      // console.log(res)
      const {
        meta: { status, msg },
        data: { total, users }
      } = res.data;
      if (status === 200) {
        this.tableData = users;
        // console.log(this.tableData);

        this.total = total;
        this.$message.success(msg);
      } else {
        this.$message.warning(msg);
      }
    }
  }
};
</script>

<style>
.card {
  height: 100%;
}
.input-with-select {
  margin-top: 30px;
  width: 350px;
}
</style>
