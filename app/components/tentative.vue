<script setup lang="ts">
interface TimelineItem {
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
        <UButton size="xs" color="gray" variant="soft" class="add-btn" @click="isOpen = true">Add Tentative</UButton>
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
            <p class="timeline-status" :class="item.status">{{ item.status === 'done' ? 'Done' : item.status === 'upcoming' ? 'Upcoming' : 'Pending' }}</p>
          </div>
          <h3 class="timeline-title">{{ item.title }}</h3>
          <p class="timeline-location">{{ item.location }}</p>
          <p v-if="item.note" class="timeline-note">{{ item.note }}</p>
        </div>
      </li>
    </ul>

    <UModal v-model="isOpen">
      <div class="modal-wrap">
        <h3 class="modal-title">Add Tentative</h3>
        <UForm @submit.prevent="handleSubmit" class="modal-form">
          <UFormField label="Time" name="Time">
            <UInput v-model="form.time" placeholder="e.g. 3:00 PM" />
          </UFormField>
          <UFormField label="Title" name="Title">
            <UInput v-model="form.title" placeholder="e.g. Ceremony start" />
          </UFormField>
          <UFormField label="Note" name="Note">
            <UInput v-model="form.note" placeholder="Optional details" />
          </UFormField>

          <div class="modal-actions">
            <UButton type="submit" :loading="submitting">Submit</UButton>
            <UButton color="gray" variant="soft" @click="isOpen = false">Cancel</UButton>
          </div>
          <p v-if="submitError" class="error-text">{{ submitError }}</p>
          <p v-if="submitSuccess" class="success-text">{{ submitSuccess }}</p>
        </UForm>
      </div>
    </UModal>
  </section>
</template>

<style scoped>
.tentative-wrap {
  background: #fffdf8;
  border: 1px solid #ece3d8;
  border-radius: 16px;
  padding: 1rem;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.03);
  margin-top: 1rem;
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
.add-btn { margin-left: 4px; }
.timeline-list { list-style: none; margin: 0; padding: 0; }
.timeline-item { display: flex; gap: 0.8rem; margin-bottom: 0.8rem; }
.timeline-meta { position: relative; display: flex; flex-direction: column; align-items: center; width: 22px; }
.timeline-dot { width: 12px; height: 12px; border-radius: 50%; border: 2px solid #f0d2a2; background: #fff; margin-top: 4px; }
.timeline-dot.done { background: #22c55e; border-color: #22c55e; }
.timeline-dot.upcoming { background: #f59e0b; border-color: #f59e0b; }
.timeline-dot.pending { background: #a0aec0; border-color: #a0aec0; }
.timeline-line { width: 2px; background: #e3d5c4; flex: 1; margin-top: 4px; border-radius: 999px; }
.timeline-card { background: #fff; border: 1px solid #ede6db; border-radius: 12px; padding: 0.55rem 0.75rem; flex: 1; }
.timeline-top { display: flex; justify-content: space-between; align-items: baseline; gap: 0.6rem; margin-bottom: 0.25rem; }
.timeline-time { margin: 0; font-size: 11px; color: #8c7660; font-weight: 600; text-transform: uppercase; letter-spacing: 0.03em; }
.timeline-status { margin: 0; font-size: 10px; font-weight: 700; text-transform: uppercase; border-radius: 999px; padding: 0.15rem 0.45rem; letter-spacing: 0.02em; background: #f1f5f9; color: #334155; }
.timeline-status.done { background: #e6fffa; color: #0f6f52; }
.timeline-status.upcoming { background: #fff7ed; color: #92400e; }
.timeline-status.pending { background: #f1f5f9; color: #475569; }
.timeline-title { margin: 0; font-size: 14px; color: #2e2a24; }
.timeline-location { margin: 0.2rem 0 0; font-size: 12px; color: #7c6750; }
.timeline-note { margin: 0.35rem 0 0; font-size: 12px; color: #4a4035; }
.modal-wrap { padding: 1rem; }
.modal-title { margin: 0 0 0.75rem; font-size: 1.1rem; }
.modal-form { display: flex; flex-direction: column; gap: 0.75rem; }
.modal-actions { display: flex; gap: 0.5rem; margin-top: 0.5rem; }
.error-text { color: #ef4444; font-size: 0.85rem; margin: 0.25rem 0 0; }
.success-text { color: #22c55e; font-size: 0.85rem; margin: 0.25rem 0 0; }
</style>
