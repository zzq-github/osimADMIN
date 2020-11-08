<template>
  <nav class="site-navbar" :class="'site-navbar--' + navbarLayoutType">
    <div class="site-navbar__header">
      <h1 class="site-navbar__brand" @click="$router.push({ name: 'home' })">
        <a class="site-navbar__brand-lg" href="javascript:;">Zound后台管理系统</a>
        <a class="site-navbar__brand-mini" href="javascript:;">Zound</a>
      </h1>
    </div>
    <div class="site-navbar__body clearfix">
      <el-menu
        class="site-navbar__menu"
        mode="horizontal">
        <el-menu-item class="site-navbar__switch" index="0" @click="sidebarFold = !sidebarFold">
          <icon-svg name="zhedie"></icon-svg>
        </el-menu-item>
      </el-menu>
      <el-menu
        class="site-navbar__menu site-navbar__menu--right"
        mode="horizontal">
        <el-menu-item class="site-navbar__avatar" index="3">
          <el-dropdown :show-timeout="0" placement="bottom">
            <span class="el-dropdown-link">
              <span>当前品牌：</span>
              {{ selectBrand }}
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item v-for="(item, index) in brandList" :key="index" @click.native="doExchange(item)">{{ item.brandName }}</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-menu-item>
        <el-menu-item class="site-navbar__avatar" index="3">
          <!-- <span>欢迎您</span> -->
          <el-dropdown :show-timeout="0" placement="bottom">
            <span class="el-dropdown-link">
              <!-- <img src="~@/assets/img/avatar.png" alt="admin"> -->
              <span>欢迎您！</span>
              {{loginInfo.account}}
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="logoutHandle()">退出</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-menu-item>
      </el-menu> 
    </div>
  </nav>
</template>

<script>
  import { logout } from '@/api/baseApi'
  import { clearLoginInfo } from '@/utils'
  export default {
    data () {
      return {
        updatePassowrdVisible: false,
        loginInfo: {},
        selectBrand: 'adidas',
        brandList: [
          { tillName: 'ads.demo.wecrm', pwd: 'ads1234', brandName: 'adidas' },
          { tillName: 'ads.demo.wecrm', pwd: 'ads1234', brandName: 'marshal' },
          { tillName: 'ads.demo.wecrm', pwd: 'ads1234', brandName: 'urbanears' }
        ]
      }
    },
    computed: {
      navbarLayoutType: {
        get () { return this.$store.state.common.navbarLayoutType }
      },
      sidebarFold: {
        get () { return this.$store.state.common.sidebarFold },
        set (val) { this.$store.commit('common/updateSidebarFold', val) }
      },
      mainTabs: {
        get () { return this.$store.state.common.mainTabs },
        set (val) { this.$store.commit('common/updateMainTabs', val) }
      },
      // userName: {
      //   get () { return this.$store.state.user.name }
      // }
    },
    mounted () {
      sessionStorage.setItem('headerInfo', JSON.stringify(this.brandList[0]))
      let loginInfo = JSON.parse(sessionStorage.getItem('loginInfo'))
      this.loginInfo = loginInfo
    },
    methods: {
      doExchange (value) {
        this.selectBrand = value.brandName
        sessionStorage.setItem('headerInfo', JSON.stringify(value))
      },
      // 退出
      logoutHandle () {
        this.$confirm(`确定进行[退出]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          sessionStorage.clear();
          this.$router.push({ name: 'login' })
        }).catch(() => {})
      }
    }
  }
</script>
