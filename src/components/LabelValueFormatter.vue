<script setup lang="ts">
const props = defineProps<{
  label: string;
  value: string | number;
  additionalValue?: string | number;
  // monospace?: boolean;
}>();
</script>

<template>
  <dl>
    <dt>
      {{ label }}
    </dt>
    <dd>
      {{ value }}
    </dd>
    <dd v-if="additionalValue">
      {{ additionalValue }}
    </dd>
  </dl>
</template>

<style scoped>
dl {
  dd {
    font-weight: bold;
    font-family: var(--monospace-family);
  }
}
</style>
