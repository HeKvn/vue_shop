<template>
  <div class="login_container">
    <div class="login_box">
        <!-- 头像区域 -->
        <div class="avatar_box">
            <img src="../assets/logo.png" alt="">
        </div>
        <!-- 登陆表单区域 -->
        <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
            <!-- 用户名 -->
            <el-form-item prop="username">
                <el-input v-model="loginForm.username" prefix-icon="el-icon-user"></el-input>
            </el-form-item>
            <!-- 密码 -->
            <el-form-item prop="password">
                <el-input v-model="loginForm.password" prefix-icon="el-icon-lock" show-password></el-input>
            </el-form-item>
            <!-- 按钮区域 -->
            <el-form-item class="btns">
                <el-button type="primary" plain @click="login">登陆</el-button>
                <el-button type="info" plain @click="resetLoginForm">重置</el-button>
            </el-form-item>
        </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 这是登陆表单的数据绑定对象
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 这是表单的验证规则对象（校验规则的属性名称要一致，如username要和表单中的属性名称username保持一致，否则会出错不执行）
      loginFormRules: {
        // 验证用户名是否合法
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 5, max: 11, message: '长度在 5 到 11 个字符', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 点击重置按钮，重置表单数据
    resetLoginForm () {
    // console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      const that = this
      that.$refs.loginFormRef.validate(async (val) => {
        // console.log(val)
        if (val) {
          // 解构赋值
          const { data: res } = await that.$http.post('login', that.loginForm)
          if (res.meta.status !== 200) {
            that.$message.error('登陆失败')
          } else {
            that.$message.success('登陆成功')
            // 1. 将登陆成功后的 token，保存到客户端的 sessionStorage 中
            //    1. 项目中除了登陆之外的其他API接口，必须在登陆之后才能访问
            //    2. token 只应在当前网站打开期间生效，所以将 token 保存在 sessionStorage中
            window.sessionStorage.setItem('token', res.data.token)
            // 2. 通过编程式导航跳转到后台主页，路由地址是 /home
            that.$router.push('/home')
          }
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container{
    background-color: #2b4b6b;
    height: 100%;
}

.login_box{
    width: 450px;
    height: 300px;
    background-color: white;
    border-radius: 3px;
    // 居中的另一个方法
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    .avatar_box{
        height: 130px;
        width: 130px;
        border: 1px solid #eee;
        border-radius: 50%;
        padding: 10px;
        box-shadow: 0 0 10px #ddd;
        position: absolute;
        left: 50%;
        transform: translate(-50%, -50%);
        background-color: white;
        img{
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background-color: #eee;
        }
    }
}

.login_form{
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 10px;
    box-sizing: border-box;
}

.btns{
    display: flex;
    justify-content: flex-end;
}

</style>
