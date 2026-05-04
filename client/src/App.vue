<template>
  <div class="min-h-screen bg-slate-50 flex flex-col font-sans">
    <header class="sticky top-0 z-[100] bg-white border-b border-slate-200 shadow-sm h-nav">
      <div class="max-w-8xl mx-auto px-8 h-full flex items-center gap-6">
        <div class="flex items-baseline gap-3 flex-shrink-0">
          <h1 class="text-lg font-bold text-slate-900 tracking-tight">{{ t('nav.companyName') }}</h1>
          <span class="text-sm text-slate-400 font-normal border-l border-slate-200 pl-3">{{ t('nav.subtitle') }}</span>
        </div>
        <nav class="flex items-center gap-1 flex-1">
          <router-link
            to="/"
            class="relative px-3 py-2 text-sm font-medium rounded-lg transition-colors"
            :class="$route.path === '/' ? 'text-indigo-600 bg-indigo-50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-100'"
          >
            {{ t('nav.overview') }}
          </router-link>
          <router-link
            to="/inventory"
            class="relative px-3 py-2 text-sm font-medium rounded-lg transition-colors"
            :class="$route.path === '/inventory' ? 'text-indigo-600 bg-indigo-50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-100'"
          >
            {{ t('nav.inventory') }}
          </router-link>
          <router-link
            to="/orders"
            class="relative px-3 py-2 text-sm font-medium rounded-lg transition-colors"
            :class="$route.path === '/orders' ? 'text-indigo-600 bg-indigo-50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-100'"
          >
            {{ t('nav.orders') }}
          </router-link>
          <router-link
            to="/spending"
            class="relative px-3 py-2 text-sm font-medium rounded-lg transition-colors"
            :class="$route.path === '/spending' ? 'text-indigo-600 bg-indigo-50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-100'"
          >
            {{ t('nav.finance') }}
          </router-link>
          <router-link
            to="/demand"
            class="relative px-3 py-2 text-sm font-medium rounded-lg transition-colors"
            :class="$route.path === '/demand' ? 'text-indigo-600 bg-indigo-50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-100'"
          >
            {{ t('nav.demandForecast') }}
          </router-link>
          <router-link
            to="/reports"
            class="relative px-3 py-2 text-sm font-medium rounded-lg transition-colors"
            :class="$route.path === '/reports' ? 'text-indigo-600 bg-indigo-50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-100'"
          >
            Reports
          </router-link>
        </nav>
        <div class="flex items-center gap-2 ml-auto">
          <LanguageSwitcher />
          <ProfileMenu
            @show-profile-details="showProfileDetails = true"
            @show-tasks="showTasks = true"
          />
        </div>
      </div>
    </header>
    <FilterBar />
    <main class="flex-1 max-w-8xl w-full mx-auto px-8 py-6">
      <router-view />
    </main>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>
