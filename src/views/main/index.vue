<template>
  <div>
    <el-container>
      <el-header class="herder">
        <div class="header-left">
          <img src="../../assets/logo.png" class="logo">
          <i>贺米粒值得您信任的品牌</i>
        </div>
        <el-button type="text" @click="logout">退出登录</el-button>
      </el-header>
      <el-container class="main">
        <el-aside width="180px">
          <el-menu :default-active="defaultRouer" router class="el-menu-vertical-demo">
            <el-menu-item :index="item.path" v-for="(item,index) in menus" :key="index">
              <i :class="item.icon"></i>
              <span slot="title">{{item.name}}</span>
            </el-menu-item>
          </el-menu>
        </el-aside>
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>
<script>
export default {
  data() {
    return {
      menus: [
        {
          name: "用户管理",
          path: "user",
          icon: "el-icon-user"
        },
        {
          name: "商户管理",
          path: "merchant",
          icon: "el-icon-connection"
        },
        {
          name: "终端管理",
          path: "terminal",
          icon: "el-icon-monitor"
        }
      ],
      defaultRouer: "user"
    };
  },
  created() {
    this.defaultRouer = this.$route.name;
  },
  methods: {
    logout() {
      localStorage.removeItem("code");
      this.$router.push({ path: "/login" });
    }
  }
};
</script>
<style lang="less" scoped>
.herder {
  background: #303133;
  color: #fff;
  display: flex;
  justify-content: space-between;
  align-items: center;
  .header-left {
    display: flex;
    align-items: center;
  }
  .logo {
    width: 40px;
  }
}
.main {
  height: calc(100vh - 60px);
  .el-menu {
    box-sizing: border-box;
    border: none;
    height: 100%;
    .el-menu-item {
      cursor: pointer;
      width: 100%;
      height: 40px;
      line-height: 40px;
    }
  }
  .el-main {
    background: #f2f6fc;
  }
}
.router-link-active {
  color: #409eff;
}
</style>