<script setup lang="ts">
definePageMeta({ layout: 'dashboard-layout' })

interface BankQr {
  id: number
  bank_name: string
  account_name: string
  account_number: string
  qr_image?: string
  is_published?: boolean
}

const bankQr    = ref<BankQr | null>(null)
const loading   = ref(true)
const fetchErr  = ref('')
const isEditing = ref(false)

const form = reactive({
  bank_name:      '',
  account_name:   '',
  account_number: '',
  qr_image:       '',
})

const saving         = ref(false)
const saveError      = ref('')
const saveSuccess    = ref('')
const publishing     = ref(false)
const publishError   = ref('')
const publishSuccess = ref('')

function getToken() { return localStorage.getItem('token') ?? '' }

function fillForm(q: BankQr) {
  form.bank_name      = q.bank_name
  form.account_name   = q.account_name
  form.account_number = q.account_number
  form.qr_image       = q.qr_image ?? ''
}

async function fetchQr() {
  loading.value = true
  fetchErr.value = ''
  try {
    const res = await $fetch('/bankqr/show', {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'GET',
      headers: { Accept: 'application/json', Authorization: `Bearer ${getToken()}` },
    }) as any
    bankQr.value = res?.data ?? null
    if (bankQr.value) fillForm(bankQr.value)
  } catch (err: any) {
    const status = err?.status ?? err?.response?.status
    if (status === 404) bankQr.value = null
    else fetchErr.value = err?.data?.message || err?.message || 'Failed to load'
  } finally {
    loading.value = false
  }
}

async function handleSave() {
  saving.value = true
  saveError.value = ''
  saveSuccess.value = ''
  try {
    const endpoint = bankQr.value ? '/bankqr/update' : '/bankqr/store'
    const res = await $fetch(endpoint, {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
        Authorization: `Bearer ${getToken()}`,
      },
      body: {
        bank_name:      form.bank_name,
        account_name:   form.account_name,
        account_number: form.account_number,
        qr_image:       form.qr_image || undefined,
      },
    }) as any
    saveSuccess.value = res?.message ?? 'Saved successfully'
    await fetchQr()
    isEditing.value = false
  } catch (err: any) {
    saveError.value = err?.data?.message || err?.message || 'Failed to save'
  } finally {
    saving.value = false
  }
}

async function handlePublish() {
  publishing.value = true
  publishError.value = ''
  publishSuccess.value = ''
  try {
    const res = await $fetch('/bankqr/publish', {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'POST',
      headers: { Accept: 'application/json', Authorization: `Bearer ${getToken()}` },
    }) as any
    publishSuccess.value = res?.message ?? 'Published!'
    if (bankQr.value) bankQr.value.is_published = true
  } catch (err: any) {
    publishError.value = err?.data?.message || err?.message || 'Failed to publish'
  } finally {
    publishing.value = false
  }
}

onMounted(fetchQr)
</script>

<template>
  <div class="bqr-page">

    <!-- Page header -->
    <div class="page-header">
      <div>
        <p class="page-eyebrow">Payment</p>
        <h1 class="page-title">Bank QR</h1>
        <p class="page-sub">Share your bank details so guests can send gifts digitally.</p>
      </div>
      <button
        v-if="bankQr && !isEditing"
        class="btn-edit"
        @click="isEditing = true"
      >
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 20h9M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/></svg>
        Edit Details
      </button>
    </div>

    <!-- Loading -->
    <div v-if="loading" class="state-card">
      <div class="sk-row">
        <div class="sk-box sk-box--sq" />
        <div class="sk-lines">
          <div class="sk-line sk-line--md" />
          <div class="sk-line sk-line--sm" />
          <div class="sk-line sk-line--lg" />
        </div>
      </div>
    </div>

    <!-- Fetch error -->
    <div v-else-if="fetchErr" class="state-card state-card--error">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="#dc2626" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
      <p>{{ fetchErr }}</p>
    </div>

    <!-- Empty state -->
    <div v-else-if="!bankQr && !isEditing" class="state-card state-card--empty">
      <div class="empty-icon">
        <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="#c4a96b" stroke-width="1.5"><rect x="2" y="5" width="20" height="14" rx="2"/><path d="M2 10h20"/></svg>
      </div>
      <h2 class="empty-title">No Bank QR Yet</h2>
      <p class="empty-sub">Add your bank details so guests can easily send gifts via QR or transfer.</p>
      <button class="btn-primary" @click="isEditing = true">
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
        Add Bank Details
      </button>
    </div>

    <!-- Form (create or edit) -->
    <div v-else-if="isEditing" class="form-card">
      <div class="form-card-header">
        <h2 class="form-card-title">{{ bankQr ? 'Edit' : 'Add' }} Bank Details</h2>
        <button v-if="bankQr" class="btn-ghost" @click="isEditing = false; fillForm(bankQr)">Cancel</button>
      </div>

      <form class="form-grid" @submit.prevent="handleSave">
        <div class="field-group">
          <label class="field-label" for="bank-name">Bank Name</label>
          <input id="bank-name" v-model="form.bank_name" class="field-input" placeholder="e.g. Maybank" required />
        </div>

        <div class="field-group">
          <label class="field-label" for="account-number">Account Number</label>
          <input id="account-number" v-model="form.account_number" class="field-input" placeholder="e.g. 1234 5678 9012" required />
        </div>

        <div class="field-group field-group--full">
          <label class="field-label" for="account-name">Account Name</label>
          <input id="account-name" v-model="form.account_name" class="field-input" placeholder="e.g. EMMA BINTI ALI" required />
        </div>

        <div class="field-group field-group--full">
          <label class="field-label" for="qr-image">
            QR Image URL
            <span class="field-optional">optional</span>
          </label>
          <input id="qr-image" v-model="form.qr_image" class="field-input" placeholder="https://your-qr-image.png" />
        </div>

        <!-- QR preview -->
        <div v-if="form.qr_image" class="qr-preview-wrap field-group--full">
          <p class="field-label">Preview</p>
          <img :src="form.qr_image" alt="QR preview" class="qr-preview-img" />
        </div>

        <p v-if="saveError" class="feedback-error field-group--full">{{ saveError }}</p>
        <p v-if="saveSuccess" class="feedback-success field-group--full">{{ saveSuccess }}</p>

        <div class="form-actions field-group--full">
          <button type="submit" class="btn-primary" :disabled="saving">
            <span v-if="saving" class="spinner" />
            {{ saving ? 'Saving…' : 'Save Details' }}
          </button>
          <button v-if="bankQr" type="button" class="btn-secondary" @click="isEditing = false; fillForm(bankQr)">Cancel</button>
        </div>
      </form>
    </div>

    <!-- Display card -->
    <div v-else class="display-card">
      <div class="display-left">
        <p class="display-eyebrow">Bank Details</p>

        <div class="detail-row">
          <span class="detail-label">Bank</span>
          <span class="detail-value">{{ bankQr!.bank_name }}</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Account Name</span>
          <span class="detail-value">{{ bankQr!.account_name }}</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Account Number</span>
          <span class="detail-value detail-value--mono">{{ bankQr!.account_number }}</span>
        </div>

        <!-- Publish -->
        <div class="publish-row">
          <span v-if="bankQr!.is_published" class="badge-published">
            <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6L9 17l-5-5"/></svg>
            Published
          </span>
          <template v-else>
            <button class="btn-publish" :disabled="publishing" @click="handlePublish">
              <span v-if="publishing" class="spinner spinner--white" />
              <svg v-else width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 2L11 13"/><path d="M22 2L15 22l-4-9-9-4 20-7z"/></svg>
              {{ publishing ? 'Publishing…' : 'Publish QR' }}
            </button>
          </template>
          <p v-if="publishError" class="feedback-error">{{ publishError }}</p>
          <p v-if="publishSuccess" class="feedback-success">{{ publishSuccess }}</p>
        </div>
      </div>

      <!-- QR Image -->
      <div class="display-right">
        <div v-if="bankQr!.qr_image" class="qr-frame">
          <img :src="bankQr!.qr_image" alt="Bank QR Code" class="qr-img" />
        </div>
        <div v-else class="qr-placeholder">
          <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="#c4a96b" stroke-width="1.2">
            <rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/>
            <rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="3" height="3"/>
            <rect x="19" y="14" width="2" height="2"/><rect x="14" y="19" width="2" height="2"/>
            <rect x="18" y="18" width="3" height="3"/>
          </svg>
          <p class="qr-placeholder-text">No QR image added</p>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
.bqr-page {
  max-width: 720px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  padding: 0.5rem 0 2rem;
}

/* Header */
.page-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
}
.page-eyebrow {
  margin: 0;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: #b89b6a;
  font-weight: 600;
}
.page-title {
  margin: 0.2rem 0 0.3rem;
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 2rem;
  font-weight: 400;
  font-style: italic;
  color: #2c2416;
  line-height: 1.1;
}
.page-sub { margin: 0; font-size: 13px; color: #7c6448; }

/* State cards */
.state-card {
  background: #fffdf8;
  border: 1px solid #ece3d8;
  border-radius: 16px;
  padding: 3rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  text-align: center;
}
.state-card--error { border-color: #fecaca; background: #fff8f8; color: #dc2626; }
.state-card--error p { margin: 0; font-size: 13.5px; }
.state-card--empty {}

/* Skeleton */
.sk-row { display: flex; gap: 1.5rem; width: 100%; max-width: 420px; }
.sk-box { border-radius: 12px; background: linear-gradient(90deg, #f0ebe0 25%, #e8e0d0 50%, #f0ebe0 75%); background-size: 200% 100%; animation: shimmer 1.4s ease infinite; }
.sk-box--sq { width: 120px; height: 120px; flex-shrink: 0; }
.sk-lines { flex: 1; display: flex; flex-direction: column; gap: 0.7rem; justify-content: center; }
.sk-line { height: 12px; border-radius: 6px; background: linear-gradient(90deg, #f0ebe0 25%, #e8e0d0 50%, #f0ebe0 75%); background-size: 200% 100%; animation: shimmer 1.4s ease infinite; }
.sk-line--sm { width: 40%; }
.sk-line--md { width: 60%; }
.sk-line--lg { width: 80%; }
@keyframes shimmer { to { background-position: -200% 0; } }

/* Empty */
.empty-icon {
  width: 68px; height: 68px;
  background: #fff8ee; border: 1px solid #e8d5b5; border-radius: 20px;
  display: flex; align-items: center; justify-content: center;
}
.empty-title {
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 1.4rem; font-style: italic; font-weight: 400;
  color: #2c2416; margin: 0;
}
.empty-sub { font-size: 13px; color: #7c6448; margin: 0; line-height: 1.55; max-width: 340px; }

/* Form card */
.form-card {
  background: #fffdf8;
  border: 1px solid #ece3d8;
  border-radius: 16px;
  padding: 1.75rem;
  box-shadow: 0 4px 20px rgba(0,0,0,0.04);
}
.form-card-header {
  display: flex; align-items: center; justify-content: space-between;
  margin-bottom: 1.25rem;
}
.form-card-title { margin: 0; font-size: 1.05rem; font-weight: 600; color: #2c2416; }

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}
.field-group { display: flex; flex-direction: column; gap: 0.3rem; }
.field-group--full { grid-column: 1 / -1; }
.field-label {
  font-size: 11px; font-weight: 600; text-transform: uppercase;
  letter-spacing: 0.07em; color: #7c6448;
  display: flex; align-items: center; gap: 5px;
}
.field-optional { font-size: 9.5px; font-weight: 400; text-transform: lowercase; letter-spacing: 0; color: #b8a48a; }
.field-input {
  box-sizing: border-box;
  padding: 0.55rem 0.75rem;
  font-size: 13.5px; color: #2c2416;
  background: #fff; border: 1px solid #ddd4c4;
  border-radius: 10px; outline: none; font-family: inherit;
  transition: border-color 0.18s, box-shadow 0.18s;
}
.field-input::placeholder { color: #c0af99; }
.field-input:focus { border-color: #c4a76b; box-shadow: 0 0 0 3px rgba(196,167,107,0.14); }

/* QR preview */
.qr-preview-wrap { display: flex; flex-direction: column; gap: 0.4rem; }
.qr-preview-img { width: 140px; height: 140px; object-fit: contain; border: 1px solid #ece3d8; border-radius: 12px; background: #fff; }

/* Feedback */
.feedback-error { margin: 0; font-size: 12px; color: #dc2626; background: #fff1f1; border: 1px solid #fecaca; border-radius: 8px; padding: 0.4rem 0.7rem; }
.feedback-success { margin: 0; font-size: 12px; color: #16a34a; background: #f0fdf4; border: 1px solid #bbf7d0; border-radius: 8px; padding: 0.4rem 0.7rem; }

/* Form actions */
.form-actions { display: flex; gap: 0.6rem; align-items: center; margin-top: 0.25rem; }

/* Display card */
.display-card {
  background: #fffdf8;
  border: 1px solid #ece3d8;
  border-radius: 16px;
  padding: 1.75rem;
  display: flex;
  gap: 2rem;
  align-items: flex-start;
  box-shadow: 0 4px 20px rgba(0,0,0,0.04);
}
.display-left { flex: 1; display: flex; flex-direction: column; gap: 0.85rem; }
.display-eyebrow {
  margin: 0; font-size: 10px; text-transform: uppercase;
  letter-spacing: 0.09em; color: #b89b6a; font-weight: 600;
}
.detail-row { display: flex; flex-direction: column; gap: 2px; }
.detail-label { font-size: 11px; text-transform: uppercase; letter-spacing: 0.07em; color: #a8967c; font-weight: 600; }
.detail-value { font-size: 15px; font-weight: 500; color: #2c2416; font-family: 'Cormorant Garamond', Georgia, serif; }
.detail-value--mono { font-family: 'DM Mono', 'Courier New', monospace; font-size: 14px; letter-spacing: 0.05em; }

/* Publish */
.publish-row { display: flex; flex-direction: column; align-items: flex-start; gap: 6px; margin-top: 0.5rem; }
.badge-published {
  display: inline-flex; align-items: center; gap: 5px;
  font-size: 11.5px; font-weight: 600; color: #16a34a;
  background: #f0fdf4; border: 1px solid #bbf7d0;
  border-radius: 999px; padding: 0.25rem 0.65rem;
}

/* QR frame */
.display-right { flex-shrink: 0; }
.qr-frame {
  width: 160px; height: 160px;
  background: #fff; border: 1px solid #ece3d8;
  border-radius: 14px; overflow: hidden;
  display: flex; align-items: center; justify-content: center;
}
.qr-img { width: 100%; height: 100%; object-fit: contain; }
.qr-placeholder {
  width: 160px; height: 160px;
  background: #fffbf0; border: 1px dashed #ddc98a;
  border-radius: 14px;
  display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 8px;
}
.qr-placeholder-text { margin: 0; font-size: 11px; color: #b89b6a; text-align: center; }

/* Buttons */
.btn-primary {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 0.55rem 1.2rem; font-size: 13px; font-weight: 600;
  font-family: inherit; color: #fff;
  background: linear-gradient(135deg, #c4a76b 0%, #a8865a 100%);
  border: none; border-radius: 10px; cursor: pointer;
  transition: opacity 0.18s, transform 0.12s, box-shadow 0.18s;
  box-shadow: 0 2px 10px rgba(168,134,90,0.28);
}
.btn-primary:hover:not(:disabled) { opacity: 0.9; transform: translateY(-1px); }
.btn-primary:disabled { opacity: 0.6; cursor: not-allowed; }

.btn-secondary {
  padding: 0.5rem 1rem; font-size: 13px; font-weight: 500;
  font-family: inherit; color: #7c6448;
  background: #f5ede0; border: 1px solid #e5d5bc;
  border-radius: 10px; cursor: pointer; transition: background 0.15s;
}
.btn-secondary:hover { background: #ede0cc; }

.btn-publish {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 0.5rem 1.1rem; font-size: 13px; font-weight: 600;
  font-family: inherit; color: #fff;
  background: linear-gradient(135deg, #c4a76b 0%, #a8865a 100%);
  border: none; border-radius: 10px; cursor: pointer;
  transition: opacity 0.18s, transform 0.12s;
  box-shadow: 0 2px 8px rgba(168,134,90,0.28);
}
.btn-publish:hover:not(:disabled) { opacity: 0.9; transform: translateY(-1px); }
.btn-publish:disabled { opacity: 0.6; cursor: not-allowed; }

.btn-edit {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 0.4rem 0.9rem; font-size: 12px; font-weight: 600;
  font-family: inherit; color: #7c5c36;
  background: #fff8ee; border: 1px solid #e8d5b5;
  border-radius: 8px; cursor: pointer;
  transition: background 0.15s, transform 0.12s;
}
.btn-edit:hover { background: #fff0d6; transform: translateY(-1px); }

.btn-ghost {
  font-size: 12.5px; color: #7c6448; background: none;
  border: none; cursor: pointer; padding: 0;
  text-decoration: underline; text-underline-offset: 2px; opacity: 0.7;
}
.btn-ghost:hover { opacity: 1; }

/* Spinner */
.spinner {
  width: 12px; height: 12px;
  border: 2px solid rgba(255,255,255,0.35); border-top-color: #fff;
  border-radius: 50%; animation: spin 0.7s linear infinite; display: inline-block;
}
@keyframes spin { to { transform: rotate(360deg); } }
</style>
