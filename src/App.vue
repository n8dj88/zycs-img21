<!-- Header.vue -->
<template>
<header class="header">
  <div class="logo">
    <h1>{{ title }}</h1>
    <p>{{ desc }}</p>
  </div>
  <!-- 导航栏 -->
  <nav class="nav">
    <RouterLink 
      v-for="item in navList" 
      :key="item.path" 
      :to="item.path"
      class="nav-item"
      active-class="active"
    >
      {{ item.name }}
    </RouterLink>
  </nav>
</header>
</template>

<script setup lang="ts">
import { RouterLink } from 'vue-router'
export interface NavItem {
  name: string
  path: string
}
const props = defineProps<{
  title: string
  desc: string
  navList: NavItem[]
}>()
</script>

<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 24px;
  border-bottom: 1px solid #eee;
}
.nav {
  display: flex;
  gap: 24px;
}
.nav-item {
  color: #333;
  text-decoration: none;
}
.nav-item.active {
  color: #3b82f6;
  font-weight: 600;
}
</style>
