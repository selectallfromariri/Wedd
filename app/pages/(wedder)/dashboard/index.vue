<script lang="ts">
import card from '~/components/card.vue';
import { defineComponent, h } from 'vue'

definePageMeta({
    layout: 'dashboard-layout'
})

// Inline icon components
const MailIcon = defineComponent({
  props: ['color'],
  render() {
    return h('svg', { width: 18, height: 18, viewBox: '0 0 24 24', fill: 'none', stroke: this.color || '#c4a96b', 'stroke-width': 2 }, [
      h('rect', { x: 2, y: 4, width: 20, height: 16, rx: 2 }),
      h('path', { d: 'M2 7l10 7 10-7' })
    ])
  }
})

const CheckIcon = defineComponent({
  props: ['color'],
  render() {
    return h('svg', { width: 18, height: 18, viewBox: '0 0 24 24', fill: 'none', stroke: this.color || '#22c55e', 'stroke-width': 2.5 }, [
      h('path', { d: 'M20 6L9 17l-5-5' })
    ])
  }
})

const XIcon = defineComponent({
  props: ['color'],
  render() {
    return h('svg', { width: 18, height: 18, viewBox: '0 0 24 24', fill: 'none', stroke: this.color || '#ef4444', 'stroke-width': 2.5 }, [
      h('path', { d: 'M18 6L6 18M6 6l12 12' })
    ])
  }
})

const PhotoIcon = defineComponent({
  props: ['color'],
  render() {
    return h('svg', { width: 18, height: 18, viewBox: '0 0 24 24', fill: 'none', stroke: this.color || '#a8967c', 'stroke-width': 2 }, [
      h('rect', { x: 3, y: 3, width: 18, height: 18, rx: 2 }),
      h('circle', { cx: 8.5, cy: 8.5, r: 1.5 }),
      h('path', { d: 'M21 15l-5-5L5 21' })
    ])
  }
})

const PersonIcon = defineComponent({
  props: ['color'],
  render() {
    return h('svg', { width: 18, height: 18, viewBox: '0 0 24 24', fill: 'none', stroke: this.color || '#c084fc', 'stroke-width': 2 }, [
      h('circle', { cx: 12, cy: 8, r: 4 }),
      h('path', { d: 'M4 20c0-4 3.6-7 8-7s8 3 8 7' })
    ])
  }
})

const CameraIcon = defineComponent({
  props: ['color'],
  render() {
    return h('svg', { width: 18, height: 18, viewBox: '0 0 24 24', fill: 'none', stroke: this.color || '#f59e0b', 'stroke-width': 2 }, [
      h('path', { d: 'M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z' }),
      h('circle', { cx: 12, cy: 13, r: 4 })
    ])
  }
})

const GiftIcon = defineComponent({
  props: ['color'],
  render() {
    return h('svg', { width: 18, height: 18, viewBox: '0 0 24 24', fill: 'none', stroke: this.color || '#10b981', 'stroke-width': 2 }, [
      h('path', { d: 'M20 12v10H4V12' }),
      h('path', { d: 'M2 7h20v5H2z' }),
      h('path', { d: 'M12 22V7' }),
      h('path', { d: 'M12 7H7.5a2.5 2.5 0 0 1 0-5C11 2 12 7 12 7z' }),
      h('path', { d: 'M12 7h4.5a2.5 2.5 0 0 0 0-5C13 2 12 7 12 7z' })
    ])
  }
})

const stats = [
  { label: 'Total Invited', value: '150', icon: MailIcon, iconBg: '#fef3ee', iconColor: '#f97316' },
  { label: 'Attending',     value: '85',  icon: CheckIcon, iconBg: '#f0fdf4', iconColor: '#22c55e' },
  { label: 'Not Attending', value: '12',  icon: XIcon,     iconBg: '#fef2f2', iconColor: '#ef4444' },
  { label: 'Photos',        value: '42',  icon: PhotoIcon, iconBg: '#faf8f4', iconColor: '#a8967c' },
]

const activities = [
  {
    name: 'The Miller Family confirmed attendance',
    description: 'RSVP received for 4 guests',
    time: '2h ago',
    icon: PersonIcon,
    iconBg: '#fdf4ff',
    iconColor: '#c084fc',
  },
  {
    name: 'Sarah Jenkins shared memories',
    description: "Uploaded 5 new photos to the 'Pre-Wedding' album",
    time: '4h ago',
    icon: CameraIcon,
    iconBg: '#fffbeb',
    iconColor: '#f59e0b',
  },
  {
    name: 'Robert & Clara sent a gift',
    description: "Contributed to the 'Honeymoon in Italy' fund",
    time: 'Yesterday',
    icon: GiftIcon,
    iconBg: '#f0fdf4',
    iconColor: '#10b981',
  },
]

const timelineItems = [
  { title: 'Ceremony rehearsal', datetime: '8:30 AM', location: 'Saint Mary Church', status: 'done', note: 'Rings and vows confirmed' },
  { title: 'Venue setup', datetime: '10:00 AM', location: 'Golden Garden Hall', status: 'done', note: 'Florist and decor team arrived' },
  { title: 'Bridal makeup', datetime: '12:30 PM', location: 'Bridal Suite', status: 'upcoming', note: 'Run through hair and makeup' },
  { title: 'Guest arrival', datetime: '2:00 PM', location: 'Garden Entrance', status: 'pending', note: 'Welcome drinks served' },
  { title: 'Ceremony start', datetime: '3:00 PM', location: 'Main Garden', status: 'pending', note: 'Processional and vows' },
  { title: 'Reception dinner', datetime: '6:30 PM', location: 'Grand Ballroom', status: 'pending', note: 'Toasts, first dance, dinner service' },
]

</script>

<style scoped>
.dashboard {
  font-family: 'DM Sans', sans-serif;
  min-height: 100vh;
  padding: 2rem;
  max-width: 900px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  background: #faf8f4;
  color: #2c2416;
}

/* Stats */
.stats-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
}

.stat-card {
  background: #faf8f4;
  border: 1px solid #ede8df;
  border-radius: 16px;
  padding: 1.25rem 1rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  transition: box-shadow 0.2s;
}

.stat-card:hover {
  box-shadow: 0 4px 16px rgba(0,0,0,0.06);
}

.stat-icon {
  width: 44px;
  height: 44px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.stat-label {
  font-size: 12px;
  color: #a8967c;
  margin: 0 0 2px;
  letter-spacing: 0.01em;
}

.stat-value {
  font-size: 1.75rem;
  font-weight: 500;
  color: #2c2416;
  margin: 0;
  font-family: 'Cormorant Garamond', Georgia, serif;
  line-height: 1;
}

/* Activity */
.activity-section {
  background: #faf8f4;
  padding: 1.5rem;
}

.activity-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1.25rem;
}

.activity-title {
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 1.5rem;
  font-style: italic;
  font-weight: 400;
  color: #2c2416;
  margin: 0;
}

.view-all-btn {
  font-size: 13px;
  color: #c4a96b;
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
  font-family: 'DM Sans', sans-serif;
  transition: color 0.2s;
}

.view-all-btn:hover {
  color: #b5975a;
}

.activity-list {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.activity-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem 0;
  border-bottom: 1px solid #ede8df;
  transition: background 0.15s;
  border-radius: 8px;
  padding: 0.875rem 0.75rem;
  margin: 0 -0.75rem;
  cursor: default;
}

.activity-item:last-child {
  border-bottom: none;
}

.activity-item:hover {
  background: #f5f0e8;
}

.activity-icon {
  width: 42px;
  height: 42px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.activity-body {
  flex: 1;
  min-width: 0;
}

.activity-name {
  font-size: 14px;
  font-weight: 500;
  color: #2c2416;
  margin: 0 0 2px;
}

.activity-desc {
  font-size: 13px;
  color: #a8967c;
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.activity-time {
  font-size: 12px;
  color: #bfb09a;
  flex-shrink: 0;
}
</style>

<template>
  <div class="dashboard">
    <!-- Google Fonts -->
    <component :is="'style'">
      @import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,400;0,500;1,400;1,500&family=DM+Sans:wght@300;400;500&display=swap');
    </component>

    <!-- Hero Card Component -->
        <Card/>

    <!-- Stats Row -->
    <div class="stats-row">
      <div
        v-for="stat in stats"
        :key="stat.label"
        class="stat-card"
      >
        <div class="stat-icon" :style="{ background: stat.iconBg }">
          <component :is="stat.icon" :color="stat.iconColor" />
        </div>
        <div class="stat-body">
          <p class="stat-label">{{ stat.label }}</p>
          <p class="stat-value">{{ stat.value }}</p>
        </div>
      </div>
    </div>

    <!-- Recent Activity -->
    <section class="activity-section">
      <div class="activity-header">
        <h2 class="activity-title">Recent Activity</h2>
        <button class="view-all-btn">View All Notifications</button>
      </div>

      <div class="activity-list">
        <div
          v-for="(activity, i) in activities"
          :key="i"
          class="activity-item"
        >
          <div class="activity-icon" :style="{ background: activity.iconBg }">
            <component :is="activity.icon" :color="activity.iconColor" />
          </div>
          <div class="activity-body">
            <p class="activity-name">{{ activity.name }}</p>
            <p class="activity-desc">{{ activity.description }}</p>
          </div>
          <span class="activity-time">{{ activity.time }}</span>
        </div>
      </div>
    </section>

    <tentative :items="timelineItems" />
  </div>
</template>
