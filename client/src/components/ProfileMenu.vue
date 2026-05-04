<template>
  <div class="relative">
    <button
      type="button"
      class="flex items-center gap-2.5 px-3 py-2 bg-white border border-slate-200 rounded-lg hover:bg-slate-50 hover:border-slate-300 transition-colors cursor-pointer"
      @click="toggleDropdown"
      @blur="handleBlur"
    >
      <div class="w-8 h-8 rounded-full bg-gradient-to-br from-indigo-500 to-indigo-700 text-white flex items-center justify-center text-xs font-semibold">
        {{ getInitials(currentUser.name) }}
      </div>
      <span class="text-sm font-medium text-slate-700">{{ currentUser.name }}</span>
      <span class="text-slate-400 transition-transform" :class="{ 'rotate-180': isDropdownOpen }">
        <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
          <path d="M4 6L8 10L12 6" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        </svg>
      </span>
    </button>

    <Transition
      enter-active-class="transition duration-100 ease-out origin-top-right"
      enter-from-class="opacity-0 scale-95"
      enter-to-class="opacity-100 scale-100"
      leave-active-class="transition duration-75 ease-in origin-top-right"
      leave-from-class="opacity-100 scale-100"
      leave-to-class="opacity-0 scale-95"
    >
      <div v-if="isDropdownOpen" class="absolute top-[calc(100%+0.5rem)] right-0 min-w-[280px] bg-white border border-slate-200 rounded-xl shadow-xl z-[1000] overflow-hidden">
        <div class="px-4 py-3 flex gap-3.5 items-center bg-slate-50 border-b border-slate-100">
        <div class="w-12 h-12 rounded-full bg-gradient-to-br from-indigo-500 to-indigo-700 text-white flex items-center justify-center text-base font-bold shrink-0">
          {{ getInitials(currentUser.name) }}
        </div>
        <div>
          <div class="text-sm font-semibold text-slate-900">{{ currentUser.name }}</div>
          <div class="text-xs text-slate-500">{{ currentUser.email }}</div>
        </div>
      </div>

      <div class="border-t border-slate-100"></div>

      <button
        type="button"
        class="w-full flex items-center gap-3 px-4 py-3 text-sm font-medium text-slate-700 hover:bg-slate-50 transition-colors text-left bg-transparent border-0 cursor-pointer font-[inherit]"
        @mousedown.prevent="showProfileDetails"
      >
        <svg class="w-4 h-4 text-slate-400" width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M9 9C10.6569 9 12 7.65685 12 6C12 4.34315 10.6569 3 9 3C7.34315 3 6 4.34315 6 6C6 7.65685 7.34315 9 9 9Z" stroke="currentColor" stroke-width="1.5"/>
          <path d="M15 15C15 12.7909 12.3137 11 9 11C5.68629 11 3 12.7909 3 15" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        {{ t('profile.profileDetails') }}
      </button>

      <button
        type="button"
        class="w-full flex items-center gap-3 px-4 py-3 text-sm font-medium text-slate-700 hover:bg-slate-50 transition-colors text-left bg-transparent border-0 cursor-pointer font-[inherit]"
        @mousedown.prevent="showTasks"
      >
        <svg class="w-4 h-4 text-slate-400" width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M15 3H3C2.44772 3 2 3.44772 2 4V14C2 14.5523 2.44772 15 3 15H15C15.5523 15 16 14.5523 16 14V4C16 3.44772 15.5523 3 15 3Z" stroke="currentColor" stroke-width="1.5"/>
          <path d="M6 7L8 9L12 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        {{ t('profile.myTasks') }}
        <span v-if="pendingTaskCount > 0" class="ml-auto bg-indigo-600 text-white text-xs font-semibold px-2 py-0.5 rounded-full min-w-[20px] text-center">{{ pendingTaskCount }}</span>
      </button>

      <div class="border-t border-slate-100"></div>

      <button
        type="button"
        class="w-full flex items-center gap-3 px-4 py-3 text-sm font-medium text-red-600 hover:bg-red-50 transition-colors text-left bg-transparent border-0 cursor-pointer font-[inherit]"
        @mousedown.prevent="handleLogout"
      >
        <svg class="w-4 h-4 text-red-400" width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M7 15H4C3.44772 15 3 14.5523 3 14V4C3 3.44772 3.44772 3 4 3H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M11 12L15 9M15 9L11 6M15 9H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        {{ t('profile.logout') }}
      </button>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useAuth } from '../composables/useAuth'
import { useI18n } from '../composables/useI18n'

const { currentUser, logout, getInitials } = useAuth()
const { t } = useI18n()

const isDropdownOpen = ref(false)
const emit = defineEmits(['show-profile-details', 'show-tasks'])

const pendingTaskCount = computed(() => {
  return currentUser.value.tasks.filter(task => task.status === 'pending').length
})

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value
}

const handleBlur = () => {
  // Delay to allow mousedown events on dropdown items to fire first
  setTimeout(() => {
    isDropdownOpen.value = false
  }, 200)
}

const showProfileDetails = () => {
  isDropdownOpen.value = false
  emit('show-profile-details')
}

const showTasks = () => {
  isDropdownOpen.value = false
  emit('show-tasks')
}

const handleLogout = () => {
  isDropdownOpen.value = false
  logout()
}
</script>
