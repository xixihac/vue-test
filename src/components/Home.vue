<template>

  <el-container class="home-container">
    <el-header>Header<el-button type="info" @click="logout">退出</el-button></el-header>
    <el-container>
      <el-aside width="200px">
          <el-menu
            background-color="lightcoral"
            text-color="#fff"
            active-text-color="#ffd04b" unique-opened router :default-active="activePath">
            <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id" >
              <template slot="title">
                <i class="el-icon-location"></i>
                <span>{{item.authName}}</span>
              </template>


              <el-menu-item :index="'/'+item2.path" v-for="item2 in item.children" :key="item2.id" @click="setActivePath('/'+item2.path)">
                <template slot="title" >
                  <i class="el-icon-menu"></i>
                  <span>{{item2.authName}}</span>
                </template>
              </el-menu-item>
            </el-submenu>
          </el-menu>
      </el-aside>

      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    name: 'Home',
    data(){
      return {
        menuList: [],
        activePath:'/users'
      }
    },
    methods:{
      logout(){
        window.sessionStorage.clear();
        this.$router.push("/login");

      },
      async getMenuList(){
      const {data : res} =  await this.$http.get("/menus")
        if (res.meta.status!==200) {
          res.$message.error(res.meta.msg);
          return
        }

        this.menuList=res.data;

      },
      setActivePath(activePath){
        this.activePath=activePath;
        window.sessionStorage.setItem("activePath",activePath);
      }
    },
    created () {
      this.getMenuList();
      this.activePath = window.sessionStorage.getItem("activePath");
    }
  }
</script>

<style scoped>

  .home-container{
    height: 100%;
  }
  .el-header{
    background-color: lightgray;
  }
  .el-aside{
    background-color: lightcoral;
  }
  .el-main{
    background-color: lightblue;
  }
</style>
