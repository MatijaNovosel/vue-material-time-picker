<template>
  <div :style="styles">
    <time-picker-title
      v-if="!hideTitle"
      :color="color"
      :hour="state.inputHour"
      :minute="state.inputMinute"
      :second="state.inputSecond"
      :selecting="state.selecting"
      :use-seconds="useSeconds"
      :disabled="disabled"
      :readonly="readonly"
      @update:selecting="(value: SelectingTimes) => (state.selecting = value)"
    />
    <time-picker-clock
      :disabled="disabled"
      :readonly="readonly"
      :color="color"
      @input="onInput"
      @change="onChange"
      :use-seconds="useSeconds"
      :step="state.selecting === SelectingTimes.Hour ? 1 : 5"
      :min="0"
      :max="state.selecting === SelectingTimes.Hour ? 23 : 59"
      :selecting="state.selecting"
      :value="
        state.selecting === SelectingTimes.Hour
          ? state.inputHour
          : state.selecting === SelectingTimes.Minute
          ? state.inputMinute
          : state.inputSecond
      "
    />
  </div>
</template>

<script lang="ts" setup>
import { computed, onMounted, reactive, watch } from "vue";
import { SelectingTimes } from "./constants";
import { convertToUnit } from "./helpers";
import timePickerClock from "./timePickerClock.vue";
import timePickerTitle from "./timePickerTitle.vue";

const emit = defineEmits<{
  (e: "change", time: string): void;
  (e: "click:hour", value: number): void;
  (e: "click:minute", value: number): void;
  (e: "click:second", value: number): void;
  (e: "update:modelValue", value: string): void;
}>();

const props = withDefaults(
  defineProps<{
    disabled?: boolean;
    min?: string;
    max?: string;
    readonly?: boolean;
    hideTitle?: boolean;
    fullWidth?: boolean;
    useSeconds?: boolean;
    automatic?: boolean;
    color?: string;
    modelValue: string | null | Date;
    width?: number | string;
  }>(),
  {
    color: "#3ba13b",
    automatic: true
  }
);

const state = reactive({
  inputHour: null as number | null,
  inputMinute: null as number | null,
  inputSecond: null as number | null,
  lazyInputHour: null as number | null,
  lazyInputMinute: null as number | null,
  lazyInputSecond: null as number | null,
  selecting: SelectingTimes.Hour
});

const setInputData = (value: string | null | Date) => {
  if (value == null || value === "") {
    state.inputHour = null;
    state.inputMinute = null;
    state.inputSecond = null;
  } else if (value instanceof Date) {
    state.inputHour = value.getHours();
    state.inputMinute = value.getMinutes();
    state.inputSecond = value.getSeconds();
  } else if (typeof value === "string") {
    const [h, m, s] = value.split(":");
    state.inputHour = parseInt(h);
    state.inputMinute = parseInt(m);
    state.inputSecond = parseInt(s);
  }
};

const styles = computed(() => ({
  width: props.fullWidth ? undefined : convertToUnit(props.width || 290)
}));

const genValue = () => {
  if (
    state.inputHour != null &&
    state.inputMinute != null &&
    (!props.useSeconds || state.inputSecond != null)
  ) {
    return (
      `${state.inputHour.toString().padStart(2, "0")}:${state.inputMinute
        .toString()
        .padStart(2, "0")}` +
      (props.useSeconds
        ? `:${state.inputSecond!.toString().padStart(2, "0")}`
        : "")
    );
  }
  return null;
};

const emitValue = () => {
  const value = genValue();
  if (value !== null) emit("update:modelValue", value);
};

const onInput = (value: number) => {
  if (state.selecting === SelectingTimes.Hour) state.inputHour = value;
  else if (state.selecting === SelectingTimes.Minute) state.inputMinute = value;
  else state.inputSecond = value;
  emitValue();
};

const onChange = (value: number) => {
  switch (state.selecting) {
    case 1:
      emit("click:hour", value);
      break;
    case 2:
      emit("click:minute", value);
      break;
    case 3:
      emit("click:second", value);
      break;
  }

  const emitChange =
    state.selecting ===
    (props.useSeconds ? SelectingTimes.Second : SelectingTimes.Minute);

  if (props.automatic === true || props.automatic === undefined) {
    if (state.selecting === SelectingTimes.Hour) {
      state.selecting = SelectingTimes.Minute;
    } else if (props.useSeconds && state.selecting === SelectingTimes.Minute) {
      state.selecting = SelectingTimes.Second;
    }
  }

  if (
    state.inputHour === state.lazyInputHour &&
    state.inputMinute === state.lazyInputMinute &&
    (!props.useSeconds || state.inputSecond === state.lazyInputSecond)
  ) {
    return;
  }

  const time = genValue();
  if (time === null) return;

  state.lazyInputHour = state.inputHour;
  state.lazyInputMinute = state.inputMinute;
  props.useSeconds && (state.lazyInputSecond = state.inputSecond);

  emitChange && emit("change", time);
};

onMounted(() => setInputData(props.modelValue));

watch(
  () => props.modelValue,
  (val) => setInputData(val)
);
</script>
