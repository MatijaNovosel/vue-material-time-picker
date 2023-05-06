<template>
  <div style="display: flex">
    <time-picker disabled v-model="time" use-seconds />
    {{ time }}
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import timePicker from "./components/timePicker.vue";

const time = ref(null);
</script>
