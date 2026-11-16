<script setup>
import StatusBadge from './StatusBadge.vue'
import PriorityBadge from './PriorityBadge.vue'

defineProps({ task: { type: Object, default: null } })
const emit = defineEmits(['close'])
</script>

<template>
  <aside v-if="task" class="tf3-detail" role="region" aria-label="Деталі завдання">
    <div class="tf3-detail-head">
      <div class="tf3-eyebrow">ОБРАНЕ ЗАВДАННЯ #{{ task.id }}</div>
      <button type="button" class="tf3-action" @click="emit('close')">Закрити</button>
    </div>
    <h3 class="tf3-detail-title">{{ task.title }}</h3>
    <p class="tf3-card-description">{{ task.description }}</p>
    <div class="tf3-meta-item">
      <StatusBadge :status="task.status" />
      <PriorityBadge :priority="task.priority" />
      <span class="tf3-meta-date">Термін: {{ task.dueDate || 'без терміну' }}</span>
    </div>
  </aside>
</template>
