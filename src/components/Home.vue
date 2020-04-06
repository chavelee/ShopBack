<template>
  <div class="home">
    <el-container class="home-container">
      <el-header>
        <div class="header-left">
          <img class="header-left-logo" src="../assets/heima.png" alt="电商后台管理系统">
          <span class="header-left-title">电商后台管理系统</span>
        </div>
        <el-button @click='logout' type="info">退出</el-button>
      </el-header>
      <el-container>
        <el-aside :width="isCollapse ? '6.5rem' : '20rem'">
          <div class="toggle-button" @click="toggleCollapse">|||</div>
          <el-menu
            :unique-opened="false"
            background-color="#333744"
            text-color="#fff"
            :collapse="isCollapse"
            :collapse-transition="false"
            active-text-color="#409EFF"
            :router="true"
            :default-active="activePath"
          >
            <el-submenu :index="item.id + '' " v-for="item in menuList" :key="item.id">
              <template slot="title">
                <i :class= "iconsList[item.id]"></i>
                <span>{{item.authName}}</span>
              </template>
              <el-menu-item :index="i.path" v-for="i of item.children" :key="i.id" @click="saveNavState(i.path)">
                <template slot="title">
                  <i class="el-icon-menu"></i>
                  <span>{{i.authName}}</span>
                </template>
              </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>
        <!-- 右侧内容 -->
        <el-container>
          <el-main>
            <router-view></router-view>
          </el-main>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      menuList: [],
      iconsList: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    saveNavState (path) {
      window.sessionStorage.setItem('activePath', path)
      this.activePath = path
    }
  }
}
</script>

<style lang="stylus" scoped>
.home
  height 100%
  .home-container
    height 100%
    .el-header
      background-color #373d41
      color #fff
      display flex
      justify-content space-between
      padding-left 0
      align-items center
      font-size 2rem
      .header-left
        display flex
        align-items center
        .header-left-title
          padding-left 1.5rem
    .el-aside
      background-color #333744
      .toggle-button
        background-color #4A5064
        font-size 1rem
        line-height 2.4rem
        color #ffffff
        text-align center
        letter-spacing .2em
        cursor pointer
      ul.el-menu
        border-right 0
        .iconfont
          margin-right 1rem
    .el-main
      background-color #eaedf1
</style>
