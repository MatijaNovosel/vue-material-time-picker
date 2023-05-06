<div align="center">
  <img src="https://user-images.githubusercontent.com/36193643/236636342-a4f3b025-54a1-4a27-a6d9-9afdfbdd424b.png" />
</div>

<h1 align=center>Vue time picker</h1>
<p align=center>A material time picker component for Vue 3.</p>

## 🚀 Installation

Install using your package manager of choice:

```bash
yarn add material-time-picker
```

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
import { TimePicker } from "material-time-picker";
import "material-time-picker/dist/style.css";
const time = ref(null);
</script>
```

## 📃 Props

| Name          | Type      | Default | Description                                                                       |
| ------------- | --------- | ------- | --------------------------------------------------------------------------------- |
| `v-model`     | `string`  |         | Standard two way input, HH:mm:ss format                                           |
| `disabled`    | `boolean` | false   | Makes the component uninteractable                                                |
| `readonly`    | `boolean` | false   | Makes the component uninteractable, but without the style of the disabled variant |
| `use-seconds` | `boolean` | false   | Adds an additional step for picking the time                                      |
| `automatic`   | `boolean` | true    | Automatically switches to the next step when picking the time                     |
| `color`       | `string`  | #7e7ec2 | Color of the time picker title and clock hand as well as any active element       |
