<template>
  <div
    class="picker__title"
    :style="{
      backgroundColor: color
    }"
  >
    <div class="time-picker-title">
      <div class="time-picker-title__time">
        <div
          @click="emit('update:selecting', SelectingTimes.Hour)"
          class="picker__title__btn"
          :class="{
            'picker__title__btn--active': SelectingTimes.Hour === selecting
          }"
        >
          {{ hour == null ? "--" : hour.toString().padStart(2, "0") }}
        </div>
        <span> : </span>
        <div
          @click="emit('update:selecting', SelectingTimes.Minute)"
          class="picker__title__btn"
          :class="{
            'picker__title__btn--active': SelectingTimes.Minute === selecting
          }"
        >
          {{ minute == null ? "--" : minute.toString().padStart(2, "0") }}
        </div>
        <template v-if="useSeconds">
          <span> : </span>
          <div
            @click="emit('update:selecting', SelectingTimes.Second)"
            class="picker__title__btn"
            :class="{
              'picker__title__btn--active': SelectingTimes.Second === selecting
            }"
          >
            {{ second == null ? "--" : second.toString().padStart(2, "0") }}
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { SelectingTimes } from "./constants";

const emit = defineEmits<{
  (e: "update:selecting", type: SelectingTimes): void;
}>();

const props = defineProps<{
  disabled?: boolean;
  hour?: number | null;
  minute?: number | null;
  second?: number | null;
  readonly?: boolean;
  useSeconds?: boolean;
  color?: string;
  selecting: SelectingTimes;
}>();
</script>

<style scoped lang="sass">
.picker
  border-radius: 4px
  contain: layout style
  display: inline-flex
  flex-direction: column
  font-size: 1rem
  vertical-align: top
  position: relative

  &__title
    color: white
    border-top-left-radius: 4px
    border-top-right-radius: 4px
    padding: 16px

    &__btn
      transition: 0.3s cubic-bezier(0.25, 0.8, 0.5, 1)

      &:not(.picker__title__btn--active)
        opacity: 0.6
        cursor: pointer

        &:hover:not(:focus)
          opacity: 1

.time-picker-title
  font-family: Roboto
  color: white
  display: flex
  line-height: 1
  justify-content: flex-end

.time-picker-title__time
  white-space: nowrap
  direction: ltr

  .picker__title__btn,
  span
    align-items: center
    display: inline-flex
    height: 70px
    font-size: 60px
    justify-content: center

.picker--time
  padding: 0

  .time-picker-title__time
    text-align: center
</style>
