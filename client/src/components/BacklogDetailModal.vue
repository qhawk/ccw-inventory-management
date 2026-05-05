<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && backlogItem" class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50 p-4" @click="close">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-lg max-h-[90vh] overflow-hidden flex flex-col" @click.stop>
          <div class="flex items-center justify-between px-6 py-4 border-b border-slate-100">
            <h3 class="text-lg font-semibold text-slate-900">Inventory Shortage Details</h3>
            <button type="button" class="rounded-lg p-2 text-slate-400 hover:text-slate-600 hover:bg-slate-100 transition-colors" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="px-6 py-4 overflow-y-auto flex-1">
            <!-- Shortage Header -->
            <div class="flex items-center gap-4 pb-4 border-b border-slate-100 mb-4">
              <div class="w-14 h-14 bg-gradient-to-br from-red-500 to-red-700 rounded-xl flex items-center justify-center text-white flex-shrink-0">
                <svg width="28" height="28" viewBox="0 0 48 48" fill="none">
                  <path d="M24 8L24 28M24 34L24 36" stroke="currentColor" stroke-width="3" stroke-linecap="round"/>
                  <circle cx="24" cy="24" r="18" stroke="currentColor" stroke-width="3"/>
                </svg>
              </div>
              <div class="flex-1 min-w-0">
                <h4 class="text-xl font-bold text-slate-900 mb-1">{{ translateProductName(backlogItem.item_name) }}</h4>
                <div class="text-sm text-slate-500 font-mono">SKU: {{ backlogItem.item_sku }}</div>
              </div>
              <span :class="['badge', backlogItem.priority]">{{ backlogItem.priority }} Priority</span>
            </div>

            <!-- Shortage Summary -->
            <div class="grid grid-cols-2 gap-3 mb-5">
              <div class="p-4 rounded-xl border-2 border-red-200 bg-red-50">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500 mb-1">Shortage Amount</div>
                <div class="text-2xl font-bold text-red-600">{{ shortage }} <span class="text-sm font-medium">units</span></div>
              </div>
              <div class="p-4 rounded-xl border-2 border-amber-200 bg-amber-50">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500 mb-1">Days Delayed</div>
                <div class="text-2xl font-bold text-amber-500">{{ backlogItem.days_delayed }} <span class="text-sm font-medium">days</span></div>
              </div>
            </div>

            <!-- Info Grid -->
            <div class="grid grid-cols-2 gap-x-6 gap-y-3">
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Order ID</div>
                <div class="text-sm font-medium text-indigo-600 font-mono">{{ backlogItem.order_id }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Item SKU</div>
                <div class="text-sm font-medium text-indigo-600 font-mono">{{ backlogItem.item_sku }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Quantity Needed</div>
                <div class="text-sm font-medium text-slate-900">{{ backlogItem.quantity_needed }} units</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Quantity Available</div>
                <div class="text-sm font-medium text-slate-900">{{ backlogItem.quantity_available }} units</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Expected Date</div>
                <div class="text-sm font-medium text-slate-900">{{ formatDate(backlogItem.expected_date) }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Status</div>
                <div><span class="badge danger">Backordered</span></div>
              </div>
            </div>
          </div>

          <div class="flex justify-end gap-3 px-6 py-4 border-t border-slate-100">
            <button class="border border-slate-200 text-slate-600 hover:bg-slate-50 rounded-lg px-4 py-2 text-sm font-medium transition-colors" @click="close">Close</button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { computed } from 'vue'
import { useI18n } from '../composables/useI18n'

const { translateProductName } = useI18n()

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  backlogItem: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close'])

const shortage = computed(() => {
  if (!props.backlogItem) return 0
  return props.backlogItem.quantity_needed - props.backlogItem.quantity_available
})

const close = () => {
  emit('close')
}

const formatDate = (dateString) => {
  if (!dateString) return 'N/A'
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', {
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
