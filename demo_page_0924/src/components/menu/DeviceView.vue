<!-- src/components/menu/DeviceView.vue -->
<script setup>
</script>

<template>
  <router-view v-slot="{ Component }">
    <keep-alive include="DeviceAirport,DeviceDrone">
      <component :is="Component" />
    </keep-alive>
  </router-view>
</template>
<style scoped>
</style>
