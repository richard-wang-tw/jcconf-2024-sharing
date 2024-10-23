<script setup lang="ts">
import { ref } from "vue";

const props = defineProps({
  now: {
    default: (() => {
      const date = new Date();
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, "0"); // 月份从0开始，需要加1
      const day = String(date.getDate()).padStart(2, "0");
      return `Richard Wang @ ${year}-${month}-${day}`;
    })(),
  },
});

const now = ref(props.now);
</script>

<template>
  <div>{{ now }}</div>
</template>
