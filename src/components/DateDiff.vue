<template>
  <div class="date-diff-container">
    <!-- Floating hearts container -->
    <div class="floating-hearts" ref="heartsContainer"></div>

    <div v-if="reminders.length" class="reminder-widgets">
      <div v-for="(reminder, idx) in reminders" :key="idx" class="reminder-widget">
        <span class="reminder-name">{{ reminder.name }}</span>
        <span class="reminder-days">{{ daysSince(reminder.date, reminder.time) }} day<span
            v-if="daysSince(reminder.date, reminder.time) !== 1">s</span></span>
        <span v-if="reminder.time" class="reminder-time">({{ timeSince(reminder.date, reminder.time) }})</span>
      </div>
    </div>
    <h2>Memories <span aria-label="heart" role="img">❤️</span></h2>
    <div v-if="!reminders.length" class="result">
      <p>No reminders yet. Add your special dates in reminders.json!</p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'

const reminders = ref([])
const now = ref(Date.now())
let intervalId = null
const heartsContainer = ref(null)

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

// Function to create floating hearts
function createFloatingHearts(count = 10) {
  if (!heartsContainer.value) return

  for (let i = 0; i < count; i++) {
    const heart = document.createElement('div')
    heart.textContent = '❤️'
    heart.style.position = 'absolute'
    heart.style.fontSize = `${Math.random() * 2 + 1}rem` // Random size
    heart.style.opacity = `${Math.random() * 0.5 + 0.5}` // Random opacity
    heart.style.left = `${Math.random() * 100}%` // Random horizontal position
    heart.style.animation = `floatHearts ${Math.random() * 5 + 5}s infinite ease-in-out`
    heart.style.animationDelay = `${Math.random() * 5}s` // Random delay
    heartsContainer.value.appendChild(heart)
  }
}

onMounted(() => {
  intervalId = setInterval(() => {
    now.value = Date.now()
  }, 1000)
  createFloatingHearts(5)
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
    selected.setHours(0, 0, 0, 0)
    today.setHours(0, 0, 0, 0)
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
@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;700&family=Segoe+UI:wght@400;700&display=swap');

/* Main container styles */
.date-diff-container {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2.5rem 2rem 2rem 2rem;
  background: linear-gradient(135deg, #e6e6fa 0%, #b799ff 40%, #f3e8ff 100%);
  border-radius: 24px;
  /* Enhanced shadow for more depth and subtle glow */
  box-shadow: 0 12px 40px 0 rgba(120, 80, 180, 0.25), 0 3px 12px 0 rgba(183, 153, 255, 0.15);
  color: #2d1836;
  /* Darker text for good contrast */

  /* IMPORTANT FONT CHANGE: Use Segoe UI for general readability within the container */
  font-family: 'Segoe UI', Arial, sans-serif;
  /* Added Arial as a reliable system sans-serif fallback */

  text-align: center;
  position: relative;
  overflow: hidden;
  /* Keeps the pseudo-elements within bounds */

  /* Adding a very subtle, almost transparent border for a frosted glass effect */
  border: 1px solid rgba(255, 255, 255, 0.2);
  /* Softer white border */
  backdrop-filter: blur(2px);
  /* Optional: Adds a very slight blur to the background for a glassy look */
}

/* Pseudo-elements for background decorative swirls */
.date-diff-container::before,
.date-diff-container::after {
  content: '';
  position: absolute;
  border-radius: 50%;
  opacity: 0.18;
  /* Keep the opacity for subtle effect */
  z-index: 0;
  /* Ensures they stay behind content */
}

.date-diff-container::before {
  width: 220px;
  height: 220px;
  background: radial-gradient(circle, #b799ff 0%, #e6e6fa 80%);
  top: -60px;
  left: -80px;
  /* Add a subtle animation for a more dynamic feel */
  animation: swirl1 15s infinite alternate ease-in-out;
}

.date-diff-container::after {
  width: 160px;
  height: 160px;
  background: radial-gradient(circle, #f3e8ff 0%, #b799ff 90%);
  bottom: -40px;
  right: -60px;
  /* Add a subtle animation for a more dynamic feel */
  animation: swirl2 18s infinite alternate-reverse ease-in-out;
}

/* Ensure all actual content stays on top of the decorative elements */
.date-diff-container>* {
  position: relative;
  z-index: 1;
}

/* NEW: Styles for headings and key romantic phrases to use Dancing Script */
/* Apply this to <h1>, <h2>, or any element you want to have the romantic script font */
.date-diff-container h1,
.date-diff-container h2,
.date-diff-container .romantic-text {
  /* Use a class like 'romantic-text' for specific text */
  font-family: 'Dancing Script', cursive, sans-serif;
  font-weight: 700;
  /* Use the bold weight for Dancing Script for impact */
  color: #4a2f5f;
  /* A deeper, richer purple for headings */
  margin-bottom: 1rem;
  /* Space below headings */
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.05);
  /* Subtle text shadow for pop */
}

/* NEW: General paragraph/body text within the container */
.date-diff-container p {
  font-size: 1.05em;
  /* Slightly larger for better readability */
  line-height: 1.6;
  /* Improved line spacing */
  color: #3f264d;
  /* Slightly lighter than the main color, better for paragraphs */
  margin-bottom: 0.8rem;
  /* Space between paragraphs */
}

/* KEYFRAME ANIMATIONS for the swirls (optional, but adds a nice touch) */
@keyframes swirl1 {
  0% {
    transform: translate(0, 0) rotate(0deg) scale(1);
  }

  50% {
    transform: translate(20px, 10px) rotate(5deg) scale(1.05);
  }

  100% {
    transform: translate(0, 0) rotate(0deg) scale(1);
  }
}

@keyframes swirl2 {
  0% {
    transform: translate(0, 0) rotate(0deg) scale(1);
  }

  50% {
    transform: translate(-15px, -8px) rotate(-4deg) scale(1.03);
  }

  100% {
    transform: translate(0, 0) rotate(0deg) scale(1);
  }
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
  color: #8a5fc2;
  /* 10% darker than #b799ff */
  font-size: 0.95rem;
  font-family: 'Segoe UI', cursive, sans-serif;
  margin-top: 0.1rem;
}

/* Floating hearts container */
.floating-hearts {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none; /* Prevent interaction */
  overflow: hidden; /* Ensure hearts stay within the container */
  z-index: 0; /* Ensure it stays behind content */
}

/* Individual heart styles */
.floating-hearts div {
  position: absolute;
  font-size: 2rem;
  animation: floatHearts 6s infinite ease-in-out;
  opacity: 0.8;
  bottom: 0; /* Start from the bottom */
}

/* Keyframe animation for floating hearts */
@keyframes floatHearts {
  0% {
    transform: translateY(100%) scale(0.8); /* Start at the bottom */
    opacity: 0.5;
  }
  50% {
    transform: translateY(50%) scale(1); /* Midway up */
    opacity: 1;
  }
  100% {
    transform: translateY(0) scale(0.8); /* End at the top */
    opacity: 0.5;
  }
}
</style>
