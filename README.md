<div align="center">
  <img src="https://github.com/MatijaNovosel/vue-material-time-picker/assets/36193643/90057a83-6115-41a3-911e-9eab0702c497" />
</div>

<h1 align=center>Vue material time picker</h1>
<p align=center>A material time picker component for Vue 3.</p>
<p align=center>There is currently no support for the AM/PM time notation.</p>

## 🚀 Installation

Install using your package manager of choice:

```bash
yarn add vue-material-time-picker
```

## 📺 Demo

https://matija-components.vercel.app/time-picker

## ⚙️ Usage

Import the component locally or define it globally and include the css file:

```vue
<template>
  <div style="display: flex">
    <time-picker v-model="time" use-seconds />
    {{ time }}
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import { TimePicker } from "vue-material-time-picker";
import "vue-material-time-picker/dist/style.css";
const time = ref(null);
</script>
```

## 📃 Props

| Name          | Type            | Default | Description                                                                                                      |
| ------------- | --------------- | ------- | ---------------------------------------------------------------------------------------------------------------- |
| `v-model`     | `string/Date`   | null    | Standard two way input, HH:mm:ss format or Date                                                                  |
| `disabled`    | `boolean`       | false   | Makes the component uninteractable                                                                               |
| `readonly`    | `boolean`       | false   | Makes the component uninteractable, but without the style of the disabled variant                                |
| `use-seconds` | `boolean`       | false   | Adds an additional step for picking the time                                                                     |
| `automatic`   | `boolean`       | true    | Automatically switches to the next step when picking the time                                                    |
| `hide-title`  | `boolean`       | false   | Hides the time picker title                                                                                      |
| `width`       | `number/string` | 290px   | Sets the width of the element - can be provided as a string like "290px" or "290" or a number, defaults to 290px |
| `full-width`  | `boolean`       | false   | Ignores the previous `width` prop and sets the width to 100% of the parent container                             |
| `color`       | `string`        | #3ba13b | Color of the time picker title and clock hand as well as any active element                                      |

## 🎺 Events

| Name            | Type                      | Description                                                 |
| --------------- | ------------------------- | ----------------------------------------------------------- |
| `select:hour`   | `(value: number) => void` | Triggered when the hour value is selected                   |
| `select:minute` | `(value: number) => void` | Triggered when the minute value is selected                 |
| `select:second` | `(value: number) => void` | Triggered when the second value is selected                 |
| `click:hour`    | `() => void`              | Triggered when the hour notation is clicked on the header   |
| `click:minute`  | `() => void`              | Triggered when the minute notation is clicked on the header |
| `click:second`  | `() => void`              | Triggered when the second notation is clicked on the header |
| `change`        | `(time: string) => void`  | Triggered when the full value is selected                   |

## 🧩 Slots

### header

Use this slot if you want to override the time picker header, an example being:

```vue
<time-picker v-model="time" use-seconds>
  <template
    #header="{
      hours,
      minutes,
      seconds,
      selectHours,
      selectMinutes,
      selectSeconds
    }"
  >
    <button @click="selectHours">
      {{ hours || "--" }}
    </button>
    <button @click="selectMinutes">
      {{ minutes || "--" }}
    </button>
    <button @click="selectSeconds">
      {{ seconds || "--" }}
    </button>
  </template>
</time-picker>
```

There are a few props being exposed:

| Name            | Type          | Default | Description                                                      |
| --------------- | ------------- | ------- | ---------------------------------------------------------------- |
| `hours`         | `number/null` | null    | Hour portion of the time                                         |
| `minutes`       | `number/null` | null    | Minute portion of the time                                       |
| `seconds`       | `number/null` | null    | Seconds portion of the time                                      |
| `selectHours`   | `function`    | N/A     | Helper function used for triggering the selection of the hours   |
| `selectMinutes` | `function`    | N/A     | Helper function used for triggering the selection of the minutes |
| `selectSeconds` | `function`    | N/A     | Helper function used for triggering the selection of the seconds |
