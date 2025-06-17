<template>
  <div class="date-diff-container">
    <div v-if="reminders.length" class="reminder-widgets">
      <div v-for="(reminder, idx) in reminders" :key="idx" class="reminder-widget">
        <span class="reminder-name">{{ reminder.name }}</span>
        <span class="reminder-days">{{ daysSince(reminder.date) }} day<span v-if="daysSince(reminder.date) !== 1">s</span></span>
      </div>
    </div>
    <h2>Reminders</h2>
    <form @submit.prevent="addDate">
      <div class="form-group">
        <label for="nameInput">Occasion:</label>
        <input type="text" id="nameInput" v-model="inputName" placeholder="e.g. Anniversary" required />
      </div>
      <div class="form-group">
        <label for="dateInput">Select a date:</label>
        <input type="date" id="dateInput" v-model="inputDate" required />
      </div>
      <button type="submit">Add Reminder</button>
    </form>
    <div v-if="reminders.length" class="result">
      <div v-for="(reminder, idx) in reminders" :key="'list-' + idx" class="reminder-item">
        <p>
          <span class="reminder-name">{{ reminder.name }}</span>:
          <strong>{{ daysSince(reminder.date) }}</strong> day<span v-if="daysSince(reminder.date) !== 1">s</span> since {{ reminder.date }}
        </p>
      </div>
    </div>
    <div v-else class="result">
      <p>No reminders yet. Add your special dates above!</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const inputName = ref('')
const inputDate = ref('')
const reminders = ref([])

function addDate() {
  if (!inputName.value.trim() || !inputDate.value) return
  reminders.value.push({ name: inputName.value.trim(), date: inputDate.value })
  inputName.value = ''
  inputDate.value = ''
}

function daysSince(dateStr) {
  const selected = new Date(dateStr)
  const today = new Date()
  selected.setHours(0,0,0,0)
  today.setHours(0,0,0,0)
  const diff = today - selected
  return Math.floor(diff / (1000 * 60 * 60 * 24))
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
.form-group {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  margin-bottom: 1rem;
}
label {
  margin-bottom: 0.3rem;
  color: #7c5e99;
  font-size: 1.1rem;
  font-family: 'Segoe UI', cursive, sans-serif;
}
input[type="date"]::-webkit-input-placeholder,
input[type="date"]::placeholder {
  color: #b799ff;
  font-style: italic;
  opacity: 1;
}
input[type="date"]::-webkit-calendar-picker-indicator {
  filter: hue-rotate(270deg) brightness(1.2) saturate(1.5);
}
input[type="date"] {
  border: 1px solid #c3aed6;
  border-radius: 6px;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  background: #f3e8ff;
  color: #2d1836;
  font-family: 'Segoe UI', cursive, sans-serif;
  margin-bottom: 1rem;
  position: relative;
}
input[type="date"]:before {
  content: attr(data-placeholder);
  color: #b799ff;
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
  font-style: italic;
  font-size: 1rem;
}
input[type="text"] {
  border: 1px solid #c3aed6;
  border-radius: 6px;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  background: #f3e8ff;
  color: #2d1836;
  font-family: 'Segoe UI', cursive, sans-serif;
  margin-bottom: 1rem;
}
button {
  background: #b799ff;
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: 0.6rem 1.2rem;
  font-size: 1.1rem;
  font-family: 'Dancing Script', 'Segoe UI', cursive, sans-serif;
  margin-top: 1rem;
  cursor: pointer;
  transition: background 0.2s;
}
button:hover {
  background: #a084ca;
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
</style>
