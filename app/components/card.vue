<template>
  <div class="hero-card">
    <div class="hero-image">
      <img src="https://images.unsplash.com/photo-1519225421980-715cb0215aed?w=600&q=80" alt="Wedding arch" />
      <div class="image-overlay" />
    </div>

    <div class="hero-content">
      <div class="bolt-badge">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <path d="M13 2L4.5 13.5H11L10 22L20.5 9.5H14L13 2Z" fill="currentColor"/>
        </svg>
      </div>

      <p class="eyebrow">The Big Day</p>
      <h1 class="couple-name">Emma &amp; Liam</h1>

      <div class="meta-row">
        <div class="meta-item">
          <span class="meta-label">Countdown</span>
          <span class="meta-value">120 Days to go</span>
        </div>
        <div class="meta-item">
          <span class="meta-label">Venue</span>
          <span class="meta-value">Fairmont Grand Del Mar</span>
        </div>
      </div>

      <button class="publish-btn">
        Publish Event Card
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M5 12h14M12 5l7 7-7 7"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
// WeddingHeroCard — standalone hero component
</script>

<style scoped>
.hero-card {
  display: grid;
  grid-template-columns: 1fr 1fr;
  overflow: hidden;
  background: #faf8f4;
  min-height: 280px;
  border: 1px solid #e9e0d6;
  border-radius: 16px;
}

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

.hero-card:hover .hero-image img {
  transform: scale(1.03);
}

.image-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to right, transparent 70%, #faf8f4);
}

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
  width: 48px;
  height: 48px;
  background: #f0ebe0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #c4a96b;
}

.eyebrow {
  font-family: 'Cormorant Garamond', Georgia, serif;
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

.meta-row {
  display: flex;
  gap: 2rem;
  margin-top: 0.75rem;
}

.meta-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.meta-label {
  font-size: 10px;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #a8967c;
  font-family: 'Cormorant Garamond', Georgia, serif;
}

.meta-value {
  font-size: 15px;
  font-weight: 500;
  color: #2c2416;
  font-family: 'Cormorant Garamond', Georgia, serif;
}

.publish-btn {
  margin-top: auto;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: #c4a96b;
  color: #fff;
  border: none;
  border-radius: 50px;
  padding: 0.75rem 1.5rem;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  width: fit-content;
  transition: background 0.2s, transform 0.15s;
  font-family: inherit;
  letter-spacing: 0.02em;
}

.publish-btn:hover {
  background: #b5975a;
  transform: translateY(-1px);
}

.publish-btn:active {
  transform: translateY(0);
}
</style>