<template>
  <div class="login" :style="'background-image:url('+ Background +');'">
  <el-form ref="loginForm" :model="loginForm" :rules="loginRules" label-position="left" label-width="0px" class="login-form">
    <h3 class="title">
      MyShopAdmin
    </h3>
    <!-- 登录表单 -->
    <el-form-item prop="username">
      <el-input v-model="loginForm.username" type="text" auto-complete="off" placeholder="账号">
  <!--      <svg-icon slot="prefix" icon-class="user" class="el-input__icon input-icon" />-->
      </el-input>
    </el-form-item>
    <el-form-item prop="password">
      <el-input v-model="loginForm.password" type="password" auto-complete="off" placeholder="密码" @keyup.enter.native="handleLogin">
  <!--      <svg-icon slot="prefix" icon-class="password" class="el-input__icon input-icon" />-->
      </el-input>
    </el-form-item>
    <el-form-item prop="code">
      <el-input v-model="loginForm.code" auto-complete="off" placeholder="验证码" style="width: 63%" @keyup.enter.native="handleLogin">
  <!--      <svg-icon slot="prefix" icon-class="validCode" class="el-input__icon input-icon" />-->
      </el-input>
      <div class="login-code">
        <img :src="codeUrl" @click="getCode">
      </div>
    </el-form-item>
    <el-checkbox v-model="loginForm.rememberMe" style="margin:0 0 25px 0;">
      记住我
    </el-checkbox>
    <el-form-item style="width:100%;">
      <el-button :loading="loading" size="medium" type="primary" style="width:100%;" @click.native.prevent="handleLogin">
        <span v-if="!loading">登 录</span>
        <span v-else>登 录 中...</span>
      </el-button>
    </el-form-item>
  </el-form>
  </div>
</template>
<script>
import Cookies from 'js-cookie'
import Config from '@/settings'

import Background from '@/assets/images/background.jpg'
// 引入加密 解密依赖
import {encrypt,decrypt} from "@/utils/rsaEncrypt";
import {setToken} from "@/store/auth";

export default {
    name:"Login",

    created() {
      this.getCode()
      this.getCookie()
    },

    data(){
      return{
        loginForm:{
          username:'admin',
          password: '123456',
          code:'',
          rememberMe:false,
          uuid:''
        },
        //校验规则
        loginRules:{
          username:[{
            required:true,   //是否需要校验
            trigger:'blur', // 触发方式
            message: '用户名不能为空', //提示信息
          }],
          password:[{
            required:true,
            trigger:'blur',
            message:'密码不能为空'
          }],
          code: [{
            required:true,
            trigger:'blur',
            message:'验证码不能为空'
          }],

        },
        Background: Background,
        codeUrl:'',
        loading: false,
        //把加密后的密码 存放到cookie当中
        cookiePass: ''
      }
    },

    methods:{

      //用于获取cookie
      getCookie(){
        const username = Cookies.get('username')
        let password = Cookies.get('password')
        const rememberMe = Cookies.get('rememberMe')

        //保存cookie中加密后的密码
        this.cookiePass = password === undefined ? '':password
        password = password === undefined ? this.loginForm.password : password

        this.loginForm = {
          username: username === undefined ? this.loginForm.username : username,
          password: password,
          rememberMe: rememberMe === undefined ? false : Boolean(rememberMe),
          code: ''
        }
      },

      //验证码图片的getCode方法
      getCode(){
        this.loginForm.password = encrypt(this.loginForm.password)
        //发送请求给后端 需要用到axios
        this.$request.get('http://127.0.0.1:8000/auth/code').then(res=>{
          this.codeUrl = res.data.img
          this.loginForm.uuid = res.data.uuid
        })
      },

      //处理登录请求
      handleLogin(){

        this.$refs.loginForm.validate(valid =>{
          //如果表单校验通过则发送登录请求
          if(valid){
            const user = {
              username: this.loginForm.username,
              password: this.loginForm.password,
              rememberMe: this.loginForm.rememberMe,
              code:this.loginForm.code,
              uuid:this.loginForm.uuid
            }

            if(user.password !== this.cookiePass){
              user.password = encrypt(user.password)
            }

            //如果勾选了 rememberMe 则将用户信息保存到cookie中
            if(user.rememberMe){
              Cookies.set('username',user.username,{
                expires: Config.passCookieExpires
              })

              Cookies.set('password',user.password,{
                expires: Config.passCookieExpires
              })

              Cookies.set('rememberMe',user.rememberMe,{
                expires: Config.passCookieExpires
              })
            }else {
              Cookies.remove('username')
              Cookies.remove('password')
              Cookies.remove('rememberMe')
            }

            this.loading = true

            this.$request.post('http://127.0.0.1:8000/auth/login',user).then(res=>{
              console.log(res)
              //将token存入 cookie
              setToken(res.data.token,this.loginForm.rememberMe)
              //setToken(res.data.token,this.loginForm)
              //进行路由跳转
              //在history 模式下 可以使用push返回
              this.$router.push('/')
            }).catch(()=>{
              this.loading = false
              this.getCode()
            })
          }else{
            alert('请完善登录信息')
          }
        })

      },
    }
  }
</script>

<!-- scss css的扩展语言 -->
<style rel="stylesheet/scss" lang="scss">
.login {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  background-size: cover;
}
.title {
  margin: 0 auto 30px auto;
  text-align: center;
  color: #707070;
}

.login-form {
  border-radius: 6px;
  background: #ffffff;
  width: 385px;
  padding: 25px 25px 5px 25px;
  .el-input {
    height: 38px;
    input {
      height: 38px;
    }
  }
  .input-icon{
    height: 39px;width: 14px;margin-left: 2px;
  }
}
.login-tip {
  font-size: 13px;
  text-align: center;
  color: #bfbfbf;
}
.login-code {
  width: 33%;
  display: inline-block;
  height: 38px;
  float: right;
  img{
    cursor: pointer;
    vertical-align:middle
  }
}
</style>

