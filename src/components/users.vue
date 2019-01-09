<template>
  <el-card class="card">
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 搜索+添加用户 -->
    <el-row>
      <el-col>
        <el-input
          @clear="getAllUsers()"
          clearable
          placeholder="请输入内容"
          v-model="query"
          class="input-with-select"
        >
          <el-button @click="searchUser()" slot="append" icon="el-icon-search"></el-button>
        </el-input>
        <el-button type="primary" @click="showDiaAdd()">添加用户</el-button>
      </el-col>
    </el-row>

    <!-- 表格 -->
    <!-- data属性的值是表格的数据 -->
    <!-- el-table-column代表列，每行的行数是data绑定的数据的长度 -->
    <!-- label表示列的表头文本 -->
    <!-- prop表示每一列中每个单元格的数据 来源于tableData中每个对象的key名 -->
    <el-table class="table" height="200px" :data="tableData" style="width: 100%">
      <el-table-column prop="id" label="#" width="100"></el-table-column>
      <el-table-column prop="username" label="姓名" width="100"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="220"></el-table-column>
      <el-table-column prop="mobile" label="电话" width="120"></el-table-column>
      <el-table-column label="创建日期" width="180">
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
      <el-table-column prop label="操作" width="180">
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
            size="mini"
            @click="showDeleConfig(scope.row)"
            plain
            type="danger"
            icon="el-icon-delete"
            circle
          ></el-button>
          <el-button
            @click="showRoleDia(scope.row)"
            size="mini"
            plain
            type="success"
            icon="el-icon-check"
            circle
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

    <!-- 对话框 -->
    <!-- 添加 -->
    <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
      <el-form :model="form">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" :label-width="formLabelWidth">
          <el-input v-model="form.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="form.email" autocomplete="off"></el-input>
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

    <!-- 编辑 -->
    <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
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

    <!-- 分配角色 -->
    <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRole">
      <el-form>
        <el-form-item label="用户名" :label-width="formLabelWidth">
          {{currUsername}}
        </el-form-item>
        <el-form-item label="角色" :label-width="formLabelWidth">
          {{currRoleId}}
          <el-select v-model="currRoleId">
            <el-option disabled label="请选择" :value="-1"></el-option>
            <!-- <el-option label="测试的" value="100"></el-option> -->
            <el-option v-for="(v,i) in roles" :key="i"

            :label="v.roleName" :value="v.id"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleRole = false">取 消</el-button>
        <el-button type="primary" @click="setRole()">确 定</el-button>
      </div>
    </el-dialog>
  </el-card>
</template>

<script>
// -------------------------------------data-----------------------------------------
export default {
  data () {
    return {
      query: '',
      // 表格数据源
      tableData: [],
      pagenum: 1,
      pagesize: 2,
      total: -1,
      // 控制对话框显示或者隐藏
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      dialogFormVisibleRole: false,
      from: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      formLabelWidth: '100px',
      form: {},
      // select所绑定的数据
      currRoleId: -1,
      roles: [],
      currUsername: '',
      currUserId: -1
    }
  },
  created () {
    this.getTableData()
  },
  // --------------------------------------methods---------------------------------
  methods: {
    // 分配角色-发送请求
    async setRole () {
      // 此处 id 为用户id
      // 请求体{rid:当前修改后的id}

      const res = await this.$http.put(`users/${this.currUserId}/role`, {
        rid: this.currRoleId
      })
      console.log(res)
      this.dialogFormVisibleRole = false
    },

    // 分配角色-打开对话框
    async showRoleDia (user) {
      // user表示用户信息
      // console.log(user);

      this.currUsername = user.username
      this.currUserId = user.id
      this.dialogFormVisibleRole = true

      // 获取所有角色的名字
      const res = await this.$http.get(`roles`)
      // console.log(res);
      const {
        meta: {msg, status},
        data
      } = res.data
      if (status === 200) {
        // 获取data
        this.roles = data
      }
      // console.log(this.roles)
      // 根据用户id查询用户 -->获取当前用户的角色id
      const res2 = await this.$http.get(`users/${user.id}`)
      // console.log(res2);
      this.currRoleId = res2.data.data.rid
    },

    // 改变用户的状态
    async changeState (user) {
      // console.log(user);

      // uId-->用户id
      // type-->修改后的状态
      const res = await this.$http.put(
        `users/${user.id}/state/${user.mg_state}`
      )
      console.log(res)
    },

    // 编辑- 发送请求
    async editUser () {
      const res = await this.$http.put(`users/${this.form.id}`, this.form)
      console.log(res)
      const {
        meta: { msg, status }
      } = res.data
      if (status === 200) {
        // 关闭对话框
        this.dialogFormVisibleEdit = false
        // 刷新表格
        this.getTableData()
      } else {
        this.$message.warning(msg)
      }
    },

    // 编辑-打开对话框
    async showDiaEdit (user) {
      this.dialogFormVisibleEdit = true
      // 获取当前编辑的数据
      const res = await this.$http.get(`users/${user.id}`)
      const {
        meta: {msg, status},
        data
      } = res.data
      if (status === 200) {
        this.form = data
        // console.log(this.form);
      }
    },

    // 删除-打开一个确认框
    showDeleConfig (user) {
      this.$confirm('确定继续删除吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          // async是放在距离其最近的一个函数上
          // 删除-发送请求
          const res = await this.$http.delete(`users/${user.id}`)
          const {
            meta: {msg, status}
          } = res.data
          if (status === 200) {
            this.$message.success('删除成功')
            this.pagenum = 1
            this.getTableData()
          } else {
            this.$message.warning('msg')
          }
        })
        .catch(() => {
          this.$message.info('已取消删除')
        })
    },

    // 添加用户-发送请求
    async addUser () {
      // 1.发送请求
      const res = await this.$http.post(`users`, this.form)
      // console.log(res);
      const {
        meta: { msg, status }
      } = res.data
      if (status === 201) {
        // 2.关闭对话框
        this.dialogFormVisibleAdd = false
        // 3.刷新表格
        this.getTableData()
        // 4.成功提示
      } else {
        this.$message.warning(msg)
      }
    },

    // 添加用户-打开对话框
    showDiaAdd () {
      // 清空
      this.form = {}
      this.dialogFormVisibleAdd = true
    },

    // 输入框内点击X获取所有用户
    getAllUsers () {
      this.getTableData()
    },

    // 搜索用户
    searchUser () {
      this.pagenum = 1
      this.getTableData()
    },

    // 分页相关方法
    handleSizeChange (val) {
      // console.log(`每页 ${val} 条`)
      this.pagenum = 1
      this.pagesize = val
      this.getTableData()
    },
    handleCurrentChange (val) {
      // val是当前页码
      // 每条两页，点击不同页数获取相应的数据
      this.pagenum = val
      this.getTableData()
      // console.log(`当前页: ${val}`)
    },

    // 获取用户列表的数据
    async getTableData () {
      const AUTH_TOKEN = localStorage.getItem('token')
      this.$http.defaults.headers.common['Authorization'] = AUTH_TOKEN

      const res = await this.$http.get(
        `users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${
          this.pagesize
        }`
      )
      // console.log(res);
      const {
        meta: { status, msg },
        data: { total, users }
      } = res.data
      if (status === 200) {
        // 表示获取成功
        this.tableData = users
        // console.log(this.tableData)
        this.total = total
        this.$message.success(msg)
      } else {
        this.$message.warning(msg)
      }
    }
  }
}
</script>

<style>
.card {
  height: 100%;
}

.input-with-select {
  margin-top: 20px;
  width: 350px;
}
</style>
// create_time: 创建时间
// email: 邮箱
// id: ID
// mg_state: 用户状态
// mobile: 电话
// role_name: 角色名
// username: 用户名
