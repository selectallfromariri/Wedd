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
        <li>
          <NuxtLink to="/dashboard" class="nav-link" exact-active-class="nav-link--active">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/></svg>
            Dashboard
          </NuxtLink>
        </li>
        <li>
          <NuxtLink to="/dashboard/eventcard" class="nav-link" active-class="nav-link--active">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="4" width="20" height="16" rx="2"/><path d="M2 9h20"/></svg>
            Event Card
          </NuxtLink>
        </li>
        <li>
          <NuxtLink to="/dashboard/bankqr" class="nav-link" active-class="nav-link--active">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="3" height="3"/></svg>
            Bank QR
          </NuxtLink>
        </li>
        <li>
          <NuxtLink to="/dashboard/photodump" class="nav-link" active-class="nav-link--active">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="8.5" cy="8.5" r="1.5"/><path d="M21 15l-5-5L5 21"/></svg>
            Photo Dump
          </NuxtLink>
        </li>
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
  gap: 4px;
}
nav li {
  border-radius: 10px;
  overflow: hidden;
}
.nav-link {
  display: flex;
  align-items: center;
  gap: 9px;
  padding: 10px 12px;
  font-size: 0.9rem;
  color: #374151;
  text-decoration: none;
  border-radius: 10px;
  border: 1px solid transparent;
  transition: background 0.2s, color 0.2s;
  width: 100%;
  box-sizing: border-box;
}
.nav-link:hover {
  background: #e9e0d6;
  color: #111827;
}
.nav-link--active {
  background: #c4a96b;
  color: #fff;
  border-color: #a78b56;
}
.nav-link--active:hover {
  background: #b5975a;
  color: #fff;
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
