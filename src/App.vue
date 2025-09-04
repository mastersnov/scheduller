<script setup>
  import { computed, ref } from 'vue';
  import WeekScheduller from './components/WeekScheduller.vue';
  const schedule = ref({
    mo: [
      {
        bt: 240,
        et: 779,
      },
    ],
    tu: [],
    we: [],
    th: [
      {
        bt: 240,
        et: 779,
      },
      {
        bt: 1140,
        et: 1319,
      },
    ],
    fr: [
      {
        bt: 660,
        et: 1019,
      },
    ],
    sa: [
      {
        bt: 0,
        et: 1439,
      },
    ],
    su: [],
  });

  const output = computed(() => JSON.stringify(schedule.value, null, 2));

  function handleUpdate(newData) {
    schedule.value = newData;
  }
</script>

<template>
  <WeekScheduller
    :schedule-data="schedule"
    @update:schedule-data="handleUpdate"
  />
  <pre
    style="
      margin-top: 2rem;
      background: #f6f8fa;
      padding: 1rem;
      border-radius: 6px;
    "
  >
    <code>{{ output }}</code>
  </pre>
</template>
