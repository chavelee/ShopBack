<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加角色 -->
      <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
      <!-- 角色列表 -->
      <el-table :data="rolesUserList" border stripe width="100%">
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['borderBottom', i1 === 0 ? 'borderTop' : '', 'alignCenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <el-col :span="5">
                <el-tag
                  closable
                  type="primary"
                  @close='removeRightsById(scope.row, item1.id)'
                >
                  {{item1.authName}}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  :class="[i2 > 0 ? 'borderTop' : '', 'alignCenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      closable
                      type="success"
                      @close='removeRightsById(scope.row, item2.id)'>
                      {{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      type="warning"
                      @close='removeRightsById(scope.row, item3.id)'
                    >
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#">
        </el-table-column>
        <el-table-column label="角色名称" prop="roleName">
        </el-table-column>
        <el-table-column label="角色描述" prop="roleDesc">
        </el-table-column>
        <!-- 角色操作 -->
        <el-table-column label="操作" width="300">
          <template slot-scope="scope">
            <el-button
              :enterable="false"
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="editDialog(scope.row.id)"
            >
              编辑
            </el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="DeleteUserById(scope.row.id)"
            >
              删除
            </el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="ShowSetRightDialog(scope.row)"
            >
              分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    <!-- 添加角色表单 -->
      <div class="addRoleDialog">
        <el-dialog
          title="添加角色"
          :visible.sync="addDialogVisible"
          @close="addRoleDialogClosed"
          width="60%"
        >
          <div class="addRole">
            <el-form
              :model="addRoleInfo"
              :rules="addRoleRules"
              ref="addRoleRef"
              label-width="100px"
              class="addUserForm"
            >
              <el-form-item label="角色名称" prop="roleName">
                <el-input
                  v-model="addRoleInfo.roleName"
                  placeholder="请输入角色名称"
                ></el-input>
              </el-form-item>
              <el-form-item label="角色描述" prop="roleDesc">
                <el-input
                  v-model="addRoleInfo.roleDesc"
                  placeholder="角色描述"
                ></el-input>
              </el-form-item>
            </el-form>
          </div>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="HandleAddRole">确 定</el-button>
          </span>
        </el-dialog>
      </div>
      <!-- 编辑角色表单 -->
      <div class="editRoleDialog">
        <el-dialog
          title="编辑角色"
          width="60%"
          :visible.sync="editDialogVisible"
          @close="editDialogClosed"
        >
          <el-form
            :model="editRoleInfo"
            :rules="editRoleRules"
            ref="editRoleRef"
            label-width="100px"
          >
            <el-form-item
              label="角色名称"
              prop="roleName"
            >
              <el-input v-model="editRoleInfo.roleName"></el-input>
            </el-form-item>
            <el-form-item
              label="角色描述"
              prop="roleDesc"
            >
              <el-input v-model="editRoleInfo.roleDesc"></el-input>
            </el-form-item>
          </el-form>
           <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="HandleEditRole">确 定</el-button>
          </span>
        </el-dialog>
      </div>
      <!-- 分配权限 -->
      <div class="setRoleDailog">
        <el-dialog
          title="分配权限"
          width="60%"
          :visible.sync="setDialogVisible"
          @close="setDialogClosed"
        >
        <el-tree
          :data="rightsList"
          :props="treeProps"
          show-checkbox
          node-key="id"
          ref="treeRef"
          default-expand-all
          check-on-click-node
          :default-checked-keys="defKeys"
        >
        </el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="HandleSetRole">确 定</el-button>
          </span>
        </el-dialog>
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Roles',
  data () {
    return {
      rolesUserList: [],
      //  添加角色
      addDialogVisible: false,
      addRoleInfo: {
        roleName: '',
        roleDesc: ''
      },
      addRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 8, message: '长度在 2 到 8 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 2, max: 18, message: '长度在 2 到 18 个字符', trigger: 'blur' }
        ]
      },
      //  编辑角色
      editDialogVisible: false,
      editRoleInfo: {},
      editRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 8, message: '长度在 2 到 8 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 2, max: 18, message: '长度在 2 到 18 个字符', trigger: 'blur' }
        ]
      },
      // 分配权限
      setDialogVisible: false,
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中节点ID
      defKeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
    console.log('created')
  },
  methods: {
    // 获取角色列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesUserList = res.data
    },
    // 添加角色
    HandleAddRole () {
      this.$refs.addRoleRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRoleInfo)
        if (res.meta.status !== 201) {
          this.$message.error(res.meta.msg)
        }
        this.$message.success(res.meta.msg)
        this.addDialogVisible = false
        this.getRolesList()
      })
      console.log(this.addRoleInfo)
    },
    addRoleDialogClosed () {
      this.$refs.addRoleRef.resetFields()
    },
    // 根据ID删除权限
    async removeRightsById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('删除取消')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      this.$message.success('删除权限成功！')
      role.children = res.data
    },
    // 根据角色ID查询角色
    async editDialog (id) {
      console.log(id)
      this.editDialogVisible = true
      await this.$http.get('roles/' + id).then((res) => {
        const result = res.data
        this.editRoleInfo = result.data
        console.log(this.editRoleInfo)
      })
    },
    editDialogClosed () {
      this.$refs.editRoleRef.resetFields()
    },
    HandleEditRole () {
      this.$refs.editRoleRef.validate(async valid => {
        if (!valid) return
        await this.$http.put('roles/' + this.editRoleInfo.roleId, { roleName: this.editRoleInfo.roleName, roleDesc: this.editRoleInfo.roleDesc })
          .then((res) => {
            const result = res.data
            console.log(this.editRoleInfo)
            if (result.meta.status !== 200) {
              return this.$message.error('修改失败！')
            }
            this.$message.success('编辑成功')
            this.getRolesList()
            this.editDialogVisible = false
          })
      })
    },
    // 删除角色
    DeleteUserById (id) {
      this.$http.delete('roles/' + id)
        .then((res) => {
          const result = res.data
          if (result.meta.status !== 200) {
            return this.$message.error('删除失败！')
          }
          this.$message.success('删除成功！')
          this.getRolesList()
        })
    },
    // 分配权限
    async ShowSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取列表失败！')
      }
      this.rightsList = res.data
      this.getLeafKeys(role, this.defKeys)
      this.setDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到defkeys数组中
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    async HandleSetRole () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('修改权限失败')
      }
      this.$message.success('修改权限成功')
      this.getRolesList()
      this.setDialogVisible = false
    },
    setDialogClosed () {
      this.defKeys = []
    }
  }
}
</script>

<style lang="stylus" scoped>
  .borderTop
    border-top 1px solid #eee
  .borderBottom
    border-bottom 1px solid #eee
  .alignCenter
    display flex
    align-items center
  .el-card
    .el-tag
      margin .7rem
</style>
