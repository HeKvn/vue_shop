<template>
    <el-container class="home-container">
        <!-- 头部区域 -->
        <el-header>
          <div class="header_box">
            <img src="../assets/kun.png" alt="">
            <span>后台管理系统</span>
          </div>
          <el-button @click="logout">退出</el-button>
        </el-header>
        <!-- 主体区域 -->
        <el-container>
            <!-- 侧边栏 -->
            <el-aside :width="isCollapse ? '64px':'200px'">
              <div class="toggle-button" @click="toggleCollapse">|||</div>
              <!-- 侧边栏菜单区域 -->
              <el-menu unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
                <!-- 一级菜单(众所周知，数字拼接字符串将得到字符串) -->
                <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
                  <!-- 一级菜单的模版区 -->
                  <template slot="title">
                    <!-- 图标 -->
                    <i :class="iconObj[item.id]"></i>
                    <!-- 文本 -->
                    <span>{{item.authName}}</span>
                  </template>
                  <!-- 二级菜单 -->
                  <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id"
                  @click="saveNavState('/' + subItem.path)">
                    <template slot="title">
                      <i class="el-icon-menu"></i>
                      <span>{{subItem.authName}}</span>
                    </template>
                  </el-menu-item>
                </el-submenu>
              </el-menu>
            </el-aside>
            <!-- 右侧 -->
            <el-main><router-view></router-view></el-main>
        </el-container>
    </el-container>
</template>

<script>
export default {
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  data () {
    return {
      // 左侧菜单数据
      menulist: [],
      iconObj: {
        125: 'el-icon-user-solid',
        103: 'el-icon-box',
        101: 'el-icon-s-goods',
        102: 'el-icon-document',
        145: 'el-icon-s-data'
      },
      // 是否折叠
      isCollapse: false,
      // 导航菜单激活状态
      activePath: ''
    }
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有的菜单
    async getMenuList () {
      const { data: res } = await this.$http.get('/menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      console.log(res)
    },
    // 点击按钮切菜单的折叠与展开
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    // 保存链接的激活状态
    saveNavState (e) {
      window.sessionStorage.setItem('activePath', e)
      this.activePath = window.sessionStorage.getItem('activePath')
    }
  }
}
</script>

<style lang="less" scoped>
.home-container{
    height: 100%;
}
.el-header{
    background-color: #9f9fa0;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    .header_box{
      height: 100%;
      color: #fff;
      font-size: 20px;
      display: flex;
      align-items: center;
      img{
        height: 100%;
      }
      span{
        margin-left: 15px;
      }
    }
}
.el-aside{
    background-color: #ffffff;
}
.el-main{
    background-color: #ffffff;
}
.toggle-button{
  font-size: 10px;
  line-height: 24px;
  text-align: center;
  border-right: 1px solid #e6e6e6;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
