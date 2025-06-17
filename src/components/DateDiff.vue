<template>
  <div class="date-diff-container">
    <div v-if="reminders.length" class="reminder-widgets">
      <div v-for="(reminder, idx) in reminders" :key="idx" class="reminder-widget">
        <span class="reminder-name">{{ reminder.name }}</span>
        <span class="reminder-days">{{ daysSince(reminder.date, reminder.time) }} day<span v-if="daysSince(reminder.date, reminder.time) !== 1">s</span></span>
        <span v-if="reminder.time" class="reminder-time">({{ timeSince(reminder.date, reminder.time) }})</span>
      </div>
    </div>
    <h2>Reminders</h2>
    <div v-if="reminders.length" class="result">
      <div v-for="(reminder, idx) in reminders" :key="'list-' + idx" class="reminder-item">
        <p>
          <span class="reminder-name">{{ reminder.name }}</span>:
          <strong>{{ daysSince(reminder.date, reminder.time) }}</strong> day<span v-if="daysSince(reminder.date, reminder.time) !== 1">s</span> since {{ reminder.date }}<span v-if="reminder.time"> {{ reminder.time }}</span>
          <span v-if="reminder.time" class="reminder-time">({{ timeSince(reminder.date, reminder.time) }})</span>
        </p>
      </div>
    </div>
    <div v-else class="result">
      <p>No reminders yet. Add your special dates in reminders.json!</p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'

const reminders = ref([])
const now = ref(Date.now())
let intervalId = null

// Load reminders from localStorage or reminders.json
async function loadReminders() {
  // Always try to load reminders.json on first load
  try {
    const res = await fetch('reminders.json?_=' + Date.now()) // cache-busting
    if (res.ok) {
      reminders.value = await res.json()
      return
    }
  } catch (e) {
    reminders.value = []
  }
  // Fallback to localStorage if fetch fails
  if (localStorage.getItem('reminders')) {
    try {
      reminders.value = JSON.parse(localStorage.getItem('reminders'))
    } catch (e) {
      reminders.value = []
    }
  }
}

loadReminders()

// Persist reminders to localStorage whenever they change
watch(reminders, (val) => {
  localStorage.setItem('reminders', JSON.stringify(val))
}, { deep: true })

onMounted(() => {
  intervalId = setInterval(() => {
    now.value = Date.now()
  }, 1000)
})

onUnmounted(() => {
  if (intervalId) clearInterval(intervalId)
})

function daysSince(dateStr, timeStr) {
  // If no time, use midnight
  const selected = new Date(dateStr + (timeStr ? 'T' + timeStr : 'T00:00'))
  const today = new Date()
  // Only zero out time if no timeStr is provided
  if (!timeStr) {
    selected.setHours(0,0,0,0)
    today.setHours(0,0,0,0)
  }
  const diff = today - selected
  return Math.floor(diff / (1000 * 60 * 60 * 24))
}

function timeSince(dateStr, timeStr) {
  if (!timeStr) return ''
  const selected = new Date(dateStr + 'T' + timeStr)
  let diff = now.value - selected.getTime()
  if (diff < 0) return 'in the future'
  const days = Math.floor(diff / (1000 * 60 * 60 * 24))
  diff -= days * (1000 * 60 * 60 * 24)
  const hours = Math.floor(diff / (1000 * 60 * 60))
  diff -= hours * (1000 * 60 * 60)
  const minutes = Math.floor(diff / (1000 * 60))
  diff -= minutes * (1000 * 60)
  const seconds = Math.floor(diff / 1000)
  return `${days}d ${hours}h ${minutes}m ${seconds}s ago`
}
</script>

<style scoped>
.date-diff-container {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2.5rem 2rem 2rem 2rem;
  background: linear-gradient(135deg, #e6e6fa 0%, #b799ff 40%, #f3e8ff 100%);
  border-radius: 24px;
  box-shadow: 0 8px 32px 0 rgba(120, 80, 180, 0.18), 0 1.5px 8px 0 rgba(183, 153, 255, 0.10);
  color: #2d1836;
  font-family: 'Dancing Script', 'Segoe UI', cursive, sans-serif;
  text-align: center;
  position: relative;
  overflow: hidden;
}
.date-diff-container::before, .date-diff-container::after {
  content: '';
  position: absolute;
  border-radius: 50%;
  opacity: 0.18;
  z-index: 0;
}
.date-diff-container::before {
  width: 220px;
  height: 220px;
  background: radial-gradient(circle, #b799ff 0%, #e6e6fa 80%);
  top: -60px;
  left: -80px;
}
.date-diff-container::after {
  width: 160px;
  height: 160px;
  background: radial-gradient(circle, #f3e8ff 0%, #b799ff 90%);
  bottom: -40px;
  right: -60px;
}
.date-diff-container > * {
  position: relative;
  z-index: 1;
}
h2 {
  color: #a084ca;
  font-family: 'Dancing Script', 'Segoe UI', cursive, sans-serif;
  font-size: 2.2rem;
  margin-bottom: 1.2rem;
  letter-spacing: 1px;
}
.result {
  margin-top: 2rem;
  font-size: 1.3rem;
  color: #7c5e99;
  font-family: 'Dancing Script', 'Segoe UI', cursive, sans-serif;
}
.reminder-item {
  margin-bottom: 1.2rem;
  background: rgba(183, 153, 255, 0.13);
  border-radius: 8px;
  padding: 0.8rem 1rem;
  box-shadow: 0 1px 4px rgba(120, 80, 180, 0.06);
}
.reminder-name {
  color: #a084ca;
  font-weight: bold;
  font-family: 'Dancing Script', 'Segoe UI', cursive, sans-serif;
  font-size: 1.15rem;
}
.reminder-widgets {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  justify-content: center;
  margin-bottom: 1.5rem;
}
.reminder-widget {
  background: linear-gradient(135deg, #e6e6fa 60%, #b799ff 100%);
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(120, 80, 180, 0.10);
  padding: 0.7rem 1.3rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 120px;
  font-family: 'Dancing Script', 'Segoe UI', cursive, sans-serif;
}
.reminder-widget .reminder-name {
  color: #a084ca;
  font-size: 1.1rem;
  font-weight: bold;
  margin-bottom: 0.2rem;
}
.reminder-widget .reminder-days {
  color: #7c5e99;
  font-size: 1.05rem;
}
.reminder-time {
  display: block;
  color: #8a5fc2; /* 10% darker than #b799ff */
  font-size: 0.95rem;
  font-family: 'Segoe UI', cursive, sans-serif;
  margin-top: 0.1rem;
}
</style>
