<script setup>
import { computed, reactive, ref } from 'vue'

const props = defineProps({
  task: { type: Object, default: null },
  existingTitles: { type: Array, default: () => [] },
})
const emit = defineEmits(['save', 'cancel'])

const defaults = () => ({
  title: props.task?.title ?? '',
  description: props.task?.description ?? '',
  status: props.task?.status ?? 'todo',
  priority: props.task?.priority ?? 'medium',
  dueDate: props.task?.dueDate ?? '',
  category: props.task?.category || 'study',
})

// Копіюємо поля: форма не змінює вихідний об'єкт до натискання «Зберегти».
const form = reactive(defaults())

const showErrors = ref(false)
const isEditing = computed(() => props.task !== null)
const statusLabels = {
  todo: 'Заплановано',
  progress: 'У роботі',
  done: 'Виконано',
}
const priorityLabels = {
  low: 'Низький',
  medium: 'Середній',
  high: 'Високий',
}
const categoryLabels = {
  study: 'Навчання',
  work: 'Робота',
  personal: 'Особисте',
}

function todayString() {
  const now = new Date()
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const day = String(now.getDate()).padStart(2, '0')
  return `${now.getFullYear()}-${month}-${day}`
}

function isValidDate(value) {
  if (!/^\d{4}-\d{2}-\d{2}$/.test(value)) return false
  const date = new Date(`${value}T12:00:00`)
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return !Number.isNaN(date.getTime()) &&
    `${date.getFullYear()}-${month}-${day}` === value
}

const errors = computed(() => {
  const result = {}
  const title = form.title.trim()
  const description = form.description.trim()

  if (!title) result.title = 'Вкажіть назву завдання.'
  else if (title.length < 3) result.title = 'Назва має містити щонайменше 3 символи.'
  else if (title.length > 80) result.title = 'Максимальна довжина — 80 символів.'
  else if (!isEditing.value && props.existingTitles
    .some((item) => item.trim().toLowerCase() === title.toLowerCase())) {
    result.title = 'Завдання з такою назвою вже існує.'
  }

  if (description.length > 300) {
    result.description = 'Опис не повинен перевищувати 300 символів.'
  }
  if (!['todo', 'progress', 'done'].includes(form.status)) {
    result.status = 'Оберіть коректний статус.'
  }
  if (!['low', 'medium', 'high'].includes(form.priority)) {
    result.priority = 'Оберіть коректний пріоритет.'
  }
  if (!['study', 'work', 'personal'].includes(form.category)) {
    result.category = 'Оберіть категорію.'
  }
  if (!isValidDate(form.dueDate)) {
    result.dueDate = 'Укажіть коректну дату завершення.'
  } else if (form.dueDate < todayString() &&
    // Додаткове завдання 2: минулу дату забороняємо для нових завдань
    // і для змінених дат; стара дата редагованого завдання лишається дозволеною.
    !(isEditing.value && form.dueDate === props.task.dueDate)) {
    result.dueDate = 'Дата завершення не може бути в минулому.'
  }
  return result
})

const titleLeft = computed(() => 80 - form.title.length)
const descriptionLeft = computed(() => 300 - form.description.length)

function submitForm() {
  showErrors.value = true
  if (Object.keys(errors.value).length > 0) return

  emit('save', {
    title: form.title.trim(),
    description: form.description.trim(),
    status: form.status,
    priority: form.priority,
    dueDate: form.dueDate,
    category: form.category,
  })
}

// Додаткове завдання 4: очищення форми (лише у режимі створення)
function resetForm() {
  Object.assign(form, defaults())
  showErrors.value = false
}
</script>

<template>
  <div class="tf4-form-panel">
    <div class="tf4-form-heading">
      <div>
        <div class="tf3-eyebrow">TASKFLOW / FORM</div>
        <h2>{{ isEditing ? 'Редагування завдання' : 'Нове завдання' }}</h2>
      </div>
      <button type="button" class="tf4-close" @click="emit('cancel')">
        Закрити
      </button>
    </div>

    <p class="tf4-legend"><span class="tf4-required" aria-hidden="true">*</span> — обов'язкове поле</p>

    <form class="tf4-form" novalidate @submit.prevent="submitForm">
      <div class="tf4-field tf4-field--wide">
        <label for="tf4-title">Назва завдання <span class="tf4-required" aria-hidden="true">*</span></label>
        <input id="tf4-title" v-model.trim="form.title" type="text"
          placeholder="Наприклад: Підготувати звіт" maxlength="80" required
          aria-required="true"
          :aria-invalid="showErrors && !!errors.title"
          aria-describedby="tf4-title-hint tf4-title-error" />
        <small id="tf4-title-hint">Від 3 до 80 символів. Залишилось: {{ titleLeft }}</small>
        <p v-if="showErrors && errors.title" id="tf4-title-error" class="tf4-error">
          {{ errors.title }}
        </p>
      </div>

      <div class="tf4-field tf4-field--wide">
        <label for="tf4-description">Опис</label>
        <textarea id="tf4-description" v-model.trim="form.description"
          placeholder="Що потрібно зробити?" rows="3" maxlength="300"
          :aria-invalid="showErrors && !!errors.description"
          aria-describedby="tf4-description-hint tf4-description-error"></textarea>
        <small id="tf4-description-hint">Необов'язкове поле. Залишилось: {{ descriptionLeft }}</small>
        <p v-if="showErrors && errors.description" id="tf4-description-error" class="tf4-error">
          {{ errors.description }}
        </p>
      </div>

      <div class="tf4-field">
        <label for="tf4-status">Статус <span class="tf4-required" aria-hidden="true">*</span></label>
        <select id="tf4-status" v-model="form.status" aria-required="true"
          :aria-invalid="showErrors && !!errors.status"
          aria-describedby="tf4-status-error">
          <option value="todo">Заплановано</option>
          <option value="progress">У роботі</option>
          <option value="done">Виконано</option>
        </select>
        <p v-if="showErrors && errors.status" id="tf4-status-error" class="tf4-error">{{ errors.status }}</p>
      </div>

      <div class="tf4-field">
        <label for="tf4-priority">Пріоритет <span class="tf4-required" aria-hidden="true">*</span></label>
        <select id="tf4-priority" v-model="form.priority" aria-required="true"
          :aria-invalid="showErrors && !!errors.priority"
          aria-describedby="tf4-priority-error">
          <option value="low">Низький</option>
          <option value="medium">Середній</option>
          <option value="high">Високий</option>
        </select>
        <p v-if="showErrors && errors.priority" id="tf4-priority-error" class="tf4-error">{{ errors.priority }}</p>
      </div>

      <div class="tf4-field">
        <label for="tf4-category">Категорія <span class="tf4-required" aria-hidden="true">*</span></label>
        <select id="tf4-category" v-model="form.category" aria-required="true"
          :aria-invalid="showErrors && !!errors.category"
          aria-describedby="tf4-category-error">
          <option value="study">Навчання</option>
          <option value="work">Робота</option>
          <option value="personal">Особисте</option>
        </select>
        <p v-if="showErrors && errors.category" id="tf4-category-error" class="tf4-error">{{ errors.category }}</p>
      </div>

      <div class="tf4-field">
        <label for="tf4-date">Дата завершення <span class="tf4-required" aria-hidden="true">*</span></label>
        <input id="tf4-date" v-model="form.dueDate" type="date" required
          aria-required="true"
          :aria-invalid="showErrors && !!errors.dueDate"
          aria-describedby="tf4-date-error" />
        <p v-if="showErrors && errors.dueDate" id="tf4-date-error" class="tf4-error">
          {{ errors.dueDate }}
        </p>
      </div>

      <div class="tf4-preview tf4-field--wide" aria-live="polite">
        <strong>Попередній перегляд</strong>
        <p>{{ form.title || 'Назва нового завдання' }}</p>
        <span>{{ statusLabels[form.status] }} · {{ priorityLabels[form.priority] }} · {{ categoryLabels[form.category] }}</span>
      </div>

      <div class="tf4-actions tf4-field--wide">
        <button type="submit" class="tf4-save">
          {{ isEditing ? 'Зберегти зміни' : 'Створити завдання' }}
        </button>
        <button v-if="!isEditing" type="button" class="tf4-cancel" @click="resetForm">
          Очистити форму
        </button>
        <button type="button" class="tf4-cancel" @click="emit('cancel')">
          Скасувати
        </button>
      </div>
    </form>
  </div>
</template>
