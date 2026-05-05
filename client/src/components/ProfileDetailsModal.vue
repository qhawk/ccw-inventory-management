<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen" class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50 p-4" @click="close">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-lg max-h-[90vh] overflow-hidden flex flex-col" @click.stop>
          <div class="flex items-center justify-between px-6 py-4 border-b border-slate-100">
            <h3 class="text-lg font-semibold text-slate-900">{{ t('profileDetails.title') }}</h3>
            <button type="button" class="rounded-lg p-2 text-slate-400 hover:text-slate-600 hover:bg-slate-100 transition-colors" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="px-6 py-4 overflow-y-auto flex-1">
            <div class="flex flex-col gap-6">
              <!-- Avatar Section -->
              <div class="flex flex-col items-center gap-2 pb-5 border-b border-slate-100">
                <div class="w-24 h-24 rounded-full bg-gradient-to-br from-indigo-500 to-indigo-700 text-white flex items-center justify-center font-semibold text-2xl">
                  {{ getInitials(currentUser.name) }}
                </div>
                <h4 class="text-xl font-bold text-slate-900">{{ currentUser.name }}</h4>
                <p class="text-base text-slate-500">{{ currentUser.jobTitle }}</p>
              </div>

              <!-- Info Grid -->
              <div class="grid grid-cols-2 gap-x-6 gap-y-3">
                <div class="flex flex-col gap-1">
                  <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">{{ t('profileDetails.email') }}</div>
                  <div class="text-sm font-medium text-slate-900">{{ currentUser.email }}</div>
                </div>
                <div class="flex flex-col gap-1">
                  <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">{{ t('profileDetails.department') }}</div>
                  <div class="text-sm font-medium text-slate-900">{{ currentUser.department }}</div>
                </div>
                <div class="flex flex-col gap-1">
                  <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">{{ t('profileDetails.location') }}</div>
                  <div class="text-sm font-medium text-slate-900">{{ currentUser.location }}</div>
                </div>
                <div class="flex flex-col gap-1">
                  <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">{{ t('profileDetails.phone') }}</div>
                  <div class="text-sm font-medium text-slate-900">{{ currentUser.phone }}</div>
                </div>
                <div class="flex flex-col gap-1">
                  <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">{{ t('profileDetails.joinDate') }}</div>
                  <div class="text-sm font-medium text-slate-900">{{ formatDate(currentUser.joinDate) }}</div>
                </div>
                <div class="flex flex-col gap-1">
                  <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">{{ t('profileDetails.employeeId') }}</div>
                  <div class="text-sm font-medium text-slate-900">CC-{{ currentUser.id.toString().padStart(5, '0') }}</div>
                </div>
              </div>
            </div>
          </div>

          <div class="flex justify-end gap-3 px-6 py-4 border-t border-slate-100">
            <button type="button" class="border border-slate-200 text-slate-600 hover:bg-slate-50 rounded-lg px-4 py-2 text-sm font-medium transition-colors" @click="close">{{ t('profileDetails.close') }}</button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { useAuth } from '../composables/useAuth'
import { useI18n } from '../composables/useI18n'

const { currentUser, getInitials } = useAuth()
const { t, currentLocale } = useI18n()

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['close'])

const close = () => {
  emit('close')
}

const formatDate = (dateString) => {
  const date = new Date(dateString)
  const locale = currentLocale.value === 'ja' ? 'ja-JP' : 'en-US'
  return date.toLocaleDateString(locale, {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}
</script>

<style scoped>
.modal-enter-active, .modal-leave-active { transition: all 0.2s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(0.95); }
</style>
