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
  is_published?: boolean
}

interface Rsvp {
  attendance: 'attending' | 'not_attending'
  note?: string
}

const route  = useRoute()
const router = useRouter()
const code   = route.params.wedding_code as string

// Wedding data
const wedding     = ref<Wedding | null>(null)
const weddingLoad = ref(true)
const weddingErr  = ref('')

// Auth
const isLoggedIn = ref(false)

// RSVP state
const rsvp        = ref<Rsvp | null>(null)
const rsvpLoad    = ref(false)
const submitting  = ref(false)
const submitErr   = ref('')
const submitted   = ref(false)

// Not-attending modal
const showModal = ref(false)
const wishNote  = ref('')

function getToken() { return import.meta.client ? (localStorage.getItem('token') ?? '') : '' }

function formatDate(d: string) {
  return new Date(d).toLocaleDateString('en-GB', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' })
}

function countdown(d: string) {
  const diff = Math.ceil((new Date(d).getTime() - Date.now()) / 86400000)
  if (diff < 0) return `${Math.abs(diff)} days ago`
  if (diff === 0) return 'Today! 🎉'
  return `${diff} days to go`
}

// 1. Fetch wedding (public — no auth)
async function fetchWedding() {
  weddingLoad.value = true
  try {
    const res = await $fetch(`/wedding/${code}`, {
      baseURL: useRuntimeConfig().public.apiBase,
      headers: { Accept: 'application/json' },
    }) as any
    wedding.value = res?.data ?? res
  } catch (e: any) {
    weddingErr.value = e?.data?.message || 'Wedding not found'
  } finally {
    weddingLoad.value = false
  }
}

// 2. Check existing RSVP (only if logged in)
async function fetchRsvp() {
  rsvpLoad.value = true
  try {
    const res = await $fetch(`/visitor/rsvp/${code}`, {
      baseURL: useRuntimeConfig().public.apiBase,
      headers: { Accept: 'application/json', Authorization: `Bearer ${getToken()}` },
    }) as any
    rsvp.value = res?.data ?? null
  } catch {
    rsvp.value = null
  } finally {
    rsvpLoad.value = false
  }
}

// 3. Submit RSVP
async function submitRsvp(attendance: 'attending' | 'not_attending', note = '') {
  submitting.value = true
  submitErr.value  = ''
  try {
    await $fetch(`/visitor/rsvp/${code}`, {
      baseURL: useRuntimeConfig().public.apiBase,
      method:  'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
        Authorization: `Bearer ${getToken()}`,
      },
      body: { attendance, note: note || null },
    })
    rsvp.value    = { attendance, note }
    submitted.value = true
    showModal.value = false
    if (attendance === 'attending') {
      await router.push(`/visitor/dashboard/${code}`)
    }
  } catch (e: any) {
    submitErr.value = e?.data?.message || 'Failed to submit RSVP'
  } finally {
    submitting.value = false
  }
}

function goLogin() {
  router.push(`/?redirect=/wedding/${code}`)
}

onMounted(async () => {
  isLoggedIn.value = !!getToken()
  await fetchWedding()
})
</script>

<template>
  <div>
    <!-- Top nav -->
    <header class="top-nav">
      <div class="nav-brand">
        <div class="nav-logo">W</div>
        <span class="nav-title">WeddingAR</span>
      </div>
      <div class="nav-actions">
        <NuxtLink v-if="!isLoggedIn" to="/" class="btn-nav-login">Log In</NuxtLink>
        <NuxtLink v-if="!isLoggedIn" to="/register" class="btn-nav-register">Register</NuxtLink>
      </div>
    </header>

    <!-- Loading -->
    <div v-if="weddingLoad" class="page-center">
      <div class="big-spinner" />
    </div>

    <!-- Error -->
    <div v-else-if="weddingErr" class="page-center">
      <div class="err-card">
        <svg width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="#dc2626" stroke-width="1.5"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
        <h2>Invitation Not Found</h2>
        <p>{{ weddingErr }}</p>
      </div>
    </div>

    <!-- Wedding Page -->
    <div v-else-if="wedding" class="wedding-page">

      <!-- Hero -->
      <section class="hero-section" :style="wedding.cover_photo ? `--bg: url('${wedding.cover_photo}')` : ''">
        <div class="hero-overlay" />
        <div class="hero-content">
          <p class="hero-eyebrow">You're Invited</p>
          <h1 class="hero-couple">{{ wedding.bride_name }} <span class="hero-amp">&amp;</span> {{ wedding.groom_name }}</h1>
          <div class="hero-meta">
            <div class="hero-meta-item">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="4" width="18" height="18" rx="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></svg>
              {{ formatDate(wedding.date) }}
            </div>
            <div class="hero-meta-item">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/></svg>
              {{ wedding.venue }}
            </div>
            <div class="hero-meta-item hero-meta-item--pill">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/></svg>
              {{ countdown(wedding.date) }}
            </div>
          </div>
        </div>
      </section>

      <!-- RSVP Section -->
      <section class="rsvp-section">

        <!-- Already RSVP'd: attending, redirect to dashboard -->
        <div v-if="rsvp?.attendance === 'attending'" class="rsvp-card rsvp-card--attending">
          <div class="rsvp-icon">🎉</div>
          <h2 class="rsvp-card-title">You're attending!</h2>
          <p class="rsvp-card-sub">We can't wait to celebrate with you.</p>
          <NuxtLink :to="`/visitor/dashboard/${code}`" class="btn-primary">
            View Event Details
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M5 12h14M12 5l7 7-7 7"/></svg>
          </NuxtLink>
        </div>

        <!-- Already RSVP'd: not attending -->
        <div v-else-if="rsvp?.attendance === 'not_attending' || (submitted && rsvp?.attendance === 'not_attending')" class="rsvp-card rsvp-card--notattending">
          <div class="rsvp-icon">💌</div>
          <h2 class="rsvp-card-title">Thank you, we'll miss you!</h2>
          <p class="rsvp-card-sub" v-if="rsvp?.note">Your wish: "{{ rsvp.note }}"</p>
          <p class="rsvp-card-sub" v-else>Your RSVP has been recorded.</p>
        </div>

        <!-- Not logged in -->
        <div v-else-if="!isLoggedIn" class="rsvp-card">
          <div class="rsvp-icon">💍</div>
          <h2 class="rsvp-card-title">Will you be attending?</h2>
          <p class="rsvp-card-sub">Log in or create an account to RSVP to this wedding.</p>
          <div class="rsvp-btns">
            <button class="btn-primary" @click="goLogin">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M15 3h4a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2h-4"/><polyline points="10 17 15 12 10 7"/><line x1="15" y1="12" x2="3" y2="12"/></svg>
              Log In to RSVP
            </button>
            <NuxtLink :to="`/register?redirect=/wedding/${code}`" class="btn-secondary">Register</NuxtLink>
          </div>
        </div>

        <!-- Logged in — RSVP prompt -->
        <div v-else-if="!rsvpLoad && !rsvp" class="rsvp-card">
          <div class="rsvp-icon">💍</div>
          <h2 class="rsvp-card-title">Will you be attending?</h2>
          <p class="rsvp-card-sub">Please let {{ wedding.bride_name }} &amp; {{ wedding.groom_name }} know!</p>
          <p v-if="submitErr" class="rsvp-error">{{ submitErr }}</p>
          <div class="rsvp-btns">
            <button class="btn-attending" :disabled="submitting" @click="submitRsvp('attending')">
              <span v-if="submitting" class="spinner spinner--white" />
              <span v-else>✅</span>
              Attending
            </button>
            <button class="btn-notattending" :disabled="submitting" @click="showModal = true">
              ❌ Not Attending
            </button>
          </div>
        </div>

        <!-- RSVP loading -->
        <div v-else-if="rsvpLoad" class="rsvp-card">
          <div class="big-spinner" style="margin: 1rem auto;" />
        </div>

      </section>
    </div>

    <!-- Not Attending Modal -->
    <Transition name="modal-fade">
      <div v-if="showModal" class="modal-backdrop" @click.self="showModal = false">
        <div class="modal-box">
          <div class="modal-icon">💌</div>
          <h3 class="modal-title">Leave a Wish</h3>
          <p class="modal-sub">We'll miss you! Leave a heartfelt message for the couple.</p>
          <textarea
            v-model="wishNote"
            class="wish-textarea"
            placeholder="Wishing you both a lifetime of love and happiness… (optional)"
            rows="4"
          />
          <p v-if="submitErr" class="rsvp-error">{{ submitErr }}</p>
          <div class="modal-actions">
            <button class="btn-primary" :disabled="submitting" @click="submitRsvp('not_attending', wishNote)">
              <span v-if="submitting" class="spinner spinner--white" />
              {{ submitting ? 'Sending…' : 'Send Wish & Confirm' }}
            </button>
            <button class="btn-ghost" @click="showModal = false">Cancel</button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
/* Nav */
.top-nav {
  position: sticky; top: 0; z-index: 50;
  display: flex; align-items: center; justify-content: space-between;
  padding: 0.9rem 2rem;
  background: rgba(250,248,244,0.92);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid #ece3d8;
}
.nav-brand { display: flex; align-items: center; gap: 10px; }
.nav-logo {
  width: 34px; height: 34px; border-radius: 9px;
  background: #c4a96b; color: #fff; font-weight: 800; font-size: 14px;
  display: grid; place-items: center;
}
.nav-title { font-weight: 700; font-size: 0.95rem; color: #2c2416; }
.nav-actions { display: flex; gap: 0.6rem; }
.btn-nav-login {
  padding: 0.4rem 0.9rem; font-size: 13px; font-weight: 500;
  color: #7c5c36; background: #fff8ee; border: 1px solid #e8d5b5;
  border-radius: 8px; text-decoration: none;
  transition: background 0.15s;
}
.btn-nav-login:hover { background: #fff0d6; }
.btn-nav-register {
  padding: 0.4rem 0.9rem; font-size: 13px; font-weight: 600;
  color: #fff; background: linear-gradient(135deg, #c4a76b, #a8865a);
  border: none; border-radius: 8px; text-decoration: none;
  transition: opacity 0.15s;
}
.btn-nav-register:hover { opacity: 0.88; }

/* Loading / Error */
.page-center {
  min-height: 60vh; display: flex; align-items: center; justify-content: center;
}
.big-spinner {
  width: 40px; height: 40px;
  border: 3px solid #e8d5b5; border-top-color: #c4a76b;
  border-radius: 50%; animation: spin 0.8s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
.err-card {
  text-align: center; display: flex; flex-direction: column;
  align-items: center; gap: 0.75rem; max-width: 340px; padding: 2rem;
}
.err-card h2 { margin: 0; font-size: 1.3rem; color: #2c2416; }
.err-card p  { margin: 0; font-size: 13.5px; color: #7c6448; }

/* Hero */
.wedding-page { display: flex; flex-direction: column; }
.hero-section {
  position: relative;
  min-height: 65vh;
  background-image: var(--bg, url('https://images.unsplash.com/photo-1519225421980-715cb0215aed?w=1200&q=80'));
  background-size: cover;
  background-position: center;
  display: flex; align-items: flex-end;
}
.hero-overlay {
  position: absolute; inset: 0;
  background: linear-gradient(to top, rgba(20,14,6,0.82) 0%, rgba(20,14,6,0.3) 50%, transparent 100%);
}
.hero-content {
  position: relative;
  padding: 3rem 2.5rem;
  max-width: 800px;
  margin: 0 auto;
  width: 100%;
}
.hero-eyebrow {
  margin: 0 0 0.4rem;
  font-size: 12px; text-transform: uppercase; letter-spacing: 0.18em;
  color: #e8c87a; font-weight: 600;
}
.hero-couple {
  margin: 0 0 1.2rem;
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: clamp(2.5rem, 6vw, 4.5rem);
  font-weight: 400; font-style: italic;
  color: #fff; line-height: 1.05;
}
.hero-amp { color: #e8c87a; font-style: normal; margin: 0 0.15em; }
.hero-meta { display: flex; flex-wrap: wrap; gap: 1rem; align-items: center; }
.hero-meta-item {
  display: flex; align-items: center; gap: 6px;
  font-size: 13.5px; color: rgba(255,255,255,0.88);
}
.hero-meta-item--pill {
  background: rgba(196,167,107,0.25);
  border: 1px solid rgba(196,167,107,0.5);
  border-radius: 999px; padding: 0.25rem 0.75rem;
  color: #e8c87a; font-weight: 600;
}

/* RSVP section */
.rsvp-section {
  background: #faf8f4;
  display: flex; justify-content: center;
  padding: 3rem 1.5rem;
}
.rsvp-card {
  background: #fff;
  border: 1px solid #ece3d8;
  border-radius: 20px;
  padding: 2.5rem 2rem;
  max-width: 480px; width: 100%;
  text-align: center;
  box-shadow: 0 8px 40px rgba(0,0,0,0.07);
  display: flex; flex-direction: column; align-items: center; gap: 0.75rem;
}
.rsvp-card--attending { border-color: #bbf7d0; background: #f0fdf4; }
.rsvp-card--notattending { border-color: #fce7f3; background: #fdf4ff; }
.rsvp-icon { font-size: 2.5rem; line-height: 1; }
.rsvp-card-title {
  margin: 0;
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 1.7rem; font-style: italic; font-weight: 400; color: #2c2416;
}
.rsvp-card-sub { margin: 0; font-size: 13.5px; color: #7c6448; line-height: 1.5; max-width: 320px; }
.rsvp-error { margin: 0; font-size: 12.5px; color: #dc2626; }
.rsvp-btns { display: flex; gap: 0.75rem; flex-wrap: wrap; justify-content: center; margin-top: 0.5rem; }

/* Buttons */
.btn-primary {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 0.65rem 1.4rem; font-size: 14px; font-weight: 600;
  font-family: inherit; color: #fff;
  background: linear-gradient(135deg, #c4a76b 0%, #a8865a 100%);
  border: none; border-radius: 12px; cursor: pointer; text-decoration: none;
  transition: opacity 0.18s, transform 0.12s;
  box-shadow: 0 2px 12px rgba(168,134,90,0.33);
}
.btn-primary:hover { opacity: 0.9; transform: translateY(-1px); }
.btn-primary:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }

.btn-secondary {
  padding: 0.6rem 1.2rem; font-size: 13.5px; font-weight: 500;
  font-family: inherit; color: #7c6448;
  background: #f5ede0; border: 1px solid #e5d5bc;
  border-radius: 12px; cursor: pointer; text-decoration: none;
  transition: background 0.15s;
  display: inline-flex; align-items: center;
}
.btn-secondary:hover { background: #ede0cc; }

.btn-attending {
  display: inline-flex; align-items: center; gap: 7px;
  padding: 0.7rem 1.6rem; font-size: 14px; font-weight: 600;
  font-family: inherit; color: #fff;
  background: linear-gradient(135deg, #22c55e, #16a34a);
  border: none; border-radius: 12px; cursor: pointer;
  box-shadow: 0 2px 12px rgba(34,197,94,0.28);
  transition: opacity 0.18s, transform 0.12s;
}
.btn-attending:hover { opacity: 0.9; transform: translateY(-1px); }
.btn-attending:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }

.btn-notattending {
  display: inline-flex; align-items: center; gap: 7px;
  padding: 0.7rem 1.4rem; font-size: 14px; font-weight: 500;
  font-family: inherit; color: #dc2626;
  background: #fff1f1; border: 1.5px solid #fca5a5;
  border-radius: 12px; cursor: pointer;
  transition: background 0.15s, transform 0.12s;
}
.btn-notattending:hover { background: #ffe4e6; transform: translateY(-1px); }
.btn-notattending:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }

.btn-ghost {
  background: none; border: none; cursor: pointer;
  font-size: 13px; color: #a8967c; padding: 0.5rem;
  text-decoration: underline; text-underline-offset: 2px;
}
.btn-ghost:hover { color: #7c6448; }

/* Modal */
.modal-backdrop {
  position: fixed; inset: 0; z-index: 100;
  background: rgba(20,14,6,0.55);
  backdrop-filter: blur(4px);
  display: flex; align-items: center; justify-content: center;
  padding: 1rem;
}
.modal-box {
  background: #fffdf9;
  border: 1px solid #ece3d8;
  border-radius: 20px;
  padding: 2.5rem 2rem;
  max-width: 420px; width: 100%;
  display: flex; flex-direction: column; align-items: center; gap: 0.85rem;
  box-shadow: 0 20px 60px rgba(0,0,0,0.18);
}
.modal-icon { font-size: 2.5rem; }
.modal-title {
  margin: 0;
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 1.6rem; font-style: italic; font-weight: 400; color: #2c2416;
}
.modal-sub { margin: 0; font-size: 13.5px; color: #7c6448; text-align: center; line-height: 1.55; }
.wish-textarea {
  width: 100%; box-sizing: border-box;
  padding: 0.7rem 0.85rem; font-size: 13.5px; color: #2c2416;
  background: #fff; border: 1px solid #ddd4c4; border-radius: 12px;
  outline: none; font-family: inherit; resize: vertical; line-height: 1.55;
  transition: border-color 0.18s, box-shadow 0.18s;
}
.wish-textarea::placeholder { color: #c0af99; }
.wish-textarea:focus { border-color: #c4a76b; box-shadow: 0 0 0 3px rgba(196,167,107,0.14); }
.modal-actions { display: flex; flex-direction: column; align-items: center; gap: 0.5rem; width: 100%; }
.modal-actions .btn-primary { width: 100%; justify-content: center; }

/* Spinner */
.spinner {
  width: 13px; height: 13px;
  border: 2px solid rgba(255,255,255,0.35); border-top-color: #fff;
  border-radius: 50%; animation: spin 0.7s linear infinite; display: inline-block;
}

/* Transitions */
.modal-fade-enter-active, .modal-fade-leave-active { transition: opacity 0.22s ease; }
.modal-fade-enter-active .modal-box, .modal-fade-leave-active .modal-box { transition: transform 0.22s cubic-bezier(0.34,1.56,0.64,1); }
.modal-fade-enter-from, .modal-fade-leave-to { opacity: 0; }
.modal-fade-enter-from .modal-box { transform: scale(0.92) translateY(10px); }
</style>
