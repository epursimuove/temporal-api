<script setup lang="ts">
const props = defineProps<{
  label: string;
  metaLabel?: string;
  value: string | number;
  additionalValue?: string | number;
  // monospace?: boolean;
}>();
</script>

<template>
  <dl>
    <dt>
      {{ label }}
      <span v-if="metaLabel" class="meta">({{ metaLabel }})</span>
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
  dt {
    .meta {
      color: darkgray;
      font-size: 0.85rem;
    }
  }
  
  dd {
    font-weight: bold;
    font-family: var(--monospace-family);
  }
}
</style>
