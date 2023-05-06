<template>
  <div class="picker__body">
    <div class="time-picker-clock__container">
      <div
        ref="clock"
        @mousedown="onMouseDown"
        @mouseup="onMouseUp"
        @mouseleave="(e: MouseEvent) => (state.isDragging && onMouseUp(e))"
        @touchstart="onMouseDown"
        @touchend="onMouseUp"
        @mousemove="onDragMove"
        @touchmove="onDragMove"
        class="time-picker-clock"
        :class="{
          'time-picker-clock--indeterminate': value == null
        }"
      >
        <div ref="innerClock" class="time-picker-clock__inner">
          <div
            class="time-picker-clock__hand"
            :style="{
              ...clockHandStyle,
              backgroundColor: value !== null ? color : undefined
            }"
            :class="{
              'time-picker-clock__hand--inner': isInner(value)
            }"
          />
          <span
            v-for="v in values"
            :key="v"
            class="time-picker-clock__item"
            :class="{
              'time-picker-clock__item--active': v === displayedValue
            }"
            :style="getTransform(v)"
          >
            <span>
              {{
                selecting === SelectingTimes.Hour
                  ? v
                  : v.toString().padStart(2, "0")
              }}
            </span>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, reactive, ref, watch } from "vue";
import { SelectingTimes } from "./constants";
import { Point } from "./models";

const emit = defineEmits<{
  (e: "input", value: number): void;
  (e: "change", value: number): void;
}>();

const clock = ref<HTMLElement | null>(null);
const innerClock = ref<HTMLElement | null>(null);

const innerRadiusScale = 0.62;

const props = defineProps<{
  disabled?: boolean;
  readonly?: boolean;
  color?: string;
  value: number | null;
  selecting: number;
  min: number;
  max: number;
  step: number;
}>();

const state = reactive({
  inputValue: props.value,
  isDragging: false,
  valueOnMouseDown: null as number | null,
  valueOnMouseUp: null as number | null
});

const values = computed(() => {
  const res = [];
  for (let value = props.min; value <= props.max; value = value + props.step) {
    res.push(value);
  }
  return res;
});

const count = computed(() => props.max - props.min + 1);

const roundCount = computed(() =>
  selectingHour.value ? count.value / 2 : count.value
);

const degreesPerUnit = computed(() => 360 / roundCount.value);

const degrees = computed(() => (degreesPerUnit.value * Math.PI) / 180);

const selectingHour = computed(() => props.selecting === SelectingTimes.Hour);

const clockHandStyle = computed(() => ({
  transform: `rotate(${
    degreesPerUnit.value * (displayedValue.value - props.min)
  }deg) scaleY(${handScale(displayedValue.value)})`
}));

const displayedValue = computed(() =>
  props.value == null ? props.min : props.value
);

const isInner = (value: number | null) => {
  if (value)
    return selectingHour.value && value - props.min >= roundCount.value;
  return false;
};

const handScale = (value: number) => (isInner(value) ? innerRadiusScale : 1);

const getPosition = (value: number) => {
  const rotateRadians = Math.PI / 180;
  return {
    x:
      Math.sin((value - props.min) * degrees.value + rotateRadians) *
      handScale(value),
    y:
      -Math.cos((value - props.min) * degrees.value + rotateRadians) *
      handScale(value)
  };
};

const getTransform = (i: number) => {
  const { x, y } = getPosition(i);
  return {
    backgroundColor:
      i !== null && i === displayedValue.value ? props.color : undefined,
    left: `${50 + x * 50}%`,
    top: `${50 + y * 50}%`
  };
};

const euclidean = (p0: Point, p1: Point) => {
  const dx = p1.x - p0.x;
  const dy = p1.y - p0.y;
  return Math.sqrt(dx * dx + dy * dy);
};

const angle = (center: Point, p1: Point) => {
  const value =
    2 * Math.atan2(p1.y - center.y - euclidean(center, p1), p1.x - center.x);
  return Math.abs((value * 180) / Math.PI);
};

const angleToValue = (angle: number, insideClick: boolean) => {
  const value =
    ((Math.round(angle / degreesPerUnit.value) +
      (insideClick ? roundCount.value : 0)) %
      count.value) +
    props.min;
  if (angle < 360 - degreesPerUnit.value / 2) return value;
  return insideClick ? props.max - roundCount.value + 1 : props.min;
};

const update = (value: number) => {
  if (state.inputValue !== value) {
    state.inputValue = value;
    emit("input", value);
  }
};

const setMouseDownValue = (value: number) => {
  if (state.valueOnMouseDown === null) state.valueOnMouseDown = value;
  state.valueOnMouseUp = value;
  update(value);
};

const onDragMove = (e: MouseEvent | TouchEvent) => {
  e.preventDefault();
  if ((!state.isDragging && e.type !== "click") || !clock.value) return;
  const { width, top, left } = clock.value.getBoundingClientRect();
  const { width: innerWidth } = innerClock.value!.getBoundingClientRect();
  const { clientX, clientY } = "touches" in e ? e.touches[0] : e;
  const center = { x: width / 2, y: -width / 2 };
  const coords = { x: clientX - left, y: top - clientY };
  const handAngle = Math.round(angle(center, coords) - 0 + 360) % 360;
  const insideClick =
    selectingHour.value &&
    euclidean(center, coords) <
      (innerWidth + innerWidth * innerRadiusScale) / 4;
  const checksCount = Math.ceil(15 / degreesPerUnit.value);
  let value;

  for (let i = 0; i < checksCount; i++) {
    value = angleToValue(handAngle + i * degreesPerUnit.value, insideClick);
    return setMouseDownValue(value);
  }
};

const onMouseDown = (e: MouseEvent | TouchEvent) => {
  e.preventDefault();
  state.valueOnMouseDown = null;
  state.valueOnMouseUp = null;
  state.isDragging = true;
  onDragMove(e);
};

const onMouseUp = (e: MouseEvent | TouchEvent) => {
  e.stopPropagation();
  state.isDragging = false;
  if (state.valueOnMouseUp !== null) emit("change", state.valueOnMouseUp);
};

watch(
  () => props.value,
  (val) => (state.inputValue = val)
);
</script>

<style lang="sass" scoped>

.time-picker-clock
  background-color: #eee
  font-family: Roboto
  border-radius: 100%
  position: relative
  transition: 0.3s cubic-bezier(0.25, 0.8, 0.5, 1)
  user-select: none
  width: 100%
  padding-top: 100%
  flex: 1 0 auto

  &__item--disabled
    color: pink

    &.time-picker-clock__item--active
      color: purple

  &--indeterminate
    .time-picker-clock__hand
      background-color: #bdbdbd

    &:after
      color: yellow

    .time-picker-clock__item--active
      background-color: #bdbdbd

  &__container
    display: flex
    flex-direction: column
    flex-basis: 290px
    justify-content: center
    padding: 10px

  &__hand
    height: calc(50% - 4px)
    width: 2px
    bottom: 50%
    left: calc(50% - 1px)
    transform-origin: center bottom
    position: absolute
    will-change: transform
    z-index: 1

    &:before
      background: transparent
      border-width: 2px
      border-style: solid
      border-color: v-bind(color)
      border-radius: 100%
      width: 10px
      height: 10px
      content: ''
      position: absolute
      top: -4px
      left: 50%
      transform: translate(-50%, -50%)

    &:after
      content: ''
      position: absolute
      height: 8px
      width: 8px
      top: 100%
      left: 50%
      border-radius: 100%
      border-style: solid
      border-color: v-bind(color)
      background-color: v-bind(color)
      transform: translate(-50%, -50%)

    &--inner:after
      height: 14px

.picker--full-width
  .time-picker-clock__container
    max-width: 290px

.time-picker-clock__inner
  position: absolute
  bottom: 27px
  left: 27px
  right: 27px
  top: 27px

.time-picker-clock__item
  align-items: center
  border-radius: 100%
  cursor: default
  display: flex
  font-size: 16px
  justify-content: center
  height: 40px
  position: absolute
  text-align: center
  width: 40px
  user-select: none
  transform: translate(-50%, -50%)

  > span
    z-index: 1

  &:before, &:after
    content: ''
    border-radius: 100%
    position: absolute
    top: 50%
    left: 50%
    height: 14px
    width: 14px
    transform: translate(-50%, -50%)

  &:after, &:before
    height: 40px
    width: 40px

  &--active
    color: white
    cursor: default
    z-index: 2

  &--disabled
    pointer-events: none
</style>
