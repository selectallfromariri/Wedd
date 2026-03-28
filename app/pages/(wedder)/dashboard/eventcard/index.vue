<script setup lang="ts">
definePageMeta({ layout: 'dashboard-layout' })

const weddingCode = ref('')
const shareOpen   = ref(false)
const copySuccess = ref(false)

// Fetch wedding code on mount
onMounted(async () => {
  try {
    const token = localStorage.getItem('token') ?? ''
    const res = await $fetch('/wedding/show', {
      baseURL: useRuntimeConfig().public.apiBase,
      headers: { Accept: 'application/json', Authorization: `Bearer ${token}` },
    }) as any
    weddingCode.value = res?.data?.wedding_code ?? ''
  } catch {}
})

// Shareable URL
const currentUrl = computed(() => {
  if (import.meta.client && weddingCode.value)
    return `${window.location.origin}/wedding/${weddingCode.value}`
  return ''
})

const shareText = 'You\'re invited to our wedding! Check out our event card 💍'

// WhatsApp deep link
const whatsappUrl = computed(() =>
  `https://wa.me/?text=${encodeURIComponent(shareText + '\n' + currentUrl.value)}`
)

// Telegram
const telegramUrl = computed(() =>
  `https://t.me/share/url?url=${encodeURIComponent(currentUrl.value)}&text=${encodeURIComponent(shareText)}`
)

// Twitter / X
const twitterUrl = computed(() =>
  `https://twitter.com/intent/tweet?text=${encodeURIComponent(shareText)}&url=${encodeURIComponent(currentUrl.value)}`
)

// Facebook
const facebookUrl = computed(() =>
  `https://www.facebook.com/sharer/sharer.php?u=${encodeURIComponent(currentUrl.value)}`
)

async function copyLink() {
  try {
    await navigator.clipboard.writeText(currentUrl.value)
    copySuccess.value = true
    setTimeout(() => { copySuccess.value = false }, 2000)
  } catch {
    // fallback: select input
  }
}

async function nativeShare() {
  if (navigator.share) {
    try {
      await navigator.share({ title: 'Our Wedding Event Card', text: shareText, url: currentUrl.value })
    } catch {}
  } else {
    shareOpen.value = true
  }
}
</script>

<template>
  <div class="ec-page">

    <!-- Page header -->
    <div class="page-header">
      <div>
        <p class="page-eyebrow">Invitation</p>
        <h1 class="page-title">Event Card</h1>
        <p class="page-sub">Preview and share your wedding event card with guests.</p>
      </div>
      <button class="btn-share-main" @click="nativeShare">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <circle cx="18" cy="5" r="3"/><circle cx="6" cy="12" r="3"/><circle cx="18" cy="19" r="3"/>
          <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"/>
          <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"/>
        </svg>
        Share Card
      </button>
    </div>

    <!-- The card component -->
    <Card />

    <!-- Share strip below card -->
    <div class="share-strip">
      <p class="share-strip-label">Share this card</p>
      <div class="share-strip-buttons">

        <!-- WhatsApp -->
        <a :href="whatsappUrl" target="_blank" rel="noopener" class="share-chip share-chip--wa">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
            <path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413z"/>
          </svg>
          WhatsApp
        </a>

        <!-- Telegram -->
        <a :href="telegramUrl" target="_blank" rel="noopener" class="share-chip share-chip--tg">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
            <path d="M11.944 0A12 12 0 0 0 0 12a12 12 0 0 0 12 12 12 12 0 0 0 12-12A12 12 0 0 0 12 0a12 12 0 0 0-.056 0zm4.962 7.224c.1-.002.321.023.465.14a.506.506 0 0 1 .171.325c.016.093.036.306.02.472-.18 1.898-.962 6.502-1.36 8.627-.168.9-.499 1.201-.82 1.23-.696.065-1.225-.46-1.9-.902-1.056-.693-1.653-1.124-2.678-1.8-1.185-.78-.417-1.21.258-1.91.177-.184 3.247-2.977 3.307-3.23.007-.032.014-.15-.056-.212s-.174-.041-.249-.024c-.106.024-1.793 1.14-5.061 3.345-.48.33-.913.49-1.302.48-.428-.008-1.252-.241-1.865-.44-.752-.245-1.349-.374-1.297-.789.027-.216.325-.437.893-.663 3.498-1.524 5.83-2.529 6.998-3.014 3.332-1.386 4.025-1.627 4.476-1.635z"/>
          </svg>
          Telegram
        </a>

        <!-- Twitter / X -->
        <a :href="twitterUrl" target="_blank" rel="noopener" class="share-chip share-chip--tw">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="currentColor">
            <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-4.714-6.231-5.401 6.231H2.744l7.737-8.835L2.25 2.25h6.919l4.262 5.633zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
          </svg>
          X / Twitter
        </a>

        <!-- Facebook -->
        <a :href="facebookUrl" target="_blank" rel="noopener" class="share-chip share-chip--fb">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="currentColor">
            <path d="M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z"/>
          </svg>
          Facebook
        </a>

        <!-- Copy link -->
        <button class="share-chip share-chip--copy" @click="copyLink">
          <svg v-if="!copySuccess" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
            <rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
          </svg>
          <svg v-else width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#16a34a" stroke-width="2.5"><path d="M20 6L9 17l-5-5"/></svg>
          {{ copySuccess ? 'Copied!' : 'Copy Link' }}
        </button>

      </div>

      <!-- URL display -->
      <div class="url-row">
        <span class="url-text">{{ currentUrl }}</span>
        <button class="btn-copy-icon" @click="copyLink" :title="copySuccess ? 'Copied!' : 'Copy URL'">
          <svg v-if="!copySuccess" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
          <svg v-else width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="#16a34a" stroke-width="2.5"><path d="M20 6L9 17l-5-5"/></svg>
        </button>
      </div>
    </div>

  </div>
</template>

<style scoped>
.ec-page {
  max-width: 820px;
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

/* Main share button */
.btn-share-main {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  padding: 0.55rem 1.2rem;
  font-size: 13.5px;
  font-weight: 600;
  font-family: inherit;
  color: #fff;
  background: linear-gradient(135deg, #c4a76b 0%, #a8865a 100%);
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: opacity 0.18s, transform 0.12s, box-shadow 0.18s;
  box-shadow: 0 2px 10px rgba(168,134,90,0.28);
  flex-shrink: 0;
  white-space: nowrap;
}
.btn-share-main:hover { opacity: 0.9; transform: translateY(-1px); }

/* Share strip */
.share-strip {
  background: #fffdf8;
  border: 1px solid #ece3d8;
  border-radius: 16px;
  padding: 1.25rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.share-strip-label {
  margin: 0;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.09em;
  color: #a8967c;
}

/* Share chips */
.share-strip-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
}
.share-chip {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  padding: 0.45rem 1rem;
  font-size: 13px;
  font-weight: 500;
  font-family: inherit;
  border-radius: 999px;
  cursor: pointer;
  text-decoration: none;
  border: 1.5px solid transparent;
  transition: transform 0.13s, box-shadow 0.16s, opacity 0.16s;
  white-space: nowrap;
}
.share-chip:hover { transform: translateY(-2px); box-shadow: 0 4px 14px rgba(0,0,0,0.12); }
.share-chip:active { transform: translateY(0); }

.share-chip--wa {
  background: #25D366;
  color: #fff;
  border-color: #1ebe5d;
}
.share-chip--tg {
  background: #229ED9;
  color: #fff;
  border-color: #1a8fca;
}
.share-chip--tw {
  background: #000;
  color: #fff;
  border-color: #333;
}
.share-chip--fb {
  background: #1877F2;
  color: #fff;
  border-color: #1464d4;
}
.share-chip--copy {
  background: #f5ede0;
  color: #7c5c36;
  border-color: #e8d5b5;
}
.share-chip--copy:hover { background: #fff0d6; }

/* URL row */
.url-row {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  background: #f5f0e8;
  border: 1px solid #e5d8c4;
  border-radius: 10px;
  padding: 0.5rem 0.75rem;
}
.url-text {
  flex: 1;
  font-size: 12px;
  color: #7c6448;
  font-family: 'DM Mono', 'Courier New', monospace;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.btn-copy-icon {
  flex-shrink: 0;
  background: none;
  border: none;
  cursor: pointer;
  color: #a8967c;
  padding: 2px;
  display: flex;
  align-items: center;
  transition: color 0.15s;
}
.btn-copy-icon:hover { color: #c4a76b; }
</style>
