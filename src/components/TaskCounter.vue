<script setup>
defineProps({
  label: { type: String, required: true },
  count: { type: Number, required: true },
  total: { type: Number, default: null },
})
</script>

<template>
  <div class="tf3-stat">
    <span class="tf3-stat-label">{{ label }}</span>
    <strong>{{ count }}<span v-if="total !== null" class="tf3-stat-total"> / {{ total }}</span></strong>
  </div>
</template>
