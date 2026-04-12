<script setup>
import {
  HomeFilled,
  Place,
  Cpu,
  MapLocation,
  List,
  Clock,
  Setting,
  ArrowDown
} from '@element-plus/icons-vue'
import { computed, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'

/** 只缓存 RouteShell（它内部用 v-show 切 MapView/Airport/Airspace） */
const cachedViews = computed(() => ['RouteShell'])

/** 全局兜底事件监听：AirportTask2 等处用 window.dispatchEvent 也能生效 */
const router = useRouter()
const handleRouteShellAction = (e) => {
  const evt = e?.detail
  if (!evt || !evt.type) return

  // 统一处理“从 AirportTask2 查看航线” → 进入 /route?view=map
  if (evt.type === 'open-map-from-airport') {
    const d = evt.data || {}
    router.push({
      path: '/route',
      query: {
        view: 'map',
        type: d.mapType || (d.preset3dKey ? '预设航线' : '航点飞行'),
        taskId: d.taskId || '',
        startStation: d.startStation || '',
        preset3d: d.preset3dKey || d.preset || '',
        restore: (typeof d.restore === 'boolean' ? d.restore : !d.preset3dKey) ? '1' : '0',
        lock: (typeof d.lock === 'boolean' ? d.lock : !!d.preset3dKey) ? '1' : '0'
      }
    })
    return
  }

  // 进入空域编辑
  if (evt.type === 'open-map-from-airspace' || evt.type === 'open-airspace') {
    router.push({ path: '/route', query: { view: 'airspace' } })
    return
  }

  // 其它：回到任务编排
  if (evt.type === 'open-airport-task') {
    router.push({ path: '/route', query: { view: 'airport' } })
    return
  }
}

onMounted(() => {
  window.addEventListener('route-shell-action', handleRouteShellAction)
})
onUnmounted(() => {
  window.removeEventListener('route-shell-action', handleRouteShellAction)
})
</script>

<template>
  <div class="common-layout">
    <el-container style="height: 100vh">
      <!-- 顶部导航栏 -->
      <el-header class="header">
  <!-- 左侧部分，包含图标和标题 -->
  <div class="header-left">
    <div class="legend-row">
      <img src="/icons/drone.svg"  class="legend-icon" />
    </div>
    <div class="header-title">高速公路无人机控制系统</div>
  </div>
  
  <!-- 右侧部分，包含用户信息 -->
  <div class="user-section">
    <el-dropdown>
      <span class="el-dropdown-link">
        用户<el-icon class="el-icon--right"><arrow-down /></el-icon>
      </span>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item>个人中心</el-dropdown-item>
          <el-dropdown-item>退出登录</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
  </div>
</el-header>


      <el-container>
        <!-- 左侧菜单栏 -->
        <el-aside width="150px" style="background-color: #1E1E2D; color: white;">
          <el-menu
            active-text-color="#ffd04b"
            background-color="#1E1E2D"
            class="el-menu-vertical"
            text-color="#fff"
            style="border-right: none"
            :default-active="$route.path"
            :default-openeds="['route','history','task','device','system']" 
            :router="true"
          >
            <el-menu-item index="/home">
              <el-icon><HomeFilled /></el-icon>
              <span>首页</span>
            </el-menu-item>

            <el-menu-item index="/screen/uav">
              <el-icon><Place /></el-icon>
              <span>无人机大屏</span>
            </el-menu-item>

            <el-sub-menu index="route">
              <template #title>
                <el-icon><MapLocation /></el-icon>
                <span>航线空城</span>
              </template>
              <el-menu-item index="/route/plan">航线任务</el-menu-item>
              <el-menu-item index="/route/airspace">空域管理</el-menu-item>
            </el-sub-menu>

            <el-sub-menu index="history">
              <template #title>
                <el-icon><Clock /></el-icon>
                <span>成果数据</span>
              </template>
              <el-menu-item index="/history/airport">历史任务</el-menu-item>
              <el-menu-item index="/history/stats">数据统计</el-menu-item>
              <!-- <el-menu-item index="/history/statistics">成果统计</el-menu-item> -->
            </el-sub-menu>

            <el-sub-menu index="task">
              <template #title>
                <el-icon><List /></el-icon>
                <span>任务管理</span>
              </template>
              <el-menu-item index="/task/airport">每日任务</el-menu-item>
              <!-- <el-menu-item index="/task/completion">任务状态</el-menu-item> -->
            </el-sub-menu>

            <el-sub-menu index="device">
              <template #title>
                <el-icon><Cpu /></el-icon>
                <span>设备管理</span>
              </template>
              <el-menu-item index="/device/airport">机场设备</el-menu-item>
              <el-menu-item index="/device/drone">无人机</el-menu-item>
            </el-sub-menu>

            <el-sub-menu index="system">
              <template #title>
                <el-icon><Setting /></el-icon>
                <span>系统管理</span>
              </template>
              <el-menu-item index="/system/user">用户管理</el-menu-item>
              <el-menu-item index="/system/role">角色管理</el-menu-item>
            </el-sub-menu>
          </el-menu>
        </el-aside>

        <!-- 主内容区 -->
        <el-main style="padding: 0px; background-color: #f0f2f5;">
          <router-view v-slot="{ Component, route }">
            <!-- 只缓存 RouteShell；用 route.name 作为 key，避免 query 变化导致销毁重建 -->
            <keep-alive :include="cachedViews">
              <component :is="Component" :key="route.name" />
            </keep-alive>
          </router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<style lang="css">
html, body {
  height: 100%;
  margin: 0;
}

.common-layout {
  height: 100%;
}

.el-container {
  height: 100%;
}

.el-menu-vertical {
  height: 100%;
}

.el-main {
  height: 100%;
  overflow: auto; /* 添加滚动条 */
}

.el-dropdown-link {
  display: flex;
  align-items: center;
}

/* 子菜单缩进样式 */
.el-menu-item,
.el-sub-menu__title {
  display: flex;
  align-items: center;
  font-family: "Roboto", "Montserrat", sans-serif;
  transition: all 0.3s ease;



}

.el-menu-item .el-icon,
.el-sub-menu__title .el-icon {
  flex-shrink: 0; /* 防止图标被压缩 */
  margin-right: 10px;
}

/* header 样式 */
.header {
  background-color: #1E1E2D;
  color: white;
  display: flex;
  justify-content: space-between; /* 让左右两部分分别靠左和靠右 */
  align-items: center;
  padding: 10px;
}

/* 左侧部分的容器 */
.header-left {
  display: flex;
  align-items: center;
  justify-content: flex-start; /* 左对齐 */
}

/* 图标和文本行样式 */
.legend-row {
  display: flex;
  align-items: center;
  margin-right: 10px; /* 图标和标题之间的间距 */
}

/* 图标样式 */
.legend-icon {
  width: 35px; /* 图标大小 */
  height: 35px; /* 图标大小 */
  margin-left: 10px; /* 图标和文本之间的间距 */
}

/* 标题样式 */
.header-title {
  font-size: 24px;
  font-weight: bold;
  margin-left: 5px; /* 图标和标题之间的间距 */
}

/* 右侧用户菜单部分 */
.user-section {
  margin-left: auto; /* 将用户部分推到右边 */
}

/* 下拉菜单链接样式 */
.el-dropdown-link {
  color: white;
  cursor: pointer;
  
}

/* 下拉菜单图标样式 */
.el-icon--right {
  margin-left: 20px;
}

/* 子菜单标题（一级项） */
.el-sub-menu__title {
  background-color: #1E1E2D !important; 
  color: #fff !important;              
  font-weight: 700;                    
  font-size: 16px;                      
  padding: 10px 12px !important;        
  transition: all 0.3s ease;
}

.el-sub-menu__title:hover {
  background-color: #2a2a3a !important;
}


.el-menu-item:hover {
  background-color: #344055 !important;
  color: #fff !important;
}


.el-menu-item.is-active {
  color: #ffd92e !important;
  font-weight: bold;
}


</style>
