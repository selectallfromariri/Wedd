<script setup lang="ts">
definePageMeta({ layout: 'visitor-layout' })

interface Wedding {
  id: number
  wedding_code: string
  bride_name: string
  groom_name: string
  date: string
  venue: string
  cover_photo?: string
}

interface Rsvp {
  attendance: 'attending' | 'not_attending'
  note?: string
}

interface TentativeItem {
  id: number
  time: string
  title: string
  note?: string
  published: boolean
}

interface BankQr {
  bank_name: string
  account_name: string
  account_number: string
  qr_image?: string
  is_published: boolean
}

interface PhotoItem {
  id: number
  image_url: string
  visitor_id: number
  created_at?: string
}

const route  = useRoute()
const router = useRouter()
const code   = route.params.wedding_code as string

function getToken() { return import.meta.client ? (localStorage.getItem('token') ?? '') : '' }

const wedding    = ref<Wedding | null>(null)
const rsvp       = ref<Rsvp | null>(null)
const tentatives = ref<TentativeItem[]>([])
const bankQr     = ref<BankQr | null>(null)
const photos     = ref<PhotoItem[]>([])
const loading    = ref(true)
const error      = ref('')

// Photo upload
const photoFile      = ref<File | null>(null)
const uploading      = ref(false)
const uploadSuccess  = ref('')
const uploadError    = ref('')
const fileInputRef   = ref<HTMLInputElement | null>(null)

function formatDate(d: string) {
  return new Date(d).toLocaleDateString('en-GB', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' })
}

function countdown(d: string) {
  const diff = Math.ceil((new Date(d).getTime() - Date.now()) / 86400000)
  if (diff < 0) return `${Math.abs(diff)} days ago`
  if (diff === 0) return 'Today! 🎉'
  return `${diff} days to go`
}

function normalizeStatus(s: string) {
  const v = (s || '').toLowerCase()
  if (v === 'done' || v === 'completed') return 'done'
  if (v === 'upcoming' || v === 'scheduled') return 'upcoming'
  return 'pending'
}

const headers = computed(() => ({
  Accept: 'application/json',
  Authorization: `Bearer ${getToken()}`,
}))

async function loadAll() {
  loading.value = true
  error.value = ''

  const token = getToken()
  if (!token) {
    router.replace(`/?redirect=/visitor/dashboard/${code}`)
    return
  }

  try {
    // 1. Fetch wedding details (+ tentatives + bank_qr)
    const res = await $fetch(`/wedding/code/${code}`, {
      method: 'GET',
      baseURL: useRuntimeConfig().public.apiBase,
      headers: {
        Accept: 'application/json',
      },
    }) as any

    const data = res?.data ?? res
    wedding.value       = data
    const allTentatives = Array.isArray(data?.tentatives) ? data.tentatives : []
    tentatives.value    = allTentatives.filter((t: any) => t.is_published == 1 || t.is_published === true)
    
    bankQr.value        = data?.bank_qr?.is_published ? data.bank_qr : null

    // 2. Fetch user's RSVP
    try {
      const rsvpRes = await $fetch(`/visitor/rsvp/${code}`, {
        method: 'GET',
        baseURL: useRuntimeConfig().public.apiBase,
        headers: { Accept: 'application/json', Authorization: `Bearer ${token}` },
      }) as any
      rsvp.value = rsvpRes?.data ?? null
    } catch {
      rsvp.value = null // RSVP not found or failed
    }

    // 3. Fetch Photos (The Photo Dump)
    try {
      const photoRes = await $fetch(`/photo/index/${code}`, {
        method: 'GET',
        baseURL: useRuntimeConfig().public.apiBase,
        headers: { Accept: 'application/json', Authorization: `Bearer ${token}` },
      }) as any
      // We assume data is an array; reverse to show newest first!
      photos.value = Array.isArray(photoRes?.data) ? photoRes.data.reverse() : []
    } catch {
      photos.value = []
    }

  } catch (e: any) {
    error.value = e?.data?.message || 'Could not load wedding details'
  } finally {
    loading.value = false
  }
}

function onFileChange(e: Event) {
  const f = (e.target as HTMLInputElement).files?.[0]
  if (f) photoFile.value = f
}

const previewUrl = computed(() => photoFile.value ? URL.createObjectURL(photoFile.value) : '')

async function uploadPhoto() {
  if (!photoFile.value) return
  uploading.value = true
  uploadSuccess.value = ''
  uploadError.value   = ''
  try {
    const body = new FormData()
    body.append('image_url', photoFile.value)  // must match backend validation rule
    await $fetch(`/photo/store/${code}`, {
      baseURL: useRuntimeConfig().public.apiBase,
      method:  'POST',
      headers: { Authorization: `Bearer ${getToken()}` },
      body,
    })
    uploadSuccess.value = 'Photo uploaded! 🎉'
    photoFile.value   = null
    if (fileInputRef.value) fileInputRef.value.value = ''
    
    // Refresh photos so the new one appears immediately
    try {
      const photoRes = await $fetch(`/photo/index/${code}`, {
        method: 'GET',
        baseURL: useRuntimeConfig().public.apiBase,
        headers: { Accept: 'application/json', Authorization: `Bearer ${getToken()}` },
      }) as any
      photos.value = Array.isArray(photoRes?.data) ? photoRes.data.reverse() : []
    } catch {}
  } catch (e: any) {
    uploadError.value = e?.data?.message || 'Upload failed'
  } finally {
    uploading.value = false
  }
}

onMounted(loadAll)
</script>

<template>
  <div class="vd-page">

    <!-- Top nav -->
    <header class="top-nav">
      <div class="nav-brand">
        <div class="nav-logo">
          <img src="~/assets/images/{C37CA420-6487-43AE-910C-8FFFE00DE730}.png" alt="Logo" class="logo-img" />
        </div>
        <span class="nav-title">WeddingAR</span>
      </div>
      <NuxtLink :to="`/wedding/code/${code}`" class="btn-nav-back">
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M19 12H5M12 5l-7 7 7 7"/></svg>
        Invitation
      </NuxtLink>
    </header>

    <!-- Loading -->
    <div v-if="loading" class="page-center">
      <div class="big-spinner" />
    </div>

    <!-- Error -->
    <div v-else-if="error" class="page-center">
      <div class="err-card">
        <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="#dc2626" stroke-width="1.5"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
        <h2>Something went wrong</h2>
        <p>{{ error }}</p>
      </div>
    </div>

    <template v-else-if="wedding">

      <!-- 1. Wedding Hero -->
      <section class="hero-section" :style="wedding.cover_photo ? `--bg:url('${wedding.cover_photo}')` : ''">
        <div class="hero-overlay" />
        <div class="hero-content">
          <p class="hero-eyebrow">Wedding Day</p>
          <h1 class="hero-couple">{{ wedding.bride_name }} <span class="hero-amp">&amp;</span> {{ wedding.groom_name }}</h1>
          <div class="hero-meta">
            <div class="hero-meta-item">
              <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="4" width="18" height="18" rx="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></svg>
              {{ formatDate(wedding.date) }}
            </div>
            <div class="hero-meta-item">
              <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/></svg>
              {{ wedding.venue }}
            </div>
            <div class="hero-meta-item hero-meta-item--pill">
              <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/></svg>
              {{ countdown(wedding.date) }}
            </div>
          </div>
        </div>
      </section>

      <div class="content-area">

        <!-- 2. RSVP Status Badge -->
        <div v-if="rsvp" class="status-badge" :class="rsvp.attendance === 'attending' ? 'status-badge--attending' : 'status-badge--notattending'">
          <span class="status-emoji">{{ rsvp.attendance === 'attending' ? '' : '' }}</span>
          <div>
            <p class="status-label">Your RSVP</p>
            <p class="status-value">{{ rsvp.attendance === 'attending' ? 'You are attending!' : 'You\'re not attending' }}</p>
            <p v-if="rsvp.note" class="status-note">"{{ rsvp.note }}"</p>
          </div>
        </div>

        <!-- 3. Tentative / Schedule -->
        <section v-if="tentatives.length" class="section-card">
          <div class="section-header">
            <p class="section-eyebrow">Timeline</p>
            <h2 class="section-title">Wedding Day Schedule</h2>
          </div>
          <ul class="timeline-list">
            <li v-for="(item, i) in tentatives" :key="item.id" class="timeline-item">
              <div class="tl-meta">
                <div class="tl-dot" />
                <div v-if="i < tentatives.length - 1" class="tl-line" />
              </div>
              <div class="tl-card">
                <p class="tl-time">{{ item.time }}</p>
                <p class="tl-title">{{ item.title }}</p>
                <p v-if="item.note" class="tl-note">{{ item.note }}</p>
              </div>
            </li>
          </ul>
        </section>

        <!-- 4. Bank QR -->
        <section v-if="bankQr" class="section-card">
          <div class="section-header">
            <p class="section-eyebrow">Gift</p>
            <h2 class="section-title">Send a Gift 💝</h2>
            <p class="section-sub">Transfer directly to the couple's account or scan the QR code.</p>
          </div>
          <div class="bank-details">
            <div class="bank-info">
              <div class="bank-row">
                <span class="bank-label">Bank</span>
                <span class="bank-value">{{ bankQr.bank_name }}</span>
              </div>
              <div class="bank-row">
                <span class="bank-label">Account Name</span>
                <span class="bank-value">{{ bankQr.account_name }}</span>
              </div>
              <div class="bank-row">
                <span class="bank-label">Account No.</span>
                <span class="bank-value bank-value--mono">{{ bankQr.account_number }}</span>
              </div>
            </div>
            <div v-if="bankQr.qr_image" class="qr-frame">
              <img :src="bankQr.qr_image" alt="QR Code" class="qr-img" />
            </div>
          </div>
        </section>

        <!-- 5. Photo Upload -->
        <section class="section-card">
          <div class="section-header">
            <p class="section-eyebrow">Memories</p>
            <h2 class="section-title">Share a Photo 📸</h2>
            <p class="section-sub">Upload your favourite moments from the wedding!</p>
          </div>

          <div class="upload-area" @click="fileInputRef?.click()" @dragover.prevent @drop.prevent="e => { const f = e.dataTransfer?.files?.[0]; if(f) photoFile = f }">
            <input ref="fileInputRef" type="file" accept="image/*" class="upload-input" @change="onFileChange" />
            <div v-if="photoFile" class="upload-preview">
              <img :src="previewUrl" alt="Preview" class="upload-preview-img" />
              <p class="upload-filename">{{ photoFile.name }}</p>
            </div>
            <div v-else class="upload-placeholder">
              <svg width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="#c4a96b" stroke-width="1.5"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
              <p class="upload-hint">Click or drag a photo here</p>
              <p class="upload-hint-sub">JPG, PNG, HEIC supported</p>
            </div>
          </div>

          <p v-if="uploadError"   class="feedback-error">{{ uploadError }}</p>
          <p v-if="uploadSuccess" class="feedback-success">{{ uploadSuccess }}</p>

          <button v-if="photoFile" class="btn-upload" :disabled="uploading" @click="uploadPhoto">
            <span v-if="uploading" class="spinner spinner--white" />
            <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
            {{ uploading ? 'Uploading…' : 'Upload Photo' }}
          </button>
        </section>

        <!-- 6. Photo Dump Gallery -->
        <section v-if="photos.length" class="section-card">
          <div class="section-header">
            <p class="section-eyebrow">Gallery</p>
            <h2 class="section-title">Photo Dump ✨</h2>
            <p class="section-sub">Moments captured and shared by everyone.</p>
          </div>
          <div class="photo-grid">
            <div v-for="p in photos" :key="p.id" class="photo-item">
              <!-- Using useRuntimeConfig().public.apiBase or assuming the backend returns full URLs -->
              <!-- If image_url is just a path, it might need to be prepended by your domain -->
              <img :src="p.image_url" alt="Wedding Photo" loading="lazy" />
            </div>
          </div>
        </section>

      </div>
    </template>
  </div>
</template>

<style scoped>
/* Nav */
.top-nav {
  position: sticky; top: 0; z-index: 50;
  display: flex; align-items: center; justify-content: space-between;
  padding: 0.85rem 2rem;
  background: rgba(250,248,244,0.92);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid #ece3d8;
}
.nav-brand { display: flex; align-items: center; gap: 10px; }
.nav-logo {
  width: 32px; height: 32px; border-radius: 8px;
  background: transparent;
  display: flex; align-items: center; justify-content: center; overflow: hidden;
}
.logo-img { width: 100%; height: 100%; object-fit: contain; }
.nav-title { font-weight: 700; font-size: 0.92rem; color: #2c2416; }
.btn-nav-back {
  display: inline-flex; align-items: center; gap: 5px;
  padding: 0.35rem 0.8rem; font-size: 12.5px; font-weight: 500;
  color: #7c5c36; background: #fff8ee; border: 1px solid #e8d5b5;
  border-radius: 8px; text-decoration: none; transition: background 0.15s;
}
.btn-nav-back:hover { background: #fff0d6; }

/* Layout */
.page-center { min-height: 60vh; display: flex; align-items: center; justify-content: center; }
.big-spinner { width: 38px; height: 38px; border: 3px solid #e8d5b5; border-top-color: #c4a76b; border-radius: 50%; animation: spin 0.8s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }
.err-card { text-align: center; display: flex; flex-direction: column; align-items: center; gap: 0.75rem; max-width: 340px; padding: 2rem; }
.err-card h2 { margin: 0; font-size: 1.2rem; color: #2c2416; }
.err-card p  { margin: 0; font-size: 13px; color: #7c6448; }
.vd-page { display: flex; flex-direction: column; }

/* Hero */
.hero-section {
  position: relative; min-height: 55vh;
  background-image: var(--bg, url('https://images.unsplash.com/photo-1519225421980-715cb0215aed?w=1200&q=80'));
  background-size: cover; background-position: center;
  display: flex; align-items: flex-end;
}
.hero-overlay { position: absolute; inset: 0; background: linear-gradient(to top, rgba(20,14,6,0.82) 0%, rgba(20,14,6,0.25) 55%, transparent 100%); }
.hero-content { position: relative; padding: 2.5rem 2rem; max-width: 700px; margin: 0 auto; width: 100%; }
.hero-eyebrow { margin: 0 0 0.3rem; font-size: 11px; text-transform: uppercase; letter-spacing: 0.18em; color: #e8c87a; font-weight: 600; }
.hero-couple { margin: 0 0 1rem; font-family: 'Cormorant Garamond', Georgia, serif; font-size: clamp(2.2rem, 5vw, 3.8rem); font-weight: 400; font-style: italic; color: #fff; line-height: 1.05; }
.hero-amp { color: #e8c87a; font-style: normal; margin: 0 0.1em; }
.hero-meta { display: flex; flex-wrap: wrap; gap: 0.85rem; align-items: center; }
.hero-meta-item { display: flex; align-items: center; gap: 6px; font-size: 13px; color: rgba(255,255,255,0.87); }
.hero-meta-item--pill { background: rgba(196,167,107,0.25); border: 1px solid rgba(196,167,107,0.5); border-radius: 999px; padding: 0.2rem 0.7rem; color: #e8c87a; font-weight: 600; }

/* Content */
.content-area { display: flex; flex-direction: column; gap: 1.25rem; max-width: 700px; margin: 0 auto; width: 100%; padding: 1.75rem 1.5rem 3rem; box-sizing: border-box; }

/* RSVP badge */
.status-badge {
  display: flex; align-items: center; gap: 1rem;
  padding: 1.1rem 1.25rem; border-radius: 14px; border: 1px solid #ece3d8;
}
.status-badge--attending { background: #f0fdf4; border-color: #bbf7d0; }
.status-badge--notattending { background: #fdf4ff; border-color: #f0abfc; }
.status-emoji { font-size: 2rem; line-height: 1; flex-shrink: 0; }
.status-label { margin: 0; font-size: 10px; text-transform: uppercase; letter-spacing: 0.08em; color: #a8967c; font-weight: 600; }
.status-value { margin: 0.15rem 0 0; font-size: 15px; font-weight: 600; color: #2c2416; }
.status-note { margin: 0.2rem 0 0; font-size: 12.5px; color: #7c6448; font-style: italic; }

/* Section cards */
.section-card { background: #fffdf8; border: 1px solid #ece3d8; border-radius: 16px; padding: 1.5rem; }
.section-header { margin-bottom: 1.1rem; }
.section-eyebrow { margin: 0 0 0.2rem; font-size: 10px; text-transform: uppercase; letter-spacing: 0.09em; color: #b89b6a; font-weight: 600; }
.section-title { margin: 0 0 0.3rem; font-family: 'Cormorant Garamond', Georgia, serif; font-size: 1.35rem; font-style: italic; font-weight: 400; color: #2c2416; }
.section-sub { margin: 0; font-size: 13px; color: #7c6448; }

/* Timeline */
.timeline-list { list-style: none; margin: 0; padding: 0; }
.timeline-item { display: flex; gap: 0.75rem; margin-bottom: 0.75rem; }
.tl-meta { position: relative; display: flex; flex-direction: column; align-items: center; width: 18px; }
.tl-dot { width: 10px; height: 10px; border-radius: 50%; background: #c4a96b; border: 2px solid #fff; box-shadow: 0 0 0 2px #c4a96b; margin-top: 5px; flex-shrink: 0; }
.tl-line { width: 2px; background: #e3d5c4; flex: 1; margin-top: 5px; border-radius: 999px; }
.tl-card { flex: 1; padding-bottom: 0.5rem; }
.tl-time { margin: 0 0 0.1rem; font-size: 10px; color: #b89b6a; font-weight: 600; text-transform: uppercase; letter-spacing: 0.05em; }
.tl-title { margin: 0; font-size: 14px; font-weight: 500; color: #2c2416; }
.tl-note { margin: 0.2rem 0 0; font-size: 12px; color: #7c6448; }

/* Bank details */
.bank-details { display: flex; gap: 1.5rem; align-items: flex-start; flex-wrap: wrap; }
.bank-info { flex: 1; display: flex; flex-direction: column; gap: 0.75rem; }
.bank-row { display: flex; flex-direction: column; gap: 2px; }
.bank-label { font-size: 10.5px; text-transform: uppercase; letter-spacing: 0.07em; color: #a8967c; font-weight: 600; }
.bank-value { font-size: 15px; font-weight: 500; color: #2c2416; font-family: 'Cormorant Garamond', Georgia, serif; }
.bank-value--mono { font-family: 'DM Mono', 'Courier New', monospace; font-size: 14px; letter-spacing: 0.04em; }
.qr-frame { width: 130px; height: 130px; background: #fff; border: 1px solid #ece3d8; border-radius: 12px; overflow: hidden; flex-shrink: 0; }
.qr-img { width: 100%; height: 100%; object-fit: contain; }

/* Photo upload */
.upload-area {
  border: 2px dashed #ddc98a; border-radius: 14px;
  padding: 2rem 1rem; text-align: center; cursor: pointer;
  transition: border-color 0.18s, background 0.18s; background: #fffbf0; margin-bottom: 0.75rem;
}
.upload-area:hover { border-color: #c4a76b; background: #fff8e6; }
.upload-input { display: none; }
.upload-placeholder { display: flex; flex-direction: column; align-items: center; gap: 0.5rem; }
.upload-hint { margin: 0; font-size: 13.5px; font-weight: 500; color: #7c6040; }
.upload-hint-sub { margin: 0; font-size: 11.5px; color: #b89b6a; }
.upload-preview { display: flex; flex-direction: column; align-items: center; gap: 0.5rem; }
.upload-preview-img { max-height: 180px; border-radius: 10px; object-fit: cover; }
.upload-filename { margin: 0; font-size: 12px; color: #7c6448; }
.feedback-error   { margin: 0.25rem 0 0; font-size: 12px; color: #dc2626; }
.feedback-success { margin: 0.25rem 0 0; font-size: 12px; color: #16a34a; }
.btn-upload {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 0.6rem 1.3rem; font-size: 13.5px; font-weight: 600;
  font-family: inherit; color: #fff;
  background: linear-gradient(135deg, #c4a76b, #a8865a);
  border: none; border-radius: 10px; cursor: pointer;
  box-shadow: 0 2px 10px rgba(168,134,90,0.28);
  transition: opacity 0.18s, transform 0.12s;
}
.btn-upload:hover:not(:disabled) { opacity: 0.9; transform: translateY(-1px); }
.btn-upload:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }

.spinner { width: 12px; height: 12px; border: 2px solid rgba(255,255,255,0.35); border-top-color: #fff; border-radius: 50%; animation: spin 0.7s linear infinite; display: inline-block; }

/* Photo Gallery */
.photo-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  margin-top: 1rem;
}
@media (min-width: 600px) {
  .photo-grid { grid-template-columns: repeat(3, 1fr); }
}
.photo-item {
  width: 100%; aspect-ratio: 4 / 5;
  border-radius: 12px; overflow: hidden;
  background: #f5f0e6;
  border: 1px solid #efe5d4;
}
.photo-item img {
  width: 100%; height: 100%; object-fit: cover;
  transition: transform 0.35s ease;
}
.photo-item:hover img { transform: scale(1.05); }
</style>
