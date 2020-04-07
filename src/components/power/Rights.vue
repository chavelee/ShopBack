<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="rightsTable">
      <el-table
        :data="rightsTableData"
        stripe
        border>
        <el-table-column
          label="#"
          type="index">
        </el-table-column>
        <el-table-column
          prop="authName"
          label="权限名称">
        </el-table-column>
        <el-table-column
          prop="path"
          label="路径">
        </el-table-column>
        <el-table-column
          prop="level"
          label="权限等级"
        >
          <template slot-scope="scope">
            <el-tag disable-transitions type="primary" v-if="scope.row.level === '0'">
              一级
            </el-tag>
            <el-tag disable-transitions type="success" v-else-if="scope.row.level === '1'">
              二级
            </el-tag>
            <el-tag disable-transitions type="warning" v-else>
              三级
            </el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Power',
  data () {
    return {
      rightsTableData: []
    }
  },
  created () {
    this.getRightsList()
  },
  methods: {
    // 获取权限地址
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败')
      }
      this.rightsTableData = res.data
      console.log(res)
    }
  }
}
</script>

<style lang="stylus" scoped>
  .el-table--striped .el-table__body tr.el-table__row--striped td
    background #e8e8e8
</style>
