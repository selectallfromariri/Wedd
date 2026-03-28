<script setup lang="ts"></script>

<template>
  <div class="dashboard-layout">
    <USideBar />
    <main class="dashboard-content">
      <slot />
    </main>
  </div>
</template>

<style scoped>
.dashboard-layout {
  display: flex;
  min-height: 100vh;
  background: #faf8f4;
}
.dashboard-content {
  flex: 1;
  padding: 20px;
  background: #faf8f4;
}
</style>