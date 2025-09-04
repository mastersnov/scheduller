<script setup>
  import { ref, watch, onMounted, onBeforeUnmount, computed } from 'vue';

  const props = defineProps({
    scheduleData: {
      type: Object,
      required: true,
    },
  });
  const emit = defineEmits(['update:scheduleData']);

  const days = [
    { id: 'mo', label: 'MO' },
    { id: 'tu', label: 'TU' },
    { id: 'we', label: 'WE' },
    { id: 'th', label: 'TH' },
    { id: 'fr', label: 'FR' },
    { id: 'sa', label: 'SA' },
    { id: 'su', label: 'SU' },
  ];

  const hours = Array.from(
    { length: 24 },
    (_, i) => i.toString().padStart(2, '0') + ':00'
  );

  // schedule[dayIndex][hour] = true/false
  const schedule = ref(days.map(() => Array(24).fill(false)));

  // Конвертация из JSON в таблицу
  function loadFromJson(data) {
    schedule.value = days.map((day) => {
      const row = Array(24).fill(false);
      const intervals = data[day.id] || [];
      intervals.forEach(({ bt, et }) => {
        for (let m = bt; m <= et; m++) {
          const hour = Math.floor(m / 60);
          row[hour] = true;
        }
      });
      return row;
    });
  }

  // Конвертация из таблицы в JSON
  const getJson = computed(() => {
    const result = {};
    days.forEach((day, di) => {
      const row = schedule.value[di];
      const intervals = [];
      let current = null;
      for (let h = 0; h < 24; h++) {
        if (row[h]) {
          if (!current) {
            current = { bt: h * 60, et: h * 60 + 59 };
          } else {
            current.et = h * 60 + 59;
          }
        } else {
          if (current) {
            intervals.push(current);
            current = null;
          }
        }
      }
      if (current) intervals.push(current);
      result[day.id] = intervals;
    });
    return result;
  });

  // обновление при изменении пропа
  watch(
    () => props.scheduleData,
    (val) => {
      if (val) loadFromJson(val);
    },
    { immediate: true }
  );

  // Логика выделения мышкой
  let isSelecting = false;
  let selectValue = true;

  function startSelection(dayIndex, hourIndex) {
    isSelecting = true;
    selectValue = !schedule.value[dayIndex][hourIndex];
    schedule.value[dayIndex][hourIndex] = selectValue;
  }

  function handleMouseOver(dayIndex, hourIndex) {
    if (!isSelecting) return;
    schedule.value[dayIndex][hourIndex] = selectValue;
  }

  function stopSelection() {
    isSelecting = false;
  }

  onMounted(() => {
    window.addEventListener('mouseup', stopSelection);
  });
  onBeforeUnmount(() => {
    window.removeEventListener('mouseup', stopSelection);
  });

  // All Day
  function isAllDay(dayIndex) {
    return schedule.value[dayIndex].every(Boolean);
  }

  function toggleAllDay(dayIndex) {
    const all = isAllDay(dayIndex);
    schedule.value[dayIndex] = Array(24).fill(!all);
  }

  // Buttons
  function clear() {
    schedule.value = days.map(() => Array(24).fill(false));
    emit('update:scheduleData', getJson.value);
  }

  function emitSave() {
    emit('update:scheduleData', getJson.value);
  }
</script>

<template>
  <div class="scheduler">
    <h3>Set Schedule</h3>

    <div class="table-wrapper">
      <table>
        <thead>
          <tr>
            <th>Day</th>
            <th>All</th>
            <th v-for="h in hours" :key="h">{{ h }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(day, di) in days" :key="day.id">
            <td class="day-label">{{ day.label }}</td>

            <!-- ALL DAY -->
            <td class="allday">
              <input
                type="checkbox"
                :checked="isAllDay(di)"
                @change="toggleAllDay(di)"
              />
            </td>

            <!-- Часы -->
            <td
              v-for="hi in 24"
              :key="`${day.id}-${hi}`"
              class="hour-cell"
              :class="{ active: schedule[di][hi] }"
              @mousedown.prevent="startSelection(di, hi)"
              @mouseover="handleMouseOver(di, hi)"
            ></td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Кнопки -->
    <div class="actions">
      <button @click="clear">Clear</button>
      <button @click="emitSave">Save Changes</button>
    </div>
  </div>
</template>

<style scoped>
  .scheduler {
    max-width: 900px;
    margin: 20px auto;
    font-family: sans-serif;
    user-select: none;
  }

  .table-wrapper {
    overflow-x: auto;
  }

  table {
    border-collapse: collapse;
    width: 100%;
  }

  th,
  td {
    border: 1px solid #ddd;
    text-align: center;
    font-size: 12px;
    padding: 4px;
  }

  .day-label {
    font-weight: bold;
    width: 40px;
  }

  .allday {
    width: 40px;
    background-color: #555;
  }

  .hour-cell {
    width: 20px;
    height: 20px;
    cursor: pointer;
  }

  .hour-cell.active {
    background-color: #ccc;
  }

  .actions {
    margin-top: 12px;
    display: flex;
    justify-content: center;
    gap: 10px;
  }

  button {
    padding: 6px 12px;
    border: none;
    background: #777;
    color: #fff;
    border-radius: 4px;
    cursor: pointer;
  }

  button:hover {
    background: #555;
  }
</style>
