<script setup lang="ts">
interface TimelineItem {
  id: number | string
  title: string
  datetime: string
  location: string
  status: 'done' | 'upcoming' | 'pending'
  note?: string
}

const items = ref<TimelineItem[]>([])
const loading = ref(true)
const error = ref('')
const isOpen = ref(false)
const form = reactive({ time: '', title: '', note: '' })
const submitting = ref(false)
const submitError = ref('')
const submitSuccess = ref('')

function normalizeStatus(s: string): TimelineItem['status'] {
  const v = (s || '').toLowerCase()
  if (v === 'done' || v === 'completed') return 'done'
  if (v === 'upcoming' || v === 'scheduled') return 'upcoming'
  return 'pending'
}

onMounted(async () => {
  loading.value = true
  error.value = ''
  try {
    const token = localStorage.getItem('token')
    if (!token) {
      error.value = 'Not authenticated'
      return
    }
    const res = await $fetch('/tentative/index', {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'GET',
      headers: {
        Accept: 'application/json',
        Authorization: `Bearer ${token}`,
      },
    }) as any

    if (Array.isArray(res?.data)) {
      items.value = res.data.map((t: any) => ({
        id: t.id,
        title: t.title ?? t.name ?? 'Untitled',
        datetime: t.datetime ?? t.time ?? '',
        location: t.location ?? t.place ?? '',
        status: normalizeStatus(t.status ?? 'pending'),
        note: t.note ?? t.description ?? undefined,
      }))
    } else if (res?.message) {
      items.value = []
    } else {
      items.value = []
    }
  } catch (err: any) {
    error.value = err?.data?.error || err?.message || 'Failed to load timeline'
  } finally {
    loading.value = false
  }
})

async function handleSubmit() {
  submitting.value = true
  submitError.value = ''
  submitSuccess.value = ''
  try {
    const token = localStorage.getItem('token')
    if (!token) {
      submitError.value = 'Not authenticated'
      return
    }
    const res = await $fetch('/tentative/store', {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`,
      },
      body: {
        time: form.time,
        title: form.title,
        note: form.note,
      },
    }) as any

    const created = res?.data
    if (created) {
      items.value.push({
        id: created.id,
        title: created.title ?? 'Untitled',
        datetime: created.time ?? '',
        location: created.location ?? '',
        status: normalizeStatus(created.status ?? 'pending'),
        note: created.note ?? undefined,
      })
      submitSuccess.value = 'Tentative created'
      isOpen.value = false
      form.time = ''
      form.title = ''
      form.note = ''
    } else {
      submitError.value = 'Unexpected response'
    }
  } catch (err: any) {
    submitError.value = err?.data?.error || err?.message || 'Failed to create tentative'
  } finally {
    submitting.value = false
  }
}

const deletingId = ref<number | string | null>(null)
const publishing = ref(false)
const publishSuccess = ref('')
const publishError = ref('')

async function handlePublish() {
  publishing.value = true
  publishSuccess.value = ''
  publishError.value = ''
  try {
    const token = localStorage.getItem('token')
    if (!token) { publishError.value = 'Not authenticated'; return }
    await $fetch('/tentative/publish', {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'POST',
      headers: {
        Accept: 'application/json',
        Authorization: `Bearer ${token}`,
      },
    })
    publishSuccess.value = 'Timeline published successfully!'
  } catch (err: any) {
    publishError.value = err?.data?.error || err?.message || 'Failed to publish'
  } finally {
    publishing.value = false
  }
}

async function handleDelete(id: number | string) {
  if (deletingId.value) return
  deletingId.value = id
  try {
    const token = localStorage.getItem('token')
    if (!token) return
    await $fetch(`/tentative/destroy/${id}`, {
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`,
      },
      body: { tentative_id: id },
    })
    window.location.reload()
  } catch (err: any) {
    console.error('Delete failed', err)
  } finally {
    deletingId.value = null
  }
}
</script>

<template>
  <section class="tentative-wrap">
    <div class="tentative-header">
      <div>
        <p class="small-label">Wedding Plan</p>
        <h2>Wedding Day Timeline</h2>
        <p class="subtitle">Track milestone tasks and vendors from early setup through the reception.</p>
      </div>
      <div class="header-actions">
        <span class="status-pill">Tentative Schedule</span>
        <button class="add-btn" @click="isOpen = true">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
          Add Tentative
        </button>
      </div>
    </div>

    <div v-if="loading" class="subtitle">Loading timeline...</div>
    <div v-else-if="error" class="subtitle">{{ error }}</div>

    <ul v-else class="timeline-list">
      <li v-for="(item, index) in items" :key="item.title + index" class="timeline-item">
        <div class="timeline-meta">
          <div class="timeline-dot" :class="item.status"></div>
          <div class="timeline-line" v-if="index < items.length - 1"></div>
        </div>
        <div class="timeline-card">
          <div class="timeline-top">
            <p class="timeline-time">{{ item.datetime }}</p>
            <div class="timeline-top-right">
              <p class="timeline-status" :class="item.status">{{ item.status === 'done' ? 'Done' : item.status === 'upcoming' ? 'Upcoming' : 'Pending' }}</p>
              <button
                class="delete-btn"
                :class="{ 'delete-btn--loading': deletingId === item.id }"
                :disabled="deletingId === item.id"
                @click="handleDelete(item.id)"
                :aria-label="'Delete ' + item.title"
              >
                <span v-if="deletingId === item.id" class="delete-spinner" />
                <svg v-else width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                  <polyline points="3 6 5 6 21 6"/>
                  <path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/>
                  <path d="M10 11v6M14 11v6"/>
                  <path d="M9 6V4a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v2"/>
                </svg>
              </button>
            </div>
          </div>
          <h3 class="timeline-title">{{ item.title }}</h3>
          <p class="timeline-location">{{ item.location }}</p>
          <p v-if="item.note" class="timeline-note">{{ item.note }}</p>
        </div>
      </li>
    </ul>

    <!-- Publish strip -->
    <div v-if="!loading && !error && items.length" class="publish-strip">
      <div class="publish-strip-info">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 2L11 13"/><path d="M22 2L15 22l-4-9-9-4 20-7z"/></svg>
        <span>Ready to share your timeline with guests?</span>
      </div>
      <div class="publish-feedback">
        <p v-if="publishError" class="pub-error">{{ publishError }}</p>
        <p v-if="publishSuccess" class="pub-success">{{ publishSuccess }}</p>
      </div>
      <button class="btn-publish" :disabled="publishing" @click="handlePublish">
        <span v-if="publishing" class="publish-spinner" />
        <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 2L11 13"/><path d="M22 2L15 22l-4-9-9-4 20-7z"/></svg>
        {{ publishing ? 'Publishing…' : 'Publish Timeline' }}
      </button>
    </div>

    <!-- Slide-in inner panel -->
    <Transition name="backdrop-fade">
      <div v-if="isOpen" class="panel-backdrop" @click.self="isOpen = false" />
    </Transition>

    <Transition name="panel-slide">
      <div v-if="isOpen" class="add-panel" role="dialog" aria-modal="true" aria-label="Add Tentative">
        <div class="panel-header">
          <div class="panel-title-group">
            <span class="panel-eyebrow">New Entry</span>
            <h3 class="panel-title">Add Tentative</h3>
          </div>
          <button class="panel-close" @click="isOpen = false" aria-label="Close">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
          </button>
        </div>

        <form class="panel-form" @submit.prevent="handleSubmit">
          <div class="field-group">
            <label class="field-label" for="tentative-time">
              <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/></svg>
              Time
            </label>
            <input
              id="tentative-time"
              v-model="form.time"
              class="field-input"
              type="text"
              placeholder="e.g. 3:00 PM"
              autocomplete="off"
            />
          </div>

          <div class="field-group">
            <label class="field-label" for="tentative-title">
              <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 20h9M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/></svg>
              Title
            </label>
            <input
              id="tentative-title"
              v-model="form.title"
              class="field-input"
              type="text"
              placeholder="e.g. Ceremony start"
              autocomplete="off"
            />
          </div>

          <div class="field-group">
            <label class="field-label" for="tentative-note">
              <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
              Note
              <span class="field-optional">optional</span>
            </label>
            <textarea
              id="tentative-note"
              v-model="form.note"
              class="field-input field-textarea"
              placeholder="Add any details or reminders..."
              rows="3"
            />
          </div>

          <p v-if="submitError" class="panel-error">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
            {{ submitError }}
          </p>
          <p v-if="submitSuccess" class="panel-success">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M20 6L9 17l-5-5"/></svg>
            {{ submitSuccess }}
          </p>

          <div class="panel-actions">
            <button type="submit" class="btn-submit" :disabled="submitting">
              <span v-if="submitting" class="btn-spinner" />
              <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
              {{ submitting ? 'Saving…' : 'Save Tentative' }}
            </button>
            <button type="button" class="btn-cancel" @click="isOpen = false">Cancel</button>
          </div>
        </form>
      </div>
    </Transition>
  </section>
</template>

<style scoped>
.tentative-wrap {
  position: relative;
  background: #fffdf8;
  border: 1px solid #ece3d8;
  border-radius: 16px;
  padding: 1rem;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.03);
  margin-top: 1rem;
  overflow: hidden;
}
.tentative-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 0.75rem;
  gap: 1rem;
}
.header-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}
.small-label {
  margin: 0;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: #8f7561;
}
.tentative-header h2 {
  margin: 0.1rem 0 0.35rem;
  font-size: 1.15rem;
  letter-spacing: -0.02em;
}
.subtitle { margin: 0; font-size: 0.9rem; color: #6f5d49; max-width: 480px; }
.status-pill { font-size: 11px; background: #fff6df; color: #a96b16; border-radius: 999px; padding: 0.2rem 0.65rem; font-weight: 600; letter-spacing: 0.02em; }

/* Add button */
.add-btn {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-size: 12px;
  font-weight: 600;
  color: #7c5c36;
  background: #fff8ee;
  border: 1px solid #e8d5b5;
  border-radius: 8px;
  padding: 0.3rem 0.7rem;
  cursor: pointer;
  transition: background 0.18s, box-shadow 0.18s, transform 0.12s;
}
.add-btn:hover {
  background: #fff0d6;
  box-shadow: 0 2px 8px rgba(180,130,60,0.13);
  transform: translateY(-1px);
}
.add-btn:active { transform: translateY(0); }

/* Timeline */
.timeline-list { list-style: none; margin: 0; padding: 0; }
.timeline-item { display: flex; gap: 0.8rem; margin-bottom: 0.8rem; }
.timeline-meta { position: relative; display: flex; flex-direction: column; align-items: center; width: 22px; }
.timeline-dot { width: 12px; height: 12px; border-radius: 50%; border: 2px solid #f0d2a2; background: #fff; margin-top: 4px; }
.timeline-dot.done { background: #22c55e; border-color: #22c55e; }
.timeline-dot.upcoming { background: #f59e0b; border-color: #f59e0b; }
.timeline-dot.pending { background: #a0aec0; border-color: #a0aec0; }
.timeline-line { width: 2px; background: #e3d5c4; flex: 1; margin-top: 4px; border-radius: 999px; }
.timeline-card { background: #fff; border: 1px solid #ede6db; border-radius: 12px; padding: 0.55rem 0.75rem; flex: 1; }
.timeline-top { display: flex; justify-content: space-between; align-items: center; gap: 0.6rem; margin-bottom: 0.25rem; }
.timeline-top-right { display: flex; align-items: center; gap: 0.4rem; }
.timeline-time { margin: 0; font-size: 11px; color: #8c7660; font-weight: 600; text-transform: uppercase; letter-spacing: 0.03em; }
.timeline-status { margin: 0; font-size: 10px; font-weight: 700; text-transform: uppercase; border-radius: 999px; padding: 0.15rem 0.45rem; letter-spacing: 0.02em; background: #f1f5f9; color: #334155; }

/* Delete button */
.delete-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 22px;
  height: 22px;
  background: transparent;
  border: 1px solid transparent;
  border-radius: 6px;
  color: #c0ad99;
  cursor: pointer;
  transition: background 0.15s, color 0.15s, border-color 0.15s, transform 0.1s;
  flex-shrink: 0;
  padding: 0;
}
.delete-btn:hover {
  background: #fff0f0;
  border-color: #fca5a5;
  color: #ef4444;
  transform: scale(1.1);
}
.delete-btn:active { transform: scale(0.97); }
.delete-btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }
.delete-btn--loading { pointer-events: none; }

.delete-spinner {
  width: 10px;
  height: 10px;
  border: 1.5px solid rgba(239,68,68,0.3);
  border-top-color: #ef4444;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
}
.timeline-status.done { background: #e6fffa; color: #0f6f52; }
.timeline-status.upcoming { background: #fff7ed; color: #92400e; }
.timeline-status.pending { background: #f1f5f9; color: #475569; }
.timeline-title { margin: 0; font-size: 14px; color: #2e2a24; }
.timeline-location { margin: 0.2rem 0 0; font-size: 12px; color: #7c6750; }
.timeline-note { margin: 0.35rem 0 0; font-size: 12px; color: #4a4035; }

/* Publish strip */
.publish-strip {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  flex-wrap: wrap;
  margin-top: 0.75rem;
  padding: 0.75rem 1rem;
  background: linear-gradient(135deg, #fffbf0 0%, #fff8e6 100%);
  border: 1px solid #e8d5a3;
  border-radius: 12px;
}
.publish-strip-info {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12.5px;
  color: #7c6040;
  flex: 1;
  min-width: 160px;
}
.publish-feedback {
  display: flex;
  flex-direction: column;
  gap: 2px;
}
.pub-error {
  margin: 0;
  font-size: 11.5px;
  color: #dc2626;
}
.pub-success {
  margin: 0;
  font-size: 11.5px;
  color: #16a34a;
  font-weight: 500;
}
.btn-publish {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 0.45rem 1rem;
  font-size: 12.5px;
  font-weight: 600;
  font-family: inherit;
  color: #fff;
  background: linear-gradient(135deg, #c4a76b 0%, #a8865a 100%);
  border: none;
  border-radius: 9px;
  cursor: pointer;
  transition: opacity 0.18s, transform 0.12s, box-shadow 0.18s;
  box-shadow: 0 2px 8px rgba(168,134,90,0.28);
  white-space: nowrap;
  flex-shrink: 0;
}
.btn-publish:hover:not(:disabled) {
  opacity: 0.9;
  transform: translateY(-1px);
  box-shadow: 0 4px 14px rgba(168,134,90,0.35);
}
.btn-publish:active:not(:disabled) { transform: translateY(0); }
.btn-publish:disabled { opacity: 0.6; cursor: not-allowed; }
.publish-spinner {
  width: 12px;
  height: 12px;
  border: 2px solid rgba(255,255,255,0.35);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
}

/* Backdrop */
.panel-backdrop {
  position: absolute;
  inset: 0;
  background: rgba(44, 36, 22, 0.22);
  backdrop-filter: blur(2px);
  z-index: 10;
  border-radius: 16px;
}

/* Slide-in panel */
.add-panel {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  width: min(340px, 92%);
  background: #fffdf9;
  border-left: 1px solid #e8d8c4;
  box-shadow: -8px 0 32px rgba(0,0,0,0.10);
  border-radius: 0 16px 16px 0;
  z-index: 20;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.panel-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding: 1.25rem 1.25rem 0;
}
.panel-title-group { display: flex; flex-direction: column; gap: 2px; }
.panel-eyebrow {
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: #b89b6a;
  font-weight: 600;
}
.panel-title {
  margin: 0;
  font-size: 1.15rem;
  font-weight: 600;
  color: #2c2416;
  letter-spacing: -0.02em;
}
.panel-close {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f5ede0;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  color: #7c6750;
  transition: background 0.15s, color 0.15s;
  flex-shrink: 0;
  margin-top: 2px;
}
.panel-close:hover { background: #ede0cc; color: #3a2e20; }

/* Form */
.panel-form {
  flex: 1;
  overflow-y: auto;
  padding: 1.1rem 1.25rem 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.field-group { display: flex; flex-direction: column; gap: 0.35rem; }
.field-label {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 12px;
  font-weight: 600;
  color: #7c6448;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}
.field-optional {
  font-size: 10px;
  font-weight: 400;
  color: #b8a48a;
  text-transform: lowercase;
  letter-spacing: 0;
  margin-left: 2px;
}
.field-input {
  width: 100%;
  box-sizing: border-box;
  padding: 0.55rem 0.75rem;
  font-size: 13.5px;
  color: #2c2416;
  background: #fff;
  border: 1px solid #ddd4c4;
  border-radius: 10px;
  outline: none;
  transition: border-color 0.18s, box-shadow 0.18s;
  font-family: inherit;
  resize: none;
}
.field-input::placeholder { color: #c0af99; }
.field-input:focus {
  border-color: #c4a76b;
  box-shadow: 0 0 0 3px rgba(196,167,107,0.14);
}
.field-textarea { line-height: 1.5; }

/* Feedback */
.panel-error {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 12.5px;
  color: #dc2626;
  margin: 0;
  background: #fff1f1;
  border: 1px solid #fecaca;
  border-radius: 8px;
  padding: 0.45rem 0.7rem;
}
.panel-success {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 12.5px;
  color: #16a34a;
  margin: 0;
  background: #f0fdf4;
  border: 1px solid #bbf7d0;
  border-radius: 8px;
  padding: 0.45rem 0.7rem;
}

/* Actions */
.panel-actions {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-top: 0.25rem;
}
.btn-submit {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  width: 100%;
  padding: 0.65rem 1rem;
  font-size: 13.5px;
  font-weight: 600;
  font-family: inherit;
  color: #fff;
  background: linear-gradient(135deg, #c4a76b 0%, #a8865a 100%);
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: opacity 0.18s, transform 0.12s, box-shadow 0.18s;
  box-shadow: 0 2px 10px rgba(168,134,90,0.3);
}
.btn-submit:hover:not(:disabled) {
  opacity: 0.92;
  transform: translateY(-1px);
  box-shadow: 0 4px 16px rgba(168,134,90,0.35);
}
.btn-submit:active:not(:disabled) { transform: translateY(0); }
.btn-submit:disabled { opacity: 0.65; cursor: not-allowed; }

.btn-cancel {
  width: 100%;
  padding: 0.55rem 1rem;
  font-size: 13px;
  font-weight: 500;
  font-family: inherit;
  color: #7c6448;
  background: #f5ede0;
  border: 1px solid #e5d5bc;
  border-radius: 10px;
  cursor: pointer;
  transition: background 0.15s;
}
.btn-cancel:hover { background: #ede0cc; }

/* Spinner */
.btn-spinner {
  width: 13px;
  height: 13px;
  border: 2px solid rgba(255,255,255,0.4);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* Transitions */
.backdrop-fade-enter-active,
.backdrop-fade-leave-active { transition: opacity 0.22s ease; }
.backdrop-fade-enter-from,
.backdrop-fade-leave-to { opacity: 0; }

.panel-slide-enter-active { transition: transform 0.28s cubic-bezier(0.22,1,0.36,1), opacity 0.22s ease; }
.panel-slide-leave-active { transition: transform 0.22s cubic-bezier(0.55,0,1,0.45), opacity 0.18s ease; }
.panel-slide-enter-from { transform: translateX(100%); opacity: 0; }
.panel-slide-leave-to  { transform: translateX(100%); opacity: 0; }
</style>
