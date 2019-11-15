<template>
  <div class="page-login">
    <van-nav-bar left-arrow @click-left="$router.back()" title="登 录"></van-nav-bar>
    <van-cell-group>
      <van-field
        @input="validMobile"
        :error-message="errMsg.mobile"
        v-model.trim="loginForm.mobile"
        label="手机号"
        placeholder="请输入手机号"
      />
      <van-field
        @input="validCode"
        :error-message="errMsg.code"
        v-model.trim="loginForm.code"
        label="验证码"
        placeholder="请输入验证码"
      >
        <van-button class="p10" slot="button" size="mini" type="primary">发送验证码</van-button>
      </van-field>
    </van-cell-group>
    <div class="btn_box">
      <van-button type="info" @click="submitLogin" block round>登 录</van-button>
    </div>
  </div>
</template>

<script>
import { login } from '@/api/user'
import { mapMutations } from 'vuex'
export default {
  data () {
    return {
      // 表单数据
      loginForm: {
        mobile: '',
        code: ''
      },
      // 错误信息
      errMsg: {
        mobile: '',
        code: ''
      }
    }
  },
  methods: {
    // 当时去焦点的时候，获取当前输入框的值，进行校验。如果失败给error-message赋值。如果成功，清空值
    checkMobile () {
      const value = this.loginForm.mobile
      if (!value) {
        this.errMsg.mobile = '请输入手机号'
        return false
      }
      if (!/^1[3-9]\d{9}$/.test(value)) {
        this.errMsg.mobile = '请输入正确的手机号'
        return false
      }
      this.errMsg.mobile = ''
    },
    checkCode () {
      const value = this.loginForm.code
      // 1.非空
      if (!value) {
        this.errMsg.code = '请输入验证码'
        return false
      }
      if (!/^\d{6}$/.test(value)) {
        this.errMsg.code = '请输入6个数字'
        return false
      }
      this.errMsg.code = ''
    },
    // 校验手机号
    validMobile () {
      const value = this.loginForm.mobile
      if (!value) {
        this.errMsg.mobile = '请输入手机号'
      } else if (!/^1[3-9]\d{9}$/.test(value)) {
        this.errMsg.mobile = '手机号格式不对'
      } else {
        this.errMsg.mobile = ''
      }
    },
    // 校验验证码
    validCode () {
      const value = this.loginForm.code
      if (!value) {
        this.errMsg.code = '请输入验证码'
      } else if (!/^\d{6}$/.test(value)) {
        this.errMsg.code = '验证码6位数字'
      } else {
        this.errMsg.code = ''
      }
    },
    // 登录时候  整体校验
    // 进行整体表单的校验  当点击登录的时候
    // 调用 checkMobile checkCode 后
    // 判断成功的标准：errMsg中没有错误信息
    async submitLogin () {
      this.checkMobile()
      this.checkCode()
      // 判断 errMsg 中没有任何错误信息 代表成功
      if (this.errMsg.mobile || this.errMsg.code) {
        return false
      }
      try {
        // 校验完毕  进行登录
        // 1. 封装一个登录的API 在api目录user模块
        // 2. 在当前组件下导入
        // 3. 调用函数发请求
        const data = await login(this.loginForm)
        // 登录成功
        // 1. 更新vuex和本地存储的token相关的数据
        this.setUser(data)
        // 2. 回跳 （地址栏有回跳地址，根据地址跳转，如果没有，默认个人中心）
        // 回跳地址  this.$route.query.redirectUrl
        // 个人中心的地址  /user
        // | 或  || 短路或   & 与  &&短路与
        this.$router.push(this.$route.query.redirectUrl || '/user')
      } catch (e) {
        // 错误提示
        this.$toast.fail('手机号或验证码错误')
      }
    },
    ...mapMutations(['setUser'])
  }
}
</script>

<style scoped lang='less'>
.p5 {
  padding: 0 5px;
}
.btn_box {
  padding: 10px;
  .van-button {
    height: 32px;
    line-height: 30px;
  }
}
</style>
