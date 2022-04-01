<template>
  <div class="register_container">
    <!-- 盒子区域 -->
    <div class="register_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div>

      <!-- 登录表单区域 -->
      <el-form ref="registerFormRef" :model="registerForm" :rules="registerFormRules" label-width="0px" class="register_form"> <!-- label是文本宽度 :model绑定上数据对象 相当于父传子 子组件拿到数据并保存起来 -->
        <!-- 用户名 -->
        <el-form-item prop="username"><!-- rules和prop是数据验证的 rules放所有规则 -->
          <el-input v-model="registerForm.username" prefix-icon="iconfont icon-user"></el-input> <!-- prefix是在前面添加图标 图标是阿里图标 v-model取对应的数据 -->
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input v-model="registerForm.password" prefix-icon="iconfont icon-3702mima" type="password"></el-input><!-- password表明要隐藏 -->
        </el-form-item>
        <el-form-item prop="mail">
          <el-input v-model="registerForm.mail"></el-input><!-- password表明要隐藏 -->
        </el-form-item>
        <el-form-item prop="mobile">
          <el-input v-model="registerForm.mobile"></el-input><!-- password表明要隐藏 -->
        </el-form-item>
        <el-form-item prop="code">
          <el-input v-model="registerForm.code"></el-input><!-- password表明要隐藏 -->
        </el-form-item>
        <el-form-item prop="role">
            <el-select v-model="registerForm.roleName" placeholder="请选择">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
        </el-form-item>

        <!-- 按钮区域 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="register">register</el-button>
          <el-button type="info" @click="resetregisterForm">reset</el-button>
        </el-form-item>
      </el-form>

    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      options:[
        {
          label: '商家',
          value:'shopper',

        },{
          label: '买家',
          value: 'purchaser',
        }
      ],
      // 这是登录表单的数据绑定对象 存储着要绑定的数据 用于数据验证 发送请求等
      registerForm: {
        username: '', /* 后面登录就不用再输 */
        password: '',
        mail: '',
        mobile: '',
        code: '',
        roleName: 'purchaser',
      },
      // 这是表单的验证规则对象
      registerFormRules: {
        // 验证用户名是否合法 都是element ui里面的规则
        username: [
          { required: true, message: '请输入登录名称', trigger: 'blur' }, /* 鼠标失焦触发验证 */
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入登录密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        mail: [
          { required: true, message: 'mail plz',trigger: 'blur'},
          {min: 5,message: 'format wrong',trigger: 'blur'}
        ],
        mobile: [
          {required: true,message: 'mobile plz',trigger: 'blur'},
          {min: 11,max: 11,message: 'length in 11',trigger: 'blur'}
        ],
        code: [
          {required: true,message: 'code plz',trigger: 'blur'},
          {min: 6,max: 6,message: 'length in 6',trigger: 'blur'}
        ]
      }
    }
  },
  methods: {
    // 点击重置按钮，重置登录表单
    resetregisterForm () {
      // console.log(this); 本组件实例对象 里面有$refs属性
      this.$refs.registerFormRef.resetFields() /* 看element ui */
    },
    register () {
      this.$refs.registerFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('/api/user/v1/register', {
          name: this.registerForm.username,
          passwd: this.registerForm.password,
          mail: this.registerForm.mail,
          mobile: this.registerForm.mobile,
          code: this.registerForm.code,
        }, {
          headers: {
            'Content-Type': 'application/json'
          }
        }) /* 加await返回的是数据 不加返回promise 而返回的数据里面只有data才是api返回的数据 其他的都是axios的 解构并重命名即data重命名为res */
        if (res.code !== 200) return this.$message.error('register 失败！') /* 参见element 弹框种类 */
        this.$message.success('register 成功')

        window.sessionStorage.setItem('token', res.data.token)
        // 2. 通过编程式导航跳转到后台主页，路由地址是 /home 但是如果没有拿到token而直接输入地址也不应该跳转 此时要用导航守卫 如果要跳转的是登录页面则直接放行 如果跳转的是其他页面则要判断是否携带token 导航守卫写在router.js里面
        this.$router.push('/')
      })
    }
  }
}
</script>

<style lang="less" scoped> /* 支持less语法格式 */
.register_container {
  background-color: #2b4b6b;
  height: 100%;
}

.register_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  // 居中
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

.register_form {
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
