<template>
  <div class="login-container">
    <el-form autoComplete="on" :model="loginForm" :rules="loginRules" ref="loginForm" label-position="left"
             label-width="0px"
             class="card-box login-form">
      <h3 class="title">智慧水务</h3>
      <el-form-item prop="username">
        <el-input v-model="loginForm.username" placeholder="请输入用户名" autoComplete="on"/>
      </el-form-item>
      <el-form-item prop="password">
        <el-input v-model="loginForm.password" type="password" placeholder="请输入密码"
                  show-password autoComplete="on"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" style="width:100%;" :loading="loading" @click.native.prevent="handleLogin">
          登录
        </el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  export default {
    name: 'log',
    data () {
      return {
        loginForm: {
          username: '',
          password: ''
        },
        loginRules: {
          username: [{required: true, trigger: 'blur', message: '请输入用户名'}],
          password: [{required: true, trigger: 'blur', message: '请输入密码'}]
        },
        loading: false
      }
    },
    methods: {
      handleLogin () {
        this.$refs.loginForm.validate(valid => {
          if (valid) {
            this.loading = true
            this.$http.post('http://116.55.241.28:8082/login/auth', {
              'data': this.loginForm
            }).then(data => {
              this.loading = false
              let userJson = JSON.parse(JSON.stringify(data))['data']
              console.log(userJson)
              if (userJson['status'] === 'success' && userJson['deleteStatus'] === 0) {
                this.$store.dispatch('setShiroToken', userJson)
                this.runFBoxAccount()
              } else if (userJson['deleteStatus'] === 1) {
                this.$message.error('账号被冻结')
              } else {
                this.$message.error('账号/密码错误')
              }
            }).catch(() => {
              this.$message.error('登陆服务器异常')
              this.loading = false
            })
          } else {
            this.$message.error('登录信息不完整')
            return false
          }
        })
      },
      runFBoxAccount () {
        this.$http.post('http://116.55.241.28:8082/equip/equipLogin', null, {
          headers: {
            'Authorization': this.$store.state.ShiroToken.token
          }
        }).then(response => {
          window.jsonobj = JSON.parse(JSON.stringify(response))['data']
          console.log('access_token: ' + window.jsonobj['access_token'])
          console.log('refresh_token: ' + window.jsonobj['refresh_token'])
          this.runEquipment()
        }).catch(function (error) {
          console.log(error)
        })
      },
      runEquipment () {
        this.$http.get('http://116.55.241.28:8082/equip/getEquipments', {
          params: {
            Authorization: 'Bearer ' + window.jsonobj['access_token'],
            XFBoxClientId: 'zzy_test'
          },
          headers: {
            'Authorization': this.$store.state.ShiroToken.token
          }
        }).then(response => {
          // 解析请求到的设备数据
          window.equipmentobj = JSON.parse(JSON.stringify(response))['data']
          window.equipmentobjarray = getArray(window.equipmentobj)
          this.gotoIndex()
        }).catch(function (error) {
          console.log(error)
        })
      },
      gotoIndex () {
        this.$router.push('/main')
      }
    }
  }
  function getArray (arrayObj) {
    let array = []
    for (let i = 0; i < arrayObj.length; i++) {
      array = array.concat(arrayObj[i]['boxRegs'])
    }
    return array
  }
</script>
<style rel="stylesheet/scss" lang="scss">
  // import "../../styles/mixin.scss";
  $bg: #2d3a4b;
  $dark_gray: #889aa4;
  $light_gray: #eee;

  .login-container {
    include :relative;
    height: 100vh;
    background-color: $bg;
    input:-webkit-autofill {
      -webkit-box-shadow: 0 0 0px 1000px #293444 inset !important;
      -webkit-text-fill-color: #fff !important;
    }
    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $light_gray;
      height: 47px;
    }
    .el-input {
      display: inline-block;
      height: 47px;
      width: 85%;
    }
    .tips {
      font-size: 14px;
      color: #fff;
      margin-bottom: 10px;
    }
    .svg-container {
      padding: 6px 5px 6px 15px;
      color: $dark_gray;
      vertical-align: middle;
      width: 30px;
      display: inline-block;
      &_login {
        font-size: 20px;
      }
    }
    .title {
      font-size: 26px;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }
    .login-form {
      position: absolute;
      left: 0;
      right: 0;
      width: 400px;
      padding: 35px 35px 15px 35px;
      margin: 120px auto;
    }
    .el-form-item {
      border: 1px solid rgba(255, 255, 255, 0.1);
      background: rgba(0, 0, 0, 0.1);
      border-radius: 5px;
      color: #454545;
    }
    .show-pwd {
      position: absolute;
      right: 10px;
      top: 7px;
      font-size: 16px;
      color: $dark_gray;
      cursor: pointer;
    }
    .thirdparty-button {
      position: absolute;
      right: 35px;
      bottom: 28px;
    }
  }
</style>
