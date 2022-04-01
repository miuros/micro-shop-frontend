<!--
 * @Description: 用户注册组件
 * @Author: hai-27
 * @Date: 2020-02-19 22:20:35
 * @LastEditors: hai-27
 * @LastEditTime: 2020-03-01 15:34:34
 -->
<template>
  <div id="register">
    <el-dialog title="注册" width="300px" center :visible.sync="isRegister">
      
      <el-form
        :model="RegisterUser"
        :rules="rules"
        status-icon
        ref="ruleForm"
        class="demo-ruleForm"
      >
        <el-form-item prop="name">
          <el-input
            prefix-icon="el-icon-user-solid"
            placeholder="name plz"
            v-model="RegisterUser.name"
          ></el-input>
        </el-form-item>
        <el-form-item prop="pass">
          <el-input
            prefix-icon="el-icon-view"
            type="password"
            placeholder="请输入密码"
            v-model="RegisterUser.passwd"
          ></el-input>
        </el-form-item>
        <el-form-item prop="confirmPass">
          <el-input
            prefix-icon="el-icon-view"
            type="password"
            placeholder="请再次输入密码"
            v-model="RegisterUser.confirmPass"
          ></el-input>
        </el-form-item>
         <el-form-item prop="mail">
          <el-input style="width:75%;margin-right:5%" v-model="RegisterUser.mail"></el-input><el-button size="medium" type="primary" @click="mailTo" style="width:20%;text-align:center;margin: auto" >mail</el-button>
        </el-form-item>
        <el-form-item prop="mobile">
          <el-input v-model="RegisterUser.mobile"></el-input><!-- password表明要隐藏 -->
        </el-form-item>
        <el-form-item prop="code">
          <el-input v-model="RegisterUser.code"></el-input><!-- password表明要隐藏 -->
        </el-form-item>
         <el-form-item prop="role">
            <el-select v-model="RegisterUser.roleName" placeholder="请选择">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
        </el-form-item>
        <el-form-item>
          <el-button size="medium" type="primary" @click="Register" style="width:100%;">注册</el-button>
        </el-form-item>

      </el-form>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: "MyRegister",
  props: ["register"],
  data() {

    // 用户名的校验方法
    // 密码的校验方法
    let validatePass = (rule, value, callback) => {
      if (value === "") {
        return callback(new Error("请输入密码"));
      }
      // 密码以字母开头,长度在6-18之间,允许字母数字和下划线
      const passwordRule = /^[a-zA-Z]\w{5,17}$/;
      if (passwordRule.test(value)) {
        this.$refs.ruleForm.validateField("checkPass");
        return callback();
      } else {
        return callback(
          new Error("字母开头,长度6-18之间,允许字母数字和下划线")
        );
      }
    };
    // 确认密码的校验方法
    let validateConfirmPass = (rule, value, callback) => {
      if (value === "") {
        return callback(new Error("请输入确认密码"));
      }
      // 校验是否以密码一致
      if (this.RegisterUser.passwd != "" && value === this.RegisterUser.passwd) {
        this.$refs.ruleForm.validateField("checkPass");
        return callback();
      } else {
        return callback(new Error("两次输入的密码不一致"));
      }
    };
    return {
      isRegister: false, // 控制注册组件是否显示
      //mail: 'mail',
      options:[
      {
        label: '商家',
        value:'shopper',

      },{
        label: '买家',
        value: 'purchaser',
      }
    ],
      RegisterUser: {
        name: "",
        passwd: "",
        confirmPass: "",
        mail: '',
        mobile: '',
        code: '',
        roleName: '',
      },
      // 用户信息校验规则,validator(校验方法),trigger(触发方式),blur为在组件 Input 失去焦点时触发
      rules: {
        name: [{ required: true,message: "name plz", trigger: "blur" },{min: 3,max:20,message:'length between 3 and 20',trigger: 'blur'}],
        pass: [{ validator: validatePass, trigger: "blur" }],
        confirmPass: [{ validator: validateConfirmPass, trigger: "blur" }],
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
    };
  },
  watch: {
    // 监听父组件传过来的register变量，设置this.isRegister的值
    register: function(val) {
      if (val) {
        this.isRegister = val;
      }
    },
    // 监听this.isRegister变量的值，更新父组件register变量的值
    isRegister: function(val) {
      if (!val) {
        this.$refs["ruleForm"].resetFields();
        this.$emit("fromChild", val);
      }
    }
  },
  methods: {
    mailTo(){
        this.$axios.get("/api/user/v1/mail?mail="+this.RegisterUser.mail).then(res => {
          if(res.data.code==="200"){
             this.notifySucceed(res.data.msg);
          }else{
            this.notifyError(res.data.msg);
          }
        })
    },
    Register() {
      // 通过element自定义表单校验规则，校验用户输入的用户信息
      this.$refs["ruleForm"].validate(valid => {
        //如果通过校验开始注册
        if (valid) {
          this.$axios
            .post("/api/user/v1/register", {
              name: this.RegisterUser.name,
              passwd: this.RegisterUser.passwd,
              mail: this.RegisterUser.mail,
              mobile: this.RegisterUser.mobile,
              code: this.RegisterUser.code,
            })
            .then(res => {
              // “001”代表注册成功，其他的均为失败
              if (res.data.code === "200") {
                // 隐藏注册组件
                this.isRegister = false;
                // 弹出通知框提示注册成功信息
                this.notifySucceed(res.data.msg);
              } else {
                // 弹出通知框提示注册失败信息
                this.notifyError(res.data.msg);
              }
            })
            .catch(err => {
              return Promise.reject(err);
            });
        } else {
          return false;
        }
      });
    }
  }
};
</script>
<style>
</style>