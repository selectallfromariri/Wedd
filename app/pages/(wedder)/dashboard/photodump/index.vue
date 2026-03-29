<script setup lang="ts">
definePageMeta({ layout: 'dashboard-layout' })

interface PhotoItem {
  id: number
  image_url: string
  created_at?: string
  visitor_id?: number
}

function getToken() {
  return import.meta.client ? (localStorage.getItem('token') ?? '') : ''
}

const photos = ref<PhotoItem[]>([])
const loading = ref(true)
const error = ref('')

async function fetchGallery() {
  loading.value = true
  error.value = ''
  try {
    const token = getToken()
    if (!token) {
       error.value = 'Not authenticated'
       return
    }
    
    // 1. Get the current wedder's wedding code
    const wedRes = await $fetch('/wedding/show', {
      baseURL: useRuntimeConfig().public.apiBase,
      headers: { Accept: 'application/json', Authorization: `Bearer ${token}` }
    }) as any
    const weddingCode = wedRes?.data?.wedding_code

    if (!weddingCode) {
       error.value = 'Create a wedding first to start receiving photos! Go back to the Dashboard to create your wedding.'
       return
    }

    // 2. Fetch the photos uploaded by visitors
    const photoRes = await $fetch(`/photo/index/${weddingCode}`, {
        method: 'GET',
        baseURL: useRuntimeConfig().public.apiBase,
        headers: { Accept: 'application/json', Authorization: `Bearer ${token}` },
    }) as any
    
    photos.value = Array.isArray(photoRes?.data) ? photoRes.data.reverse() : []

  } catch (err: any) {
    error.value = err?.data?.message || err?.message || 'Failed to load photo dump. Ensure you have created a wedding.'
  } finally {
    loading.value = false
  }
}

onMounted(fetchGallery)
</script>

<template>
  <div class="photodump-page">
    <div class="page-header">
      <NuxtLink to="/dashboard" class="btn-back">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M19 12H5M12 5l-7 7 7 7"/></svg>
        Back to Dashboard
      </NuxtLink>
      <h1 class="page-title">Visitor Photo Dump ✨</h1>
      <p class="page-sub">All the wonderful memories captured and uploaded by your guests.</p>
    </div>

    <!-- Loading -->
    <div v-if="loading" class="state-container">
      <div class="spinner"></div>
      <p>Loading your beautiful moments...</p>
    </div>

    <!-- Error / Empty Wedding -->
    <div v-else-if="error" class="state-container error-state">
      <p>{{ error }}</p>
    </div>

    <!-- Empty Photos -->
    <div v-else-if="photos.length === 0" class="state-container empty-state">
      <div class="empty-icon">📷</div>
      <h3>No photos yet!</h3>
      <p>When guests upload photos to your wedding from their invitation, they will appear right here.</p>
    </div>

    <!-- Photo Gallery Grid -->
    <div v-else class="photo-grid">
      <div v-for="p in photos" :key="p.id" class="photo-card">
         <img :src="p.image_url" alt="Wedding Guest Photo" loading="lazy" />
      </div>
    </div>

  </div>
</template>

<style scoped>
.photodump-page {
  font-family: 'DM Sans', sans-serif;
  min-height: 100vh;
  padding: 2.5rem 2rem;
  max-width: 1000px;
}

.page-header { margin-bottom: 2rem; }
.btn-back { display: inline-flex; align-items: center; gap: 6px; padding: 0.5rem 1rem; border-radius: 8px; background: #fff8ee; color: #7c5c36; text-decoration: none; font-size: 13px; font-weight: 500; border: 1px solid #e8d5b5; margin-bottom: 2rem; transition: background 0.15s; }
.btn-back:hover { background: #fff0d6; }
.page-title { font-family: 'Cormorant Garamond', Georgia, serif; font-size: 2.5rem; font-weight: 400; font-style: italic; color: #2c2416; margin: 0 0 0.2rem; line-height: 1.2; }
.page-sub { color: #7c6448; margin: 0; font-size: 14.5px; }

.state-container { display: flex; flex-direction: column; align-items: center; justify-content: center; background: #faf8f4; border: 1px dashed #e8d5b5; border-radius: 16px; min-height: 300px; gap: 1rem; color: #7c6448; text-align: center; padding: 2rem; margin-top: 2rem;}
.empty-icon { font-size: 3.5rem; margin-bottom: 0.5rem; }
.empty-state h3 { font-family: 'Cormorant Garamond', Georgia, serif; font-size: 1.8rem; color: #2c2416; margin: 0; font-style: italic; font-weight: 500;}
.empty-state p { margin: 0; max-width: 380px; font-size: 14px; line-height: 1.5; color: #a8967c; }

.error-state { border-color: #fecaca; background: #fff1f1; color: #dc2626; }

/* Spinner */
.spinner { width: 36px; height: 36px; border: 3px solid #e8d5b5; border-top-color: #c4a76b; border-radius: 50%; animation: spin 0.8s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }

/* Gallery */
.photo-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1.5rem;
  margin-top: 2rem;
}
@media (min-width: 650px) { .photo-grid { grid-template-columns: repeat(3, 1fr); } }
@media (min-width: 900px) { .photo-grid { grid-template-columns: repeat(4, 1fr); } }

.photo-card {
  width: 100%; aspect-ratio: 4 / 5;
  border-radius: 14px; overflow: hidden;
  background: #f5f0e6;
  border: 1px solid #efe5d4;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}
.photo-card:hover { transform: translateY(-4px); box-shadow: 0 10px 24px rgba(168,134,90,0.15); }
.photo-card img { width: 100%; height: 100%; object-fit: cover; transition: transform 0.4s ease; }
.photo-card:hover img { transform: scale(1.05); }
</style>
