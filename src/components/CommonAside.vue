<template>
  <el-menu
    default-active="2"
    class="el-menu-vertical-demo"
    :collapse="isCollapse"
    background-color="white"
    text-color="grey"
    active-text-color="black"
  >
    <div style="height: 60px; line-height: 60px; text-align: center">
      <img
        src="@/assets/logo.png"
        alt=""
        style="width: 32px; position: relative; top: 5px; margin-right:2px"
      />
      <span style="color: black; font-weight: bold;font-size: 18px;">
        {{ isCollapse ? '' : '人力资源管理系统' }}
      </span>
    </div>
    <el-menu-item
      v-for="item in noChildren"
      :key="item.id"
      :index="item.name"
      @click="clickMenu(item,item.path)"
    >
      <i :class="'iconfont icon-r-' + item.icon" style="font-size: 26px;"></i>
      <span slot="title" style="font-size: 20px;"> {{ item.name }}</span>
    </el-menu-item>
    <el-submenu v-for="item in hasChildren" :key="item.id" :index="item.name">
      <template slot="title">
        <i :class="'iconfont icon-r-' + item.icon" style="font-size: 26px;"></i>
        <span style="font-size: 20px;"> {{ item.name }}</span>
      </template>
      <el-menu-item
        v-for="subItem in item.children"
        @click="clickMenu(subItem,item.path + subItem.path)"
        :key="subItem.id"
        :index="subItem.name"
      >
        <i :class="'iconfont icon-r-' + subItem.icon" style="font-size: 24px;"></i>
        <span style="font-size: 18px;"> {{ subItem.name }}</span>
      </el-menu-item>
    </el-submenu>
  </el-menu>
</template>

<style lang="less" scoped>
.el-menu-vertical-demo:not(.el-menu--collapse) {
  width: 200px;
  min-height: 400px;
}

.el-menu {
  height: 100%;
  border: none; /* 无边框 */

  h3 {
    color: #fff;
    text-align: center; /* 字体居中 */
    line-height: 48px; /* 行高 */
  }
}
</style>

<script>
import { mapState } from 'vuex'

export default {
  data () {
    return {
      isCollapse: false // 控制侧边栏是否展开
    }
  },
  methods: {
    clickMenu (menu, path) {
      // 使用编程式，路由导航
      this.$router.push({
        path: path // 点击菜单，进行跳转
      })
      this.$store.commit('tag/ADD_TAG', menu)
    }
  },
  computed: {
    ...mapState('menu', ['menuList']),
    noChildren () {
      return this.menuList.filter(item => item.children.length === 0) // 没有子菜单
    },
    hasChildren () {
      return this.menuList.filter(item => item.children.length > 0) // 有子菜单
    }
  },
  mounted () {
    this.$bus.$on('collapseMenu', () => {
      this.isCollapse = !this.isCollapse
    })
  },
  beforeDestroy () {
    this.$bus.$off('collapseMenu')
  }
}
</script>
