<script setup>
import { computed, ref } from 'vue'
import AppTabbar from '../components/AppTabbar.vue'
import TaskCard from '../components/TaskCard.vue'
import TaskCounter from '../components/TaskCounter.vue'
import TaskDetails from '../components/TaskDetails.vue'
import { taskState, setTaskStatus, setTaskPriority } from '../store/tasks'
import '../css/task-components.css'

const selectedId = ref(null)

const totalTasks = computed(() => taskState.tasks.length)
const countBy = (status) => taskState.tasks.filter((t) => t.status === status).length
const plannedTasks = computed(() => countBy('todo'))
const progressTasks = computed(() => countBy('progress'))
const completedTasks = computed(() => countBy('done'))
const completedPercent = computed(() =>
  totalTasks.value ? Math.round((completedTasks.value / totalTasks.value) * 100) : 0)

const selectedTask = computed(() =>
  taskState.tasks.find((t) => t.id === selectedId.value) || null)

const activeTasks = computed(() => {
  const order = { high: 0, medium: 1, low: 2 }
  return taskState.tasks
    .filter((t) => t.status !== 'done')
    .sort((a, b) => order[a.priority] - order[b.priority])
    .slice(0, 3)
})

function handleStatusChange(payload) { setTaskStatus(payload) }
function handlePriorityChange(payload) { setTaskPriority(payload) }
function handleTaskSelect(id) { selectedId.value = id }
</script>

<template>
  <f7-page name="home" class="tf3-page">
    <f7-navbar title="TaskFlow">
      <f7-nav-right><f7-link href="/about/">Про застосунок</f7-link></f7-nav-right>
    </f7-navbar>

    <div class="tf3-container">
      <section class="tf3-hero">
        <div class="tf3-eyebrow">TASKFLOW / DASHBOARD</div>
        <h1>Панель керування</h1>
        <p>Контролюйте виконання завдань та їхні пріоритети.</p>
      </section>

      <section class="tf3-stat-grid">
        <TaskCounter label="Усього завдань" :count="totalTasks" />
        <TaskCounter label="Заплановано" :count="plannedTasks" />
        <TaskCounter label="У роботі" :count="progressTasks" />
        <TaskCounter label="Виконано" :count="completedTasks" :total="totalTasks" />
      </section>

      <section class="tf3-progress" aria-label="Відсоток виконаних завдань">
        <div class="tf3-progress-head">
          <span>Виконано завдань</span><strong>{{ completedPercent }}%</strong>
        </div>
        <div class="tf3-progress-track" role="progressbar" aria-valuemin="0"
          aria-valuemax="100" :aria-valuenow="completedPercent">
          <div class="tf3-progress-fill" :style="{ width: completedPercent + '%' }"></div>
        </div>
      </section>

      <TaskDetails :task="selectedTask" @close="selectedId = null" />

      <section class="tf3-section">
        <div class="tf3-section-heading">
          <div><div class="tf3-eyebrow">PRIORITY TASKS</div><h2>Актуальні завдання</h2></div>
          <f7-button href="/tasks/" outline small>Переглянути всі</f7-button>
        </div>
        <div v-if="activeTasks.length" class="tf3-task-grid">
          <TaskCard v-for="task in activeTasks" :key="task.id" :task="task"
            @status-change="handleStatusChange" @priority-change="handlePriorityChange"
            @task-select="handleTaskSelect" />
        </div>
        <div v-else class="tf3-empty">Наразі всі завдання виконані.</div>
      </section>
    </div>

    <AppTabbar active="home" />
  </f7-page>
</template>
