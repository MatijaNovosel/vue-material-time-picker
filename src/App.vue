<template>
  <div style="display: flex">
    <time-picker v-model="time" v-if="!hidden" use-seconds> </time-picker>
    {{ time }}
  </div>
  <div style="display: flex">
    <button @click="hide">Hide</button>
    <button @click="setVal">Set val</button>
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import timePicker from "./components/timePicker.vue";

const time = ref<Date | null | string>(null);
const hidden = ref(false);

const setVal = () => {
  time.value = "12:04:04";
};

const hide = () => {
  hidden.value = !hidden.value;
};
</script>
