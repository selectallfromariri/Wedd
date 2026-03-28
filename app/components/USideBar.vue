<template>
  <aside class="sidebar">
    <div class="brand">
      <div class="logo">W</div>
      <div>
        <p class="title">Wedding Management</p>
      </div>
    </div>

    <nav>
      <ul>
        <li><span class="dot"></span>Dashboard</li>
        <li><span class="dot"></span>Schedule</li>
        <li><span class="dot"></span>Event Card</li>
        <li><span class="dot"></span>Bank QR</li>
        <li><span class="dot"></span>Photo Dump</li>
      </ul>
    </nav>
  </aside>
</template>

<script setup lang="ts"></script>

<style scoped>
.sidebar {
  width: 260px;
  min-height: 100vh;
  background: #faf8f4;
  color: #1f2937;
  display: flex;
  flex-direction: column;
  padding: 20px 14px;
  box-sizing: border-box;
  border-right: 1px solid #e5dfd3;
}
.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
}
.logo {
  height: 40px;
  width: 40px;
  border-radius: 10px;
  display: grid;
  place-items: center;
  background-color: #c4a96b;
  color: #fff;
  font-weight: 800;
}
.title {
  margin: 0;
  font-size: 0.95rem;
  font-weight: 700;
}
.subtitle {
  margin: 0;
  color: #6b7280;
  font-size: 0.78rem;
}
nav ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
nav li {
  border-radius: 10px;
  padding: 10px 12px;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9rem;
  color: #374151;
  cursor: pointer;
  transition: all 0.2s ease;
  border: 1px solid transparent;
}
nav li:hover {
  background: #e9e0d6;
  color: #111827;
}
nav li.active {
  background: #c4a96b;
  color: #fff;
  border-color: #a78b56;
}
.dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #c4a96b;
}
.quick-card {
  margin-top: auto;
  background: #fff;
  border-radius: 12px;
  border: 1px solid #e5dfd3;
  padding: 12px;
  color: #1f2937;
}
.card-title {
  margin: 0;
  font-weight: 700;
  font-size: 0.85rem;
}
.card-body {
  margin: 8px 0 0;
  font-size: 0.78rem;
  color: #4b5563;
}
</style>
