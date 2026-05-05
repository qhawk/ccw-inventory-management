<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && costData" class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50 p-4" @click="close">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-lg max-h-[90vh] overflow-hidden flex flex-col" @click.stop>
          <div class="flex items-center justify-between px-6 py-4 border-b border-slate-100">
            <h3 class="text-lg font-semibold text-slate-900">{{ costData.month }} Cost Breakdown</h3>
            <button type="button" class="rounded-lg p-2 text-slate-400 hover:text-slate-600 hover:bg-slate-100 transition-colors" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="px-6 py-4 overflow-y-auto flex-1">
            <!-- Total Cost Summary -->
            <div class="mb-5">
              <div class="p-5 rounded-xl bg-gradient-to-br from-blue-500 to-blue-700 text-white text-center">
                <div class="text-xs font-semibold uppercase tracking-wider opacity-90 mb-1">Total Costs</div>
                <div class="text-3xl font-bold">{{ currencySymbol }}{{ totalCosts.toLocaleString() }}</div>
              </div>
            </div>

            <!-- Cost Breakdown -->
            <div class="flex flex-col gap-3">
              <!-- Procurement -->
              <div class="p-4 rounded-xl border-2 border-blue-200 bg-blue-50">
                <div class="flex items-center gap-3 mb-1.5">
                  <div class="w-10 h-10 rounded-lg bg-blue-500 text-white flex items-center justify-center flex-shrink-0">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
                      <rect x="4" y="6" width="16" height="14" rx="2" stroke="currentColor" stroke-width="2"/>
                      <path d="M8 6V4M16 6V4M4 10H20" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="flex-1">
                    <div class="font-semibold text-slate-900 text-sm">Procurement</div>
                    <div class="text-xl font-bold text-slate-900">{{ currencySymbol }}{{ costData.procurement.toLocaleString() }}</div>
                  </div>
                </div>
                <div class="text-sm text-slate-500 font-medium">{{ getProcurementPercentage() }}% of total</div>
              </div>

              <!-- Operational -->
              <div class="p-4 rounded-xl border-2 border-violet-200 bg-violet-50">
                <div class="flex items-center gap-3 mb-1.5">
                  <div class="w-10 h-10 rounded-lg bg-violet-500 text-white flex items-center justify-center flex-shrink-0">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
                      <circle cx="12" cy="12" r="8" stroke="currentColor" stroke-width="2"/>
                      <path d="M12 8V12L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="flex-1">
                    <div class="font-semibold text-slate-900 text-sm">Operational</div>
                    <div class="text-xl font-bold text-slate-900">{{ currencySymbol }}{{ costData.operational.toLocaleString() }}</div>
                  </div>
                </div>
                <div class="text-sm text-slate-500 font-medium">{{ getOperationalPercentage() }}% of total</div>
              </div>

              <!-- Labor -->
              <div class="p-4 rounded-xl border-2 border-emerald-200 bg-emerald-50">
                <div class="flex items-center gap-3 mb-1.5">
                  <div class="w-10 h-10 rounded-lg bg-emerald-500 text-white flex items-center justify-center flex-shrink-0">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
                      <circle cx="12" cy="8" r="4" stroke="currentColor" stroke-width="2"/>
                      <path d="M6 20C6 16.6863 8.68629 14 12 14C15.3137 14 18 16.6863 18 20" stroke="currentColor" stroke-width="2"/>
                    </svg>
                  </div>
                  <div class="flex-1">
                    <div class="font-semibold text-slate-900 text-sm">Labor</div>
                    <div class="text-xl font-bold text-slate-900">{{ currencySymbol }}{{ costData.labor.toLocaleString() }}</div>
                  </div>
                </div>
                <div class="text-sm text-slate-500 font-medium">{{ getLaborPercentage() }}% of total</div>
              </div>

              <!-- Overhead -->
              <div class="p-4 rounded-xl border-2 border-amber-200 bg-amber-50">
                <div class="flex items-center gap-3 mb-1.5">
                  <div class="w-10 h-10 rounded-lg bg-amber-500 text-white flex items-center justify-center flex-shrink-0">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
                      <path d="M3 12L5 10M5 10L12 3L19 10M5 10V20C5 20.5523 5.44772 21 6 21H9M19 10L21 12M19 10V20C19 20.5523 18.5523 21 18 21H15M9 21C9 21 9 18 9 16C9 14 10 14 12 14C14 14 15 14 15 16C15 18 15 21 15 21M9 21H15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="flex-1">
                    <div class="font-semibold text-slate-900 text-sm">Overhead</div>
                    <div class="text-xl font-bold text-slate-900">{{ currencySymbol }}{{ costData.overhead.toLocaleString() }}</div>
                  </div>
                </div>
                <div class="text-sm text-slate-500 font-medium">{{ getOverheadPercentage() }}% of total</div>
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

const { currentCurrency } = useI18n()

const currencySymbol = computed(() => {
  return currentCurrency.value === 'JPY' ? '¥' : '$'
})

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  costData: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close'])

const totalCosts = computed(() => {
  if (!props.costData) return 0
  return props.costData.procurement + props.costData.operational +
         props.costData.labor + props.costData.overhead
})

const getProcurementPercentage = () => {
  if (!props.costData || totalCosts.value === 0) return 0
  return ((props.costData.procurement / totalCosts.value) * 100).toFixed(1)
}

const getOperationalPercentage = () => {
  if (!props.costData || totalCosts.value === 0) return 0
  return ((props.costData.operational / totalCosts.value) * 100).toFixed(1)
}

const getLaborPercentage = () => {
  if (!props.costData || totalCosts.value === 0) return 0
  return ((props.costData.labor / totalCosts.value) * 100).toFixed(1)
}

const getOverheadPercentage = () => {
  if (!props.costData || totalCosts.value === 0) return 0
  return ((props.costData.overhead / totalCosts.value) * 100).toFixed(1)
}

const close = () => {
  emit('close')
}
</script>

<style scoped>
.modal-enter-active, .modal-leave-active { transition: all 0.2s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(0.95); }
</style>
