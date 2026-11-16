<template>
  <f7-page name="tasks">
    <f7-navbar title="Мої завдання" />

    <div class="app-content">
      <div class="page-heading">
        <h1>Завдання</h1>
        <p>Список поточних завдань</p>
      </div>

      <div class="task-card-list">
        <TaskCard v-for="task in tasks" :key="task.id" :task="task" />
      </div>

      <p class="helper-text">
        Створення, редагування та видалення завдань — у наступних заняттях.
      </p>
    </div>

    <AppTabbar active="tasks" />
  </f7-page>
</template>

<script setup>
import AppTabbar from '../components/AppTabbar.vue';
import TaskCard from '../components/TaskCard.vue';
import { tasks } from '../data/tasks.js';
</script>
