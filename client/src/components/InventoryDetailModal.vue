<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && inventoryItem" class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50 p-4" @click="close">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-lg max-h-[90vh] overflow-hidden flex flex-col" @click.stop>
          <div class="flex items-center justify-between px-6 py-4 border-b border-slate-100">
            <h3 class="text-lg font-semibold text-slate-900">Inventory Item Details</h3>
            <button type="button" class="rounded-lg p-2 text-slate-400 hover:text-slate-600 hover:bg-slate-100 transition-colors" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="px-6 py-4 overflow-y-auto flex-1">
            <!-- Item Header -->
            <div class="flex items-center gap-4 pb-4 border-b border-slate-100 mb-4">
              <div
                class="w-14 h-14 rounded-xl flex items-center justify-center text-white flex-shrink-0"
                :class="{
                  'bg-gradient-to-br from-emerald-500 to-emerald-700': getStockIconClass() === 'success-icon',
                  'bg-gradient-to-br from-amber-500 to-amber-700': getStockIconClass() === 'warning-icon',
                  'bg-gradient-to-br from-red-500 to-red-700': getStockIconClass() === 'danger-icon'
                }"
              >
                <svg width="28" height="28" viewBox="0 0 48 48" fill="none">
                  <rect x="8" y="12" width="32" height="28" rx="2" stroke="currentColor" stroke-width="2.5"/>
                  <path d="M16 8V16M32 8V16M8 20H40" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"/>
                  <path d="M16 28H32M16 34H24" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"/>
                </svg>
              </div>
              <div class="flex-1 min-w-0">
                <h4 class="text-xl font-bold text-slate-900 mb-1">{{ translateProductName(inventoryItem.name) }}</h4>
                <div class="text-sm text-slate-500 font-mono">SKU: {{ inventoryItem.sku }}</div>
              </div>
              <span :class="['badge', getStockStatusClass()]">{{ getStockStatus() }}</span>
            </div>

            <!-- Stock Summary -->
            <div class="grid grid-cols-2 gap-3 mb-5">
              <div class="p-4 rounded-xl border-2 border-blue-200 bg-blue-50">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500 mb-1">Quantity on Hand</div>
                <div class="text-2xl font-bold text-slate-900">{{ inventoryItem.quantity_on_hand }} <span class="text-sm font-medium">units</span></div>
              </div>
              <div
                class="p-4 rounded-xl border-2"
                :class="{
                  'border-emerald-200 bg-emerald-50': getSummaryCardClass() === 'success-card',
                  'border-amber-200 bg-amber-50': getSummaryCardClass() === 'warning-card',
                  'border-red-200 bg-red-50': getSummaryCardClass() === 'danger-card'
                }"
              >
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500 mb-1">Stock Level</div>
                <div class="text-2xl font-bold text-slate-900">{{ stockPercentage }}%</div>
                <div class="text-xs text-slate-500 mt-0.5">vs. reorder point</div>
              </div>
            </div>

            <!-- Info Grid -->
            <div class="grid grid-cols-2 gap-x-6 gap-y-3">
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Category</div>
                <div class="text-sm font-medium text-slate-900">{{ inventoryItem.category }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Location</div>
                <div class="text-sm font-medium text-slate-900">{{ inventoryItem.location }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Reorder Point</div>
                <div class="text-sm font-medium text-slate-900">{{ inventoryItem.reorder_point }} units</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Units Remaining</div>
                <div class="text-sm font-medium" :class="inventoryItem.quantity_on_hand <= inventoryItem.reorder_point ? 'text-red-500' : 'text-emerald-600'">
                  {{ inventoryItem.quantity_on_hand - inventoryItem.reorder_point }} units
                </div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Unit Cost</div>
                <div class="text-sm font-medium text-slate-900">{{ currencySymbol }}{{ inventoryItem.unit_cost.toFixed(2) }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Total Value</div>
                <div class="text-base font-bold text-indigo-600">
                  {{ currencySymbol }}{{ totalValue.toLocaleString(undefined, {minimumFractionDigits: 2, maximumFractionDigits: 2}) }}
                </div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Warehouse</div>
                <div class="text-sm font-medium text-slate-900">{{ translateWarehouse(inventoryItem.location) }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Status</div>
                <div><span :class="['badge', getStockStatusClass()]">{{ getStockStatus() }}</span></div>
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

const { currentCurrency, translateProductName, translateWarehouse } = useI18n()

const currencySymbol = computed(() => {
  return currentCurrency.value === 'JPY' ? '¥' : '$'
})

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  inventoryItem: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close'])

const totalValue = computed(() => {
  if (!props.inventoryItem) return 0
  return props.inventoryItem.quantity_on_hand * props.inventoryItem.unit_cost
})

const stockPercentage = computed(() => {
  if (!props.inventoryItem || props.inventoryItem.reorder_point === 0) return 0
  return Math.round((props.inventoryItem.quantity_on_hand / props.inventoryItem.reorder_point) * 100)
})

const close = () => {
  emit('close')
}

const getStockStatus = () => {
  if (!props.inventoryItem) return 'Unknown'
  if (props.inventoryItem.quantity_on_hand <= props.inventoryItem.reorder_point) {
    return 'Low Stock'
  } else if (props.inventoryItem.quantity_on_hand <= props.inventoryItem.reorder_point * 1.5) {
    return 'Adequate'
  } else {
    return 'In Stock'
  }
}

const getStockStatusClass = () => {
  const status = getStockStatus()
  if (status === 'Low Stock') return 'danger'
  if (status === 'Adequate') return 'warning'
  return 'success'
}

const getStockIconClass = () => {
  const status = getStockStatus()
  if (status === 'Low Stock') return 'danger-icon'
  if (status === 'Adequate') return 'warning-icon'
  return 'success-icon'
}

const getSummaryCardClass = () => {
  const status = getStockStatus()
  if (status === 'Low Stock') return 'danger-card'
  if (status === 'Adequate') return 'warning-card'
  return 'success-card'
}
</script>

<style scoped>
.modal-enter-active, .modal-leave-active { transition: all 0.2s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(0.95); }
</style>
