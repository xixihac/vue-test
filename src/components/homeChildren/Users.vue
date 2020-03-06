<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="10">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="success" @click="dialogVisible=true">添加员工</el-button>
        </el-col>
      </el-row>

      <el-table
        :data="userList"
        style="width: 100%" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="mobile" label="手机"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state" @change="stateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column label="操作" width="140px">
          <el-button type="primary" icon="el-icon-edit" circle size="mini"></el-button>
          <el-button type="danger" icon="el-icon-delete" circle size="mini"></el-button>
          <el-tooltip class="item" content="设置角色" placement="top" :enterable="false">
            <el-button type="warning" icon="el-icon-setting" circle size="mini"></el-button>
          </el-tooltip>
        </el-table-column>
      </el-table>
      <div class="block">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[1,2,5,10]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>

        <!--对话框-->
        <el-dialog
          title="用户添加"
          :visible.sync="dialogVisible"
          width="30%" @close="addFormClosed">
          <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px" >
            <el-form-item label="用户名" prop="username">
              <el-input v-model="addForm.username" prefix-icon="el-icon-user-solid"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
              <el-input v-model="addForm.password" prefix-icon="el-icon-user-solid"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input v-model="addForm.email" prefix-icon="el-icon-user-solid"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
              <el-input v-model="addForm.mobile" prefix-icon="el-icon-user-solid"></el-input>
            </el-form-item>


          </el-form>
          <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
        </el-dialog>
      </div>

    </el-card>


  </div>
</template>

<script>
  export default {
    name: 'users',
    data () {
      var checkEmail = (rule, value, cb) => {
        const regEmail = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
        if (regEmail.test(value)) {
          return cb()
        }
        cb(new Error('请输入合法邮箱'))
      }

      var checkMobile = (rule, value, cb) => {
        const regMobile = /^1(3|4|5|7|8)\d{9}$/
        if (regMobile.test(value)) {
          return cb()
        }
        cb(new Error("请输入合法的手机号"));

      }

      return {
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 2
        },
        total: 0,
        userList: [],
        dialogVisible: false,
        addForm: {
          username: '',
          password: '',
          email: '',
          mobile: ''
        },
        addFormRules: {
          username: [{
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          },
            {
              min: 3,
              max: 10,
              message: '输入值在3-10之间',
              trigger: 'blur'
            }],
          password: [{
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
            {
              min: 3,
              max: 10,
              message: '输入值在3-10之间',
              trigger: 'blur'
            }],
          email: [{
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          }, {
            validator: checkEmail,
            trigger: 'blur'
          }],
          mobile: [{
            required: true,
            message: '请输入手机',
            trigger: 'blur'
          }, {
            validator: checkMobile,
            trigger: 'blur'
          }]
        }
      }
    },
    created () {
      this.getUserList()
    },
    methods: {
      async getUserList () {
        const { data: res } = await this.$http.get('/users', {
          params: this.queryInfo
        })
        if (res.meta.status !== 200) {
          this.$message.error('获取用户列表失败')
          return
        }
        this.$message.success('成功获取')
        this.userList = res.data.users
        this.total = res.data.total
      },
      handleSizeChange (newSize) {
        this.queryInfo.pagesize = newSize
        this.getUserList()
      },
      handleCurrentChange (newPage) {
        this.queryInfo.pagenum = newPage
        this.getUserList()
      },
      async stateChange (userInfo) {
        const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
        console.log(res)
        if (res.meta.status !== 200) {
          this.$message.error('更新失败')
          userInfo.mg_state = !userInfo.mg_state
          return
        }
        this.$message.success('更新成功')

      },
      addFormClosed(){
        this.$refs.addFormRef.resetFields();
      },
       addUser(){
        this.$refs.addFormRef.validate(async valid=>{
          if(!valid) return;
          //校验成功
          const {data:res} = await this.$http.post("users",this.addForm);
          console.log(res);
          if (res.meta.status!==201){
            this.$message.error(res.meta.msg);
            return
          }

          this.$message.success("添加成功");
          this.dialogVisible=false;
          this.getUserList();
        })
      }
    }

  }
</script>

<style scoped>

</style>
