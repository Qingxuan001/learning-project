<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt />
      </div>
      <!-- 登录表单区域 -->
      <!-- 
        在组件 el-form 上 添加 引用ref 相当于获取这个DOM元素
        :model="loginForm" 父组件做属性绑定 下面的 el-form-item 子组件通过 prop 接收
        label-width="0px" form-item占的宽度 
      -->
      <el-form
        ref="loginFormRef"
        :model="loginForm"
        :rules="loginFormRules"
        label-width="0px"
        class="login_form"
      >
        <!-- 用户名 -->
        <!-- 
          将form-item 的 prop 属性 usename 设置为校验的规则（即子组件通过prop接收父组件数据）
        -->
        <el-form-item prop="username">
          <!-- 
            prefix-icon 为输入框前置图标  采用字体图标（element-ui中或者阿里图标库）
                将fonts文件夹放入assets  同时  在 main.js中导入字体图标
            子组件通过loginForm.username接收父组件传递过来的数据 并做双向绑定 
          -->
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            prefix-icon="iconfont icon-3702mima"
            type="password"
          ></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
// 1、export 可以导出多个命名模块，引入时（都用import）用大括号括起来
// 2、export default 只能导出一个默认模块，这个模块可以匿名，引入的时候可以给这个模块取任意名字，且不需要用大括号括起来
export default {
  data() {
    return {
      // 这是登录表单的数据绑定对象---- el-form 父组件的属性绑定
      loginForm: {
        username: 'admin',
        password: '123456',
      },
      // 这是表单的验证规则对象---- el-form 父组件的属性绑定
      loginFormRules: {
        // 验证用户名是否合法
        username: [
          { required: true, message: '请输入登录名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' },
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入登录密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' },
        ],
      },
    }
  },
  methods: {
    // 点击重置按钮，重置登录表单
    resetLoginForm() {
      // console.log(this);
      // this=>当前组件对象，其中的属性$refs包含了设置的表单ref
      // ref 加在了子组件 el-form 上，用 this.$refs.ref值  获取到组件实例（或者说引用对象）  后 可以使用组件所有方法
      // 表单内容重置 resetFields() 方法为 el-form  自带的方法（详细看element 官网）
      this.$refs.loginFormRef.resetFields()
    },
    // 前端所做的部分登录验证 以及 token 原理的使用
    login() {
      // 点击登录的时候先调用validate方法验证表单内容是否有误 返回布尔值
      this.$refs.loginFormRef.validate(async (valid) => {
        // false 未通过验证 直接return
        if (!valid) return
        // true 通过验证
        // 1.使用axios封装过得方法 this.$http.post 想服务器发送 post 请求 进行登录
        // 2.请求参数为 请求地址（请求地址的根路径已在 main.js 中处理） 携带参数
        // 3.服务器返回一个 promise 使用 async await 做异步处理 得到具体的响应对象
        // 4.再使用 对象解构  获取 data 数据  这里的 res 为自定义名字
        const { data: res } = await this.$http.post('login', this.loginForm)
        //获取返回的状态码 判断是否成功 并给用户做出相应的弹窗提示（用户体验优化）
        // $message 方法需要在 element.js 中在 vue.prototype 上挂载
        if (res.meta.status !== 200) return this.$message.error('登录失败')
        this.$message.success('登录成功')
        // 1. 将登录成功之后的 token，保存到客户端的 sessionStorage（会话存储机制） 中
        //   1.1 项目中出了登录之外的其他API接口，必须在登录之后才能访问
        //   1.2 token 只应在当前网站打开期间生效，所以将 token 保存在 sessionStorage 中
        window.sessionStorage.setItem('token', res.data.token)
        // 2. 通过编程式导航跳转到后台主页，路由地址是 /home
        this.$router.push('/home')
      })
    },
  },
}
</script>
 
<style lang="less" scoped>
// 使用 less 做预处理  scoped 使当前样式只在当前页面生效
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);

  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}

.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}

.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
