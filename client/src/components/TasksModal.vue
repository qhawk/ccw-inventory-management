<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen" class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50 p-4" @click="close">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-3xl max-h-[90vh] overflow-hidden flex flex-col" @click.stop>
          <div class="flex items-center justify-between px-6 py-4 border-b border-slate-100">
            <h3 class="text-lg font-semibold text-slate-900">{{ t('tasks.title') }}</h3>
            <button type="button" class="rounded-lg p-2 text-slate-400 hover:text-slate-600 hover:bg-slate-100 transition-colors" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="px-6 py-4 overflow-y-auto flex-1">
            <!-- Add Task Form -->
            <div class="bg-slate-50 rounded-xl p-5 mb-5">
              <div class="flex gap-3 mb-3">
                <div class="flex-1 flex flex-col gap-1.5">
                  <label for="task-title" class="text-sm font-semibold text-slate-600">{{ t('tasks.taskTitle') }}</label>
                  <input
                    id="task-title"
                    v-model="newTask.title"
                    type="text"
                    :placeholder="t('tasks.taskTitlePlaceholder')"
                    class="px-3 py-2.5 border-2 border-slate-200 rounded-lg text-sm focus:outline-none focus:border-indigo-500 font-[inherit] transition-colors"
                    @keyup.enter="handleAddTask"
                  />
                </div>
              </div>

              <div class="flex gap-3 items-end">
                <div class="flex flex-col gap-1.5 flex-1">
                  <label for="task-priority" class="text-sm font-semibold text-slate-600">{{ t('tasks.priority') }}</label>
                  <select
                    id="task-priority"
                    v-model="newTask.priority"
                    class="px-3 py-2.5 border-2 border-slate-200 rounded-lg text-sm focus:outline-none focus:border-indigo-500 font-[inherit] bg-white cursor-pointer transition-colors"
                  >
                    <option value="high">{{ t('priority.high') }}</option>
                    <option value="medium">{{ t('priority.medium') }}</option>
                    <option value="low">{{ t('priority.low') }}</option>
                  </select>
                </div>

                <div class="flex flex-col gap-1.5 flex-1">
                  <label for="task-due-date" class="text-sm font-semibold text-slate-600">{{ t('tasks.dueDate') }}</label>
                  <input
                    id="task-due-date"
                    v-model="newTask.dueDate"
                    type="date"
                    class="px-3 py-2.5 border-2 border-slate-200 rounded-lg text-sm focus:outline-none focus:border-indigo-500 font-[inherit] transition-colors"
                  />
                </div>

                <button
                  type="button"
                  @click="handleAddTask"
                  class="bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 disabled:cursor-not-allowed text-white rounded-lg px-4 py-2.5 text-sm font-medium transition-colors whitespace-nowrap"
                  :disabled="!newTask.title.trim() || !newTask.dueDate"
                >
                  {{ t('tasks.addTask') }}
                </button>
              </div>
            </div>

            <div class="h-px bg-slate-200 mb-5"></div>

            <!-- Tasks List -->
            <div v-if="sortedTasks.length === 0" class="text-center py-10 text-slate-500 text-base italic">
              {{ t('tasks.noTasks') }}
            </div>

            <div v-else class="flex flex-col gap-2.5">
              <div
                v-for="task in sortedTasks"
                :key="task.id"
                class="bg-white border-2 border-slate-200 rounded-xl px-4 py-3 transition-all hover:border-slate-300 hover:shadow-sm"
                :class="{
                  'border-l-4 border-l-red-500': task.priority === 'high',
                  'border-l-4 border-l-amber-500': task.priority === 'medium',
                  'border-l-4 border-l-blue-500': task.priority === 'low',
                  'opacity-60': task.status === 'completed'
                }"
              >
                <div class="flex justify-between items-start mb-2.5 gap-3">
                  <div class="flex items-center gap-2.5 flex-1">
                    <input
                      type="checkbox"
                      :checked="task.status === 'completed'"
                      @change="$emit('toggle-task', task.id)"
                      class="w-5 h-5 cursor-pointer accent-indigo-600 flex-shrink-0"
                    />
                    <span
                      class="flex-1 cursor-pointer select-none text-slate-900 text-sm font-semibold leading-snug"
                      :class="{ 'line-through text-slate-400': task.status === 'completed' }"
                      @click="$emit('toggle-task', task.id)"
                    >{{ task.title }}</span>
                  </div>
                  <button
                    @click="$emit('delete-task', task.id)"
                    class="w-7 h-7 bg-red-500 hover:bg-red-600 text-white rounded-md text-lg leading-none cursor-pointer flex items-center justify-center flex-shrink-0 transition-colors"
                    title="Delete task"
                  >
                    ×
                  </button>
                </div>

                <div class="flex items-center gap-3">
                  <span :class="['badge', task.priority]">{{ translatePriority(task.priority) }}</span>
                  <div class="flex items-center gap-1.5 text-xs text-slate-500">
                    <svg width="14" height="14" viewBox="0 0 14 14" fill="none" class="text-slate-400">
                      <rect x="2" y="3" width="10" height="9" rx="1" stroke="currentColor" stroke-width="1.2"/>
                      <path d="M4.5 1.5V4.5M9.5 1.5V4.5M2 6H12" stroke="currentColor" stroke-width="1.2" stroke-linecap="round"/>
                    </svg>
                    {{ formatDueDate(task.dueDate) }}
                  </div>
                  <span
                    class="ml-auto text-xs font-semibold px-2 py-0.5 rounded"
                    :class="{
                      'bg-red-100 text-red-800': getStatusClass(task.dueDate, task.status) === 'overdue',
                      'bg-amber-100 text-amber-800': getStatusClass(task.dueDate, task.status) === 'urgent',
                      'bg-blue-100 text-blue-800': getStatusClass(task.dueDate, task.status) === 'upcoming',
                      'bg-emerald-100 text-emerald-800': getStatusClass(task.dueDate, task.status) === 'completed'
                    }"
                  >
                    {{ getStatusText(task.dueDate, task.status) }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <div class="flex justify-end gap-3 px-6 py-4 border-t border-slate-100">
            <button class="border border-slate-200 text-slate-600 hover:bg-slate-50 rounded-lg px-4 py-2 text-sm font-medium transition-colors" @click="close">{{ t('profileDetails.close') }}</button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script>
import { ref, computed } from 'vue'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'TasksModal',
  props: {
    isOpen: {
      type: Boolean,
      required: true
    },
    tasks: {
      type: Array,
      default: () => []
    }
  },
  emits: ['close', 'add-task', 'delete-task', 'toggle-task'],
  setup(props, { emit }) {
    const { t, currentLocale } = useI18n()
    const newTask = ref({
      title: '',
      priority: 'medium',
      dueDate: ''
    })

    const sortedTasks = computed(() => {
      // Don't sort - just return tasks in their current order (newest first)
      return [...props.tasks]
    })

    const close = () => {
      emit('close')
    }

    const handleAddTask = () => {
      if (newTask.value.title.trim() && newTask.value.dueDate) {
        emit('add-task', {
          title: newTask.value.title.trim(),
          priority: newTask.value.priority,
          dueDate: newTask.value.dueDate
        })
        newTask.value = {
          title: '',
          priority: 'medium',
          dueDate: ''
        }
      }
    }

    const formatDueDate = (dateString) => {
      const date = new Date(dateString)
      const today = new Date()
      today.setHours(0, 0, 0, 0)
      const dueDate = new Date(date)
      dueDate.setHours(0, 0, 0, 0)

      const diffTime = dueDate - today
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))

      const isJapanese = currentLocale.value === 'ja'

      if (diffDays === 0) return isJapanese ? '今日' : 'today'
      if (diffDays === 1) return isJapanese ? '明日' : 'tomorrow'
      if (diffDays === -1) return isJapanese ? '昨日' : 'yesterday'
      if (diffDays < 0) return isJapanese ? `${Math.abs(diffDays)}日前` : `${Math.abs(diffDays)} days ago`
      if (diffDays < 7) return isJapanese ? `${diffDays}日後` : `in ${diffDays} days`

      const locale = isJapanese ? 'ja-JP' : 'en-US'
      return date.toLocaleDateString(locale, {
        month: 'short',
        day: 'numeric',
        year: date.getFullYear() !== today.getFullYear() ? 'numeric' : undefined
      })
    }

    const getStatusClass = (dueDate, status) => {
      if (status === 'completed') return 'completed'

      const today = new Date()
      today.setHours(0, 0, 0, 0)
      const due = new Date(dueDate)
      due.setHours(0, 0, 0, 0)

      const diffTime = due - today
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))

      if (diffDays < 0) return 'overdue'
      if (diffDays <= 1) return 'urgent'
      return 'upcoming'
    }

    const getStatusText = (dueDate, status) => {
      const isJapanese = currentLocale.value === 'ja'

      if (status === 'completed') return isJapanese ? '完了' : 'Completed'

      const statusClass = getStatusClass(dueDate, status)
      if (statusClass === 'overdue') return isJapanese ? '期限超過' : 'Overdue'
      if (statusClass === 'urgent') return isJapanese ? 'もうすぐ期限' : 'Due Soon'
      return isJapanese ? '予定' : 'Upcoming'
    }

    const translatePriority = (priority) => {
      const priorityMap = {
        'high': t('priority.high'),
        'medium': t('priority.medium'),
        'low': t('priority.low')
      }
      return priorityMap[priority] || priority
    }

    return {
      t,
      newTask,
      sortedTasks,
      close,
      handleAddTask,
      formatDueDate,
      getStatusClass,
      getStatusText,
      translatePriority
    }
  }
}
</script>

<style scoped>
.modal-enter-active, .modal-leave-active { transition: all 0.2s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(0.95); }
</style>
