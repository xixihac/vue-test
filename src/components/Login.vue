<template>
  <div>
    怎么回事?

    <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="80px">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="loginForm.username"  prefix-icon="el-icon-user-solid"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="loginForm.password" prefix-icon="el-icon-user" type="password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="login">登录</el-button>
        <el-button type="info" @click="resetLoginForm">重置</el-button>
      </el-form-item>
    </el-form>

  </div>


</template>

<script>
  export default {
    data() {
      return {
        loginForm: {
          username:'',
          password:''
        },
        loginFormRules:{
          username:[
            { required: true, message: '请输入用户名', trigger: 'blur' },
            { min: 3, max: 10, message: '用户名长度在 3 到 10 个字符', trigger: 'blur' }
          ],
          password:[
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 3, max: 10, message: '密码长度在 3 到 10 个字符', trigger: 'blur' }
          ]
        }
      }
    },
    methods:{
      resetLoginForm(){
        this.$refs.loginFormRef.resetFields();
      },
      login: function () {
        this.$refs.loginFormRef.validate(async valid => {
          // console.log(valid)
          if (!valid) return;
          const {data:res} = await this.$http.post('login', this.loginForm);
          console.log(res);
          if (res.meta.status==200){
            this.$message.success('成功登录');
            window.sessionStorage.setItem('token',res.data.token);
            this.$router.push("/home");
          }else{
            this.$message.error('登录失败');
          }


        })

      }
    }

    }



</script>

<style lang="less" scoped>

</style>
