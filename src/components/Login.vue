<template>
  <div class="login-container">
    <div class="login-box">
      <!-- 头像 -->
      <div class="avatar-box">
        <img class="avatar-box-img" src="../assets/logo.png" alt="">
      </div>
      <!-- 登录表单 -->
      <el-form ref="loginFormRef" class="loin-form" :model="loginForm" :rules="LoginRules">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input
            v-model="loginForm.username"
            prefix-icon="iconfont icon-user"
            placeholder="请输入用户名"
          ></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item  prop="password">
          <el-input
            v-model="loginForm.password"
            prefix-icon="iconfont icon-3702mima"
            placeholder="请输入密码"
            type="password"
          ></el-input>
        </el-form-item>
      <!-- 按钮 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLofinForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      LoginRules: {
        username: [
          { required: true, message: '请输入登录名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入登录密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetLofinForm () {
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登录失败！')
        this.$message.success('登录成功！')
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="stylus" scoped>
  .login-container
    height 100%
    background-color #2b4b6b
    .login-box
      width 45rem
      height 30rem
      background #fff
      border-radius .3rem
      position absolute
      left 50%
      top 50%
      transform translate(-50%, -50%)
      .avatar-box
        position relative
        left 50%
        transform translate(-50%, -50%)
        background #fff
        height 13rem
        width 13rem
        border .1rem solid #eeeeee
        border-radius 50%
        padding 1rem
        box-shadow 0 0 1rem #ddd
        img
          width 100%
          height 100%
          border-radius 50%
          background-color #eee
      .loin-form
        position absolute
        bottom 0
        width 100%
        padding 0 2rem
        box-sizing border-box
        .btns
          display flex
          justify-content flex-end
</style>
