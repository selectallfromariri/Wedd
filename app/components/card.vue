<script setup lang="ts">
interface Wedding {
  id: number
  bride_name: string
  groom_name: string
  date: string
  venue: string
  cover_photo?: string
  is_published?: boolean
}

const wedding = ref<Wedding | null>(null)
const loading = ref(true)
const fetchError = ref('')

// Create form
const showForm = ref(false)
const saving = ref(false)
const saveError = ref('')
const form = reactive({
  bride_name: '',
  groom_name: '',
  date: '',
  venue: '',
  cover_photo: '',
})

// Publish
const publishing = ref(false)
const publishSuccess = ref('')
const publishError = ref('')

function getToken() {
  return localStorage.getItem('token') ?? ''
}

function computeCountdown(dateStr: string): string {
  const target = new Date(dateStr)
  const now = new Date()
  const diff = Math.ceil((target.getTime() - now.getTime()) / (1000 * 60 * 60 * 24))
  if (diff < 0) return `${Math.abs(diff)} days ago`
  if (diff === 0) return 'Today!'
  return `${diff} day${diff === 1 ? '' : 's'} to go`
}

function formatDate(dateStr: string): string {
  const d = new Date(dateStr)
  return d.toLocaleDateString('en-GB', { day: 'numeric', month: 'long', year: 'numeric' })
}

async function fetchWedding() {
  loading.value = true
  fetchError.value = ''
  try {
    const token = getToken()
    if (!token) { fetchError.value = 'Not authenticated'; return }
    const res = await $fetch('/wedding/show', {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'GET',
      headers: { Accept: 'application/json', Authorization: `Bearer ${token}` },
    }) as any
    wedding.value = res?.data ?? null
  } catch (err: any) {
    if (err?.status === 404 || err?.data?.message === 'Wedding not found') {
      wedding.value = null
    } else {
      fetchError.value = err?.data?.message || err?.message || 'Failed to load wedding'
    }
  } finally {
    loading.value = false
  }
}

async function handleCreate() {
  saving.value = true
  saveError.value = ''
  try {
    const token = getToken()
    if (!token) { saveError.value = 'Not authenticated'; return }
    const res = await $fetch('/wedding/store', {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`,
      },
      body: {
        bride_name: form.bride_name,
        groom_name: form.groom_name,
        date: form.date,
        venue: form.venue,
        cover_photo: form.cover_photo || undefined,
      },
    }) as any
    wedding.value = res?.data ?? null
    showForm.value = false
  } catch (err: any) {
    saveError.value = err?.data?.message || err?.data?.error || err?.message || 'Failed to create wedding'
  } finally {
    saving.value = false
  }
}

async function handlePublish() {
  publishing.value = true
  publishSuccess.value = ''
  publishError.value = ''
  try {
    const token = getToken()
    if (!token) { publishError.value = 'Not authenticated'; return }
    await $fetch('/wedding/publish', {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'POST',
      headers: { Accept: 'application/json', Authorization: `Bearer ${token}` },
    })
    publishSuccess.value = 'Wedding published!'
    if (wedding.value) wedding.value.is_published = true
  } catch (err: any) {
    publishError.value = err?.data?.message || err?.message || 'Failed to publish'
  } finally {
    publishing.value = false
  }
}

onMounted(fetchWedding)
</script>

<template>
  <!-- Loading -->
  <div v-if="loading" class="hero-card hero-card--loading">
    <div class="skeleton-img" />
    <div class="skeleton-content">
      <div class="skeleton-line skeleton-line--sm" />
      <div class="skeleton-line skeleton-line--lg" />
      <div class="skeleton-line skeleton-line--md" />
    </div>
  </div>

  <!-- Error -->
  <div v-else-if="fetchError" class="hero-card hero-card--error">
    <p class="error-msg">{{ fetchError }}</p>
  </div>

  <!-- Empty state — no wedding yet -->
  <div v-else-if="!wedding" class="hero-card hero-card--empty">
    <div v-if="!showForm" class="empty-state">
      <div class="empty-icon">
        <svg width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="#c4a96b" stroke-width="1.5">
          <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/>
        </svg>
      </div>
      <h2 class="empty-title">No Wedding Data Yet</h2>
      <p class="empty-sub">Start by adding your wedding details to personalise your dashboard.</p>
      <button class="btn-create" @click="showForm = true">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
        Add Wedding Details
      </button>
    </div>

    <!-- Create form -->
    <form v-else class="create-form" @submit.prevent="handleCreate">
      <div class="create-form-header">
        <h3 class="create-form-title">Add Wedding Details</h3>
        <button type="button" class="create-form-close" @click="showForm = false">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
        </button>
      </div>

      <div class="form-grid">
        <div class="field-group">
          <label class="field-label" for="bride-name">Bride's Name</label>
          <input id="bride-name" v-model="form.bride_name" class="field-input" placeholder="e.g. Emma" required />
        </div>
        <div class="field-group">
          <label class="field-label" for="groom-name">Groom's Name</label>
          <input id="groom-name" v-model="form.groom_name" class="field-input" placeholder="e.g. Liam" required />
        </div>
        <div class="field-group">
          <label class="field-label" for="wedding-date">Wedding Date</label>
          <input id="wedding-date" v-model="form.date" class="field-input" type="date" required />
        </div>
        <div class="field-group">
          <label class="field-label" for="wedding-venue">Venue</label>
          <input id="wedding-venue" v-model="form.venue" class="field-input" placeholder="e.g. Grand Ballroom" required />
        </div>
        <div class="field-group field-group--full">
          <label class="field-label" for="cover-photo">Cover Photo URL <span class="field-optional">optional</span></label>
          <input id="cover-photo" v-model="form.cover_photo" class="field-input" placeholder="https://..." />
        </div>
      </div>

      <p v-if="saveError" class="save-error">{{ saveError }}</p>

      <div class="form-actions">
        <button type="submit" class="btn-save" :disabled="saving">
          <span v-if="saving" class="btn-spinner" />
          {{ saving ? 'Saving…' : 'Save Wedding' }}
        </button>
        <button type="button" class="btn-cancel" @click="showForm = false">Cancel</button>
      </div>
    </form>
  </div>

  <!-- Wedding card with real data -->
  <div v-else class="hero-card">
    <div class="hero-image">
      <img
        :src="wedding.cover_photo || 'https://images.unsplash.com/photo-1519225421980-715cb0215aed?w=600&q=80'"
        :alt="`${wedding.bride_name} & ${wedding.groom_name} wedding`"
      />
      <div class="image-overlay" />
    </div>

    <div class="hero-content">
      <div class="bolt-badge">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z" fill="currentColor"/>
        </svg>
      </div>

      <p class="eyebrow">The Big Day</p>
      <h1 class="couple-name">{{ wedding.bride_name }} &amp; {{ wedding.groom_name }}</h1>

      <div class="meta-row">
        <div class="meta-item">
          <span class="meta-label">Countdown</span>
          <span class="meta-value">{{ computeCountdown(wedding.date) }}</span>
        </div>
        <div class="meta-item">
          <span class="meta-label">Date</span>
          <span class="meta-value">{{ formatDate(wedding.date) }}</span>
        </div>
        <div class="meta-item">
          <span class="meta-label">Venue</span>
          <span class="meta-value">{{ wedding.venue }}</span>
        </div>
      </div>

      <div class="publish-area">
        <span v-if="wedding.is_published" class="published-badge">
          <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6L9 17l-5-5"/></svg>
          Published
        </span>
        <p v-if="publishError" class="pub-error-inline">{{ publishError }}</p>
        <p v-if="publishSuccess" class="pub-success-inline">{{ publishSuccess }}</p>
        <button v-if="!wedding.is_published" class="publish-btn" :disabled="publishing" @click="handlePublish">
          <span v-if="publishing" class="publish-spinner" />
          <template v-else>
            Publish Event Card
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M5 12h14M12 5l7 7-7 7"/>
            </svg>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* ── Base card ── */
.hero-card {
  display: grid;
  grid-template-columns: 1fr 1fr;
  overflow: hidden;
  background: #faf8f4;
  min-height: 280px;
  border: 1px solid #e9e0d6;
  border-radius: 16px;
}

/* ── Image pane ── */
.hero-image {
  position: relative;
  overflow: hidden;
  background: #f6f0e6;
}
.hero-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.6s ease;
  filter: saturate(0.95);
}
.hero-card:hover .hero-image img { transform: scale(1.03); }
.image-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to right, transparent 70%, #faf8f4);
}

/* ── Content pane ── */
.hero-content {
  padding: 2rem 2rem 2rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  position: relative;
}
.bolt-badge {
  position: absolute;
  top: 1.25rem;
  right: 1.25rem;
  width: 44px;
  height: 44px;
  background: #f0ebe0;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #c4a96b;
}
.eyebrow {
  font-size: 11px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: #c4a96b;
  margin: 0;
  margin-top: 0.25rem;
}
.couple-name {
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 2.8rem;
  font-weight: 400;
  font-style: italic;
  color: #2c2416;
  margin: 0;
  line-height: 1.1;
}
.meta-row { display: flex; gap: 1.5rem; margin-top: 0.75rem; flex-wrap: wrap; }
.meta-item { display: flex; flex-direction: column; gap: 4px; }
.meta-label {
  font-size: 10px;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #a8967c;
  font-family: 'Cormorant Garamond', Georgia, serif;
}
.meta-value {
  font-size: 14px;
  font-weight: 500;
  color: #2c2416;
  font-family: 'Cormorant Garamond', Georgia, serif;
}

/* Publish area */
.publish-area {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
}
.pub-error-inline { margin: 0; font-size: 11.5px; color: #dc2626; }
.pub-success-inline { margin: 0; font-size: 11.5px; color: #16a34a; font-weight: 500; }
.published-badge {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-size: 11.5px;
  font-weight: 600;
  color: #16a34a;
  background: #f0fdf4;
  border: 1px solid #bbf7d0;
  border-radius: 999px;
  padding: 0.25rem 0.65rem;
}
.publish-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: linear-gradient(135deg, #c4a96b 0%, #a8865a 100%);
  color: #fff;
  border: none;
  border-radius: 50px;
  padding: 0.65rem 1.4rem;
  font-size: 13.5px;
  font-weight: 500;
  cursor: pointer;
  width: fit-content;
  transition: opacity 0.2s, transform 0.15s, box-shadow 0.2s;
  font-family: inherit;
  box-shadow: 0 2px 10px rgba(168,134,90,0.28);
}
.publish-btn:hover:not(:disabled) { opacity: 0.9; transform: translateY(-1px); box-shadow: 0 4px 16px rgba(168,134,90,0.35); }
.publish-btn:active:not(:disabled) { transform: translateY(0); }
.publish-btn:disabled { opacity: 0.6; cursor: not-allowed; }
.publish-spinner {
  width: 13px; height: 13px;
  border: 2px solid rgba(255,255,255,0.35);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* ── Loading skeleton ── */
.hero-card--loading {
  display: grid;
  grid-template-columns: 1fr 1fr;
  min-height: 280px;
}
.skeleton-img {
  background: linear-gradient(90deg, #f0ebe0 25%, #e8e0d0 50%, #f0ebe0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.4s ease infinite;
}
.skeleton-content {
  padding: 2rem;
  display: flex;
  flex-direction: column;
  gap: 0.9rem;
  justify-content: center;
}
.skeleton-line {
  border-radius: 6px;
  background: linear-gradient(90deg, #f0ebe0 25%, #e8e0d0 50%, #f0ebe0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.4s ease infinite;
}
.skeleton-line--sm { height: 12px; width: 40%; }
.skeleton-line--lg { height: 28px; width: 80%; }
.skeleton-line--md { height: 14px; width: 55%; }
@keyframes shimmer { to { background-position: -200% 0; } }

/* ── Error state ── */
.hero-card--error {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 160px;
  grid-template-columns: 1fr;
}
.error-msg { color: #dc2626; font-size: 13.5px; margin: 0; }

/* ── Empty state ── */
.hero-card--empty {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 280px;
  grid-template-columns: 1fr;
  padding: 2rem;
}
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.65rem;
  text-align: center;
  max-width: 340px;
}
.empty-icon {
  width: 64px; height: 64px;
  background: #fff8ee;
  border: 1px solid #e8d5b5;
  border-radius: 20px;
  display: flex; align-items: center; justify-content: center;
  margin-bottom: 0.25rem;
}
.empty-title {
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 1.4rem;
  font-style: italic;
  font-weight: 400;
  color: #2c2416;
  margin: 0;
}
.empty-sub { font-size: 13px; color: #7c6448; margin: 0; line-height: 1.55; }
.btn-create {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 0.55rem 1.2rem;
  font-size: 13px; font-weight: 600; font-family: inherit;
  color: #fff;
  background: linear-gradient(135deg, #c4a76b 0%, #a8865a 100%);
  border: none; border-radius: 10px; cursor: pointer;
  margin-top: 0.4rem;
  transition: opacity 0.18s, transform 0.12s;
  box-shadow: 0 2px 10px rgba(168,134,90,0.28);
}
.btn-create:hover { opacity: 0.9; transform: translateY(-1px); }

/* ── Create form ── */
.create-form {
  width: 100%;
  max-width: 560px;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.create-form-header {
  display: flex; align-items: center; justify-content: space-between;
}
.create-form-title {
  margin: 0;
  font-size: 1.05rem;
  font-weight: 600;
  color: #2c2416;
  letter-spacing: -0.01em;
}
.create-form-close {
  width: 28px; height: 28px;
  display: flex; align-items: center; justify-content: center;
  background: #f5ede0; border: none; border-radius: 8px;
  cursor: pointer; color: #7c6750;
  transition: background 0.15s;
}
.create-form-close:hover { background: #ede0cc; }
.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.75rem;
}
.field-group { display: flex; flex-direction: column; gap: 0.3rem; }
.field-group--full { grid-column: 1 / -1; }
.field-label {
  font-size: 11px; font-weight: 600; text-transform: uppercase;
  letter-spacing: 0.07em; color: #7c6448;
}
.field-optional {
  font-size: 9.5px; font-weight: 400; text-transform: lowercase;
  letter-spacing: 0; color: #b8a48a; margin-left: 3px;
}
.field-input {
  box-sizing: border-box;
  padding: 0.5rem 0.7rem;
  font-size: 13px; color: #2c2416;
  background: #fff; border: 1px solid #ddd4c4;
  border-radius: 9px; outline: none; font-family: inherit;
  transition: border-color 0.18s, box-shadow 0.18s;
}
.field-input::placeholder { color: #c0af99; }
.field-input:focus { border-color: #c4a76b; box-shadow: 0 0 0 3px rgba(196,167,107,0.14); }
.save-error {
  font-size: 12px; color: #dc2626; margin: 0;
  background: #fff1f1; border: 1px solid #fecaca;
  border-radius: 8px; padding: 0.4rem 0.7rem;
}
.form-actions { display: flex; gap: 0.5rem; align-items: center; }
.btn-save {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 0.55rem 1.2rem; font-size: 13px; font-weight: 600;
  font-family: inherit; color: #fff;
  background: linear-gradient(135deg, #c4a76b 0%, #a8865a 100%);
  border: none; border-radius: 9px; cursor: pointer;
  transition: opacity 0.18s, transform 0.12s;
  box-shadow: 0 2px 8px rgba(168,134,90,0.28);
}
.btn-save:hover:not(:disabled) { opacity: 0.9; transform: translateY(-1px); }
.btn-save:disabled { opacity: 0.6; cursor: not-allowed; }
.btn-cancel {
  padding: 0.5rem 1rem; font-size: 13px; font-weight: 500;
  font-family: inherit; color: #7c6448;
  background: #f5ede0; border: 1px solid #e5d5bc;
  border-radius: 9px; cursor: pointer; transition: background 0.15s;
}
.btn-cancel:hover { background: #ede0cc; }
.btn-spinner {
  width: 12px; height: 12px;
  border: 2px solid rgba(255,255,255,0.35); border-top-color: #fff;
  border-radius: 50%; animation: spin 0.7s linear infinite;
  display: inline-block;
}
</style>