<template>
    <div class="login-container">
        <div class="login-top">
            <img src="~@/assets/title_img.png" alt="">
            <!--<span>吉科软数据同步平台\brCTSyncDATA</span>-->

        </div>
        <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form" autocomplete="on"
                 label-position="left">
            <div class="title-container">用户登录</div>
            <el-form-item prop="username">
                <span class="svg-container">
                  <svg-icon icon-class="user"/>
                </span>
                <el-input
                        ref="username"
                        v-model="loginForm.username"
                        placeholder="请输入用户名"
                        name="username"
                        type="text"
                        tabindex="1"
                        autocomplete="on"
                />
            </el-form-item>
            <el-form-item prop="password">
                <span class="svg-container">
                  <svg-icon icon-class="password"/>
                </span>
                <el-input
                        :key="passwordType"
                        ref="password"
                        v-model="loginForm.password"
                        :type="passwordType"
                        placeholder="请输入密码"
                        name="password"
                        tabindex="2"
                        autocomplete="on"
                        @keyup.native="checkCapslock"
                        @blur="capsTooltip = false"
                        @keyup.enter.native="handleLogin"
                />
                <span class="show-pwd" @click="showPwd">
                <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'"/>
              </span>
            </el-form-item>
            <div class="login-btn-box">
                <el-button :loading="loading" type="primary" class="login-btn" @click.native.prevent="handleLogin">登 录</el-button>
                <!--<el-button type="primary" class="login-btn2" @click="register()">取 消</el-button>-->
            </div>
        </el-form>
    </div>
</template>

<script>
  // import { validUsername } from '@/utils/validate'
  import SocialSign from './components/SocialSignin'

  export default {
    name: 'Login',
    components: { SocialSign },
    data() {
      // const validateUsername = (rule, value, callback) => {
      //   if (!validUsername(value)) {
      //     callback(new Error('Please enter the correct user name'))
      //   } else {
      //     callback()
      //   }
      // }
      const validatePassword = (rule, value, callback) => {
        if (value.length < 6) {
          callback(new Error('密码长度不能小于6'))
        } else {
          callback()
        }
      }
      return {
        loginForm: {
          username: '',
          password: ''
        },
        loginRules: {
          // username: [{ required: true, trigger: 'blur', validator: validateUsername }],
          password: [{ required: true, trigger: 'blur', validator: validatePassword }]
        },
        passwordType: 'password',
        capsTooltip: false,
        loading: false,
        showDialog: false,
        redirect: undefined,
        otherQuery: {}
      }
    },
    watch: {
      $route: {
        handler: function(route) {
          const query = route.query
          if (query) {
            this.redirect = query.redirect
            this.otherQuery = this.getOtherQuery(query)
          }
        },
        immediate: true
      }
    },
    created() {
      // window.addEventListener('storage', this.afterQRScan)
    },
    mounted() {
      if (this.loginForm.username === '') {
        this.$refs.username.focus()
      } else if (this.loginForm.password === '') {
        this.$refs.password.focus()
      }
    },
    destroyed() {
      // window.removeEventListener('storage', this.afterQRScan)
    },
    methods: {
      checkCapslock({ shiftKey, key } = {}) {
        if (key && key.length === 1) {
          if (shiftKey && (key >= 'a' && key <= 'z') || !shiftKey && (key >= 'A' && key <= 'Z')) {
            this.capsTooltip = true
          } else {
            this.capsTooltip = false
          }
        }
        if (key === 'CapsLock' && this.capsTooltip === true) {
          this.capsTooltip = false
        }
      },
      showPwd() {
        if (this.passwordType === 'password') {
          this.passwordType = ''
        } else {
          this.passwordType = 'password'
        }
        this.$nextTick(() => {
          this.$refs.password.focus()
        })
      },
      handleLogin() {
        this.$refs.loginForm.validate(valid => {
          if (valid) {
            this.loading = true
            this.$store.dispatch('user/login', this.loginForm)
              .then(() => {
                this.$router.push({ path: this.redirect || '/', query: this.otherQuery })
                this.loading = false
              })
              .catch(() => {
                this.loading = false
              })
          } else {
            console.log('error submit!!')
            return false
          }
        })
      },
      getOtherQuery(query) {
        return Object.keys(query).reduce((acc, cur) => {
          if (cur !== 'redirect') {
            acc[cur] = query[cur]
          }
          return acc
        }, {})
      }
      // afterQRScan() {
      //   if (e.key === 'x-admin-oauth-code') {
      //     const code = getQueryObject(e.newValue)
      //     const codeMap = {
      //       wechat: 'code',
      //       tencent: 'code'
      //     }
      //     const type = codeMap[this.auth_type]
      //     const codeName = code[type]
      //     if (codeName) {
      //       this.$store.dispatch('LoginByThirdparty', codeName).then(() => {
      //         this.$router.push({ path: this.redirect || '/' })
      //       })
      //     } else {
      //       alert('第三方登录失败')
      //     }
      //   }
      // }
    }
  }
</script>

<style lang="scss">
  /* 修复input 背景不协调 和光标变色 */
  /* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */
  /*
  $bg: #283443;
  $cursor: #fff;
  $dark_gray: #889aa4;
  $light_gray: #eee;
  @supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
    .login-container .el-input input {
      color: $cursor;
    }
  }*/

  /* reset element-ui css */
  .login-container {
    height: 100%;
    width: 100%;
    overflow: hidden;
    position: relative;
    background: url(../../assets/bg.jpg) no-repeat;
    background-size: 100% 100%;
    font-size: 18px;
    .title-container {
      color: #ffffff;
      font-size: 30px;
      font-weight: 500;
      margin-bottom: 50px;
      text-align: center;
    }
    .login-form {
      position: absolute;
      width: 480px;
      right: 10%;
      top: 50%;
      transform: translate(0,-50%);
      .el-form-item {
        margin-bottom: 25px;
      }
      .svg-container{
        position: absolute;
        left: 20px;
        top: 20px;
        z-index: 2;
      }
      .show-pwd{
        position: absolute;
        right: 20px;
        top: 20px;
        z-index: 3;
      }
      .svg-icon{
        width: 1.5rem;
        height: 1.5rem;
        color: #1697f1;
      }
      .el-input--medium .el-input__inner{
        height: 68px;
        line-height: 68px;
        padding: 0 55px;
        color: #666666;
        font-size: 20px;
        background-color: #fff;
        border-radius: 34px;
        border: none;
      }
      .el-form-item__error{
         margin-left: 40px;
       }
      .el-form-item__label{
        color: #ffffff;
        font-size: 18px;
        font-weight: 500;
      }
      .el-form-item.is-required:not(.is-no-asterisk)>.el-form-item__label:before{
        display: none;
      }
    }
    .login-top {
      /*font-size: 45px;*/
      width: 100%;
      /*background-position: top center;*/
      /*color: #34effb;*/
      /*font-weight: 600;*/
      /*font-family: Microsoft YaHei;*/
      /*letter-spacing: 4px;*/
      /*text-align: center;*/
      /*line-height: 60px;*/
      /*display: flex;*/
      /*align-items: center;*/
      /*justify-content: center;*/
      /*margin-top: 30px;*/
      /*background-image:-webkit-linear-gradient(bottom,#189edc,#0f7cea);*/
      /*-webkit-background-clip:text;*/
      /*-webkit-text-fill-color:transparent;*/
      img{
        margin: 30px auto 0 auto;
        display: block;
      }
    }
    .remember-password{
      margin-bottom: 20px;
      margin-top: -5px;
      margin-left: 10px;
      .el-checkbox__label{
        color: #ffffff;
        font-size: 16px;
      }
    }
    .login-btn-box{
      display: flex;
      justify-content: space-between;
    }
    .login-btn{
      width: 100%;
      height: 68px;
      line-height: 68px;
      padding: 0;
      font-size: 24px;
      border-radius: 34px;
     // font-weight: bold;
    }
  }
  .footer{
    text-align: center;
    width: 100%;
    position: fixed;
    left: 0;
    right: 0;
    bottom: 20px;
    font-size: 16px;
    color: #ffffff;
    z-index: 10;
  }
</style>
