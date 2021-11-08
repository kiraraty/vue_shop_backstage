<template>
  <el-container class="form-container">
    <el-header>
      <div>
        <img src="../assets/icons8-shop-50.png" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>

    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
          active-text-color="#409eff"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          :router="true"
          default-active="activePath"
        >
          <el-submenu
            :index="item.id + ''"
            v-for="item in menulist"
            :key="item.id"
          >
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>

            <el-menu-item
              :index="'/'+subItem.path"
              v-for="subItem in item.children"
              :key="subItem.id"
              @click="saveNavState('/'+subItem.path)"
            >
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ subItem.authName }}</span>
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
  data() {
    return {
      menulist: [],
      iconsObj: {
        125: "iconfont icon-user",
        101: "iconfont icon-tijikongjian",
        102: "iconfont icon-shangpin",
        103: "iconfont icon-danju",
        145: "iconfont icon-baobiao",
      },
      isCollapse: false,
      activePath:''
    };
  },
  created() {
    this.getMenuList();
    this.activePath=window.sessionStorage.getItem('activePath');
  },
  methods: {
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    },
    async getMenuList() {
      const { data: res } = await this.$http.get("menus");
      if (res.meta.status !== 200) return this.$message.error(res.meta.message);
      this.menulist = res.data;
      console.log(res);
    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse;
    },
    saveNavState(){
      window.sessionStorage.setItem('activePath',activePath);
      this.activePath=activePath;
    },
    
  },
};
</script>

<style lang="less" scoped>
.el-header {
  background-color: aquamarine;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  color: #fff;
  font-size: 30px;
  > div {
    display: flex;
    align-items: center;
  }
  > span {
    margin-left: 30px;
  }
}
.el-aside {
  background-color: rgb(127, 255, 191);
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: rgb(127, 255, 165);
}
.form-container {
  height: 100%;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: aqua;
  font-size: 10px;
  line-height: 24px;
  color: azure;
  text-align: center;
  cursor: pointer;
}
</style>