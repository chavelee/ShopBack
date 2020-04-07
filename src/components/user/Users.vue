<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <div>
        <el-row :gutter="20">
          <el-col :span="9">
            <el-input
              placeholder="请输入内容"
              v-model="QuerryInfo.query"
              clearable
              @clear="getUserList()"
            >
              <el-button
                slot="append"
                icon="el-icon-search"
                @click="getUserList()"
              ></el-button>
            </el-input>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="addDialogVisible = true"
              >添加用户</el-button
            >
          </el-col>
        </el-row>
        <el-table border stripe :data="userList">
          <el-table-column label="姓名" prop="username"></el-table-column>
          <el-table-column label="邮箱" prop="email"></el-table-column>
          <el-table-column label="电话" prop="mobile"></el-table-column>
          <el-table-column label="角色" prop="role_name"></el-table-column>
          <el-table-column label="状态">
            <template slot-scope="scope">
              <el-switch
                style="display: block"
                v-model="scope.row.mg_state"
                @change="changeUserState(scope.row)"
              ></el-switch>
            </template>
          </el-table-column>
          <el-table-column width="180px" label="操作">
            <template slot-scope="scope">
              <el-tooltip
                :enterable="false"
                effect="dark"
                content="编辑角色"
                placement="top-start"
              >
                <el-button
                  :enterable="false"
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="editDialog(scope.row.id)"
                ></el-button>
              </el-tooltip>
              <el-tooltip
                :enterable="false"
                effect="dark"
                content="删除角色"
                placement="top-start"
              >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="DeleteUserById(scope.row.id)"
                ></el-button>
              </el-tooltip>
              <el-tooltip
                :enterable="false"
                effect="dark"
                content="分配角色"
                placement="top-start"
              >
                <el-button
                  type="warning"
                  icon="el-icon-setting"
                  size="mini"
                  @click="setRoleDialogShow(scope.row)"
                ></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
        <div class="pagination">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="QuerryInfo.pagenum"
            :page-sizes="[1, 2, 5, 10]"
            :page-size="QuerryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total"
          ></el-pagination>
        </div>
      </div>
    </el-card>
    <!-- 添加用户对话框 -->
    <div class="addUserDialog">
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        @close="addDialogClosed"
        width="50%"
      >
        <div class="addUser">
          <el-form
            :model="adduserInfo"
            :rules="addUserRules"
            ref="addUserRef"
            label-width="70px"
            class="addUserForm"
          >
            <el-form-item label="用户名" prop="username">
              <el-input
                v-model="adduserInfo.username"
                placeholder="请输入用户名"
              ></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
              <el-input
                v-model="adduserInfo.password"
                placeholder="请输入密码"
                type="password"
              ></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input
                v-model="adduserInfo.email"
                placeholder="请输入邮箱地址"
              ></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
              <el-input
                v-model="adduserInfo.mobile"
                placeholder="请输入手机号码"
              ></el-input>
            </el-form-item>
          </el-form>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="HandleAddUser">确 定</el-button>
        </span>
      </el-dialog>
    </div>
    <!-- 修改用户 -->
    <div class="editUserDialog">
      <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        @close="editDialogCLosed"
        width="50%">
        <div class="editUser">
          <el-form
            label-width="70px"
            ref="editUserRef"
            :model="editForm"
            :rules="editRules">
            <el-form-item label="用户名">
              <el-input disabled v-model="editForm.username"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input v-model="editForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
              <el-input v-model="editForm.mobile"></el-input>
            </el-form-item>
          </el-form>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="handleEditUser()">确 定</el-button>
        </span>
      </el-dialog>
    </div>
    <!-- 分配角色对话框 -->
    <el-dialog
      title="分配角色"
      :visible.sync="setRoleDialogVisible"
      width="50%"
      @close="setRoleDialogVisibleClosed"
    >
      <div>
        <p>当前用户: {{userInfo.username}}</p>
        <p>当前角色: {{userInfo.role_name}}</p>
        <p>
          分配新角色:
          <el-select
            v-model="selectRoleId"
            placeholder="请选择">
            <el-option
              v-for="item in roleList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id">
            </el-option>
          </el-select>
          </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setUserRole()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Users',
  data () {
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^([A-Za-z0-9_\-.\u4e00-\u9fa5])+@([A-Za-z0-9_\-.])+.([A-Za-z]{2,8})$/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))
    }
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1[3|4|5|7|8]\d{9}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号码'))
    }
    return {
      QuerryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userList: [],
      total: 0,
      // 添加用户
      addDialogVisible: false,
      adduserInfo: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addUserRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 2, max: 8, message: '长度在 2 到 8 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 18, message: '长度在 6 到 18 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 编辑用户
      editDialogVisible: false,
      editForm: {},
      editRules: {
        email: [
          { required: true, message: '请输入用户邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入用户手机', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 删除用户
      deleteDialogVisible: false,
      deletUserId: {
        id: ''
      },
      // 分配角色
      setRoleDialogVisible: false,
      userInfo: {},
      roleList: [],
      selectRoleId: ''
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    // 获取用户列表
    async getUserList () {
      const { data: res } = await this.$http.get('users', {
        params: this.QuerryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败')
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 换页
    handleSizeChange (newSize) {
      this.QuerryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange (newPage) {
      this.QuerryInfo.pagenum = newPage
      this.getUserList()
    },
    // 修改用户状态
    async changeUserState (userinfo) {
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      )
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('设置状态失败')
      }
      this.$message.success('设置状态成功')
    },
    // 添加用户
    HandleAddUser () {
      this.$refs.addUserRef.validate(async vaild => {
        if (!vaild) return
        const { data: res } = await this.$http.post('users', this.adduserInfo)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败!')
        }
        this.$message.success('添加用户成功!')
        this.addDialogVisible = false
        this.getUserList()
      })
    },
    addDialogClosed () {
      this.$refs.addUserRef.resetFields()
    },
    // 查询用户信息
    async editDialog (id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogCLosed () {
      this.$refs.editUserRef.resetFields()
    },
    // 编辑用户
    async handleEditUser () {
      this.$refs.editUserRef.validate(async vaild => {
        if (!vaild) return
        const { data: res } = await this.$http.put('users/' + this.editForm.id, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户信息失败')
        }
        this.$message.success('修改用户信息成功')
        this.getUserList()
        this.editDialogVisible = false
      })
    },
    // 删除用户
    async DeleteUserById (id) {
      this.deletUserId.id = id
      const deleteResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回字符串 "confirm"
      // 如果用户取消删除，则返回字符串 "cancle"
      // console.log(deleteResult)
      if (deleteResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + this.deletUserId.id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success(res.meta.msg)
      this.getUserList()
    },
    // 分配角色
    async setRoleDialogShow (user) {
      this.userInfo = user
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.roleList = res.data
      this.setRoleDialogVisible = true
      console.log(this.userInfo)
    },
    async setUserRole () {
      await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectRoleId })
        .then((res) => {
          if (res.data.meta.status !== 200) {
            return this.$message.error(res.data.meta.msg)
          }
          this.getUserList()
          this.setRoleDialogVisible = false
          this.$message.success('修改角色成功')
        })
    },
    setRoleDialogVisibleClosed () {
      this.userInfo = {}
      this.selectRoleId = ''
    }
  }
}
</script>

<style lang="stylus"></style>
