<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && product" class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50 p-4" @click="close">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-lg max-h-[90vh] overflow-hidden flex flex-col" @click.stop>
          <div class="flex items-center justify-between px-6 py-4 border-b border-slate-100">
            <h3 class="text-lg font-semibold text-slate-900">Product Details</h3>
            <button type="button" class="rounded-lg p-2 text-slate-400 hover:text-slate-600 hover:bg-slate-100 transition-colors" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="px-6 py-4 overflow-y-auto flex-1">
            <!-- Product Header -->
            <div class="flex items-center gap-4 pb-4 border-b border-slate-100 mb-5">
              <div class="w-14 h-14 bg-gradient-to-br from-blue-500 to-blue-700 rounded-xl flex items-center justify-center text-white flex-shrink-0">
                <svg width="28" height="28" viewBox="0 0 48 48" fill="none">
                  <rect x="8" y="12" width="32" height="28" rx="2" stroke="currentColor" stroke-width="2.5"/>
                  <path d="M16 8V16M32 8V16M8 20H40" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"/>
                </svg>
              </div>
              <div class="flex-1 min-w-0">
                <h4 class="text-xl font-bold text-slate-900 mb-1">{{ product.name }}</h4>
                <div class="text-sm text-slate-500 font-mono">SKU: {{ product.sku }}</div>
              </div>
              <span :class="['badge', getStockBadgeClass(product.stockLevel)]">{{ product.stockLevel }}</span>
            </div>

            <!-- Info Grid -->
            <div class="grid grid-cols-2 gap-x-6 gap-y-3">
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Category</div>
                <div class="text-sm font-medium text-slate-900">{{ product.category }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Warehouse</div>
                <div class="text-sm font-medium text-slate-900">{{ product.warehouse }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Units Ordered</div>
                <div class="text-sm font-medium text-slate-900">{{ product.unitsOrdered }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Total Revenue</div>
                <div class="text-sm font-medium text-slate-900">{{ currencySymbol }}{{ product.revenue.toLocaleString() }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Current Stock</div>
                <div class="text-sm font-medium text-slate-900">{{ product.quantityOnHand }} units</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Reorder Point</div>
                <div class="text-sm font-medium text-slate-900">{{ product.reorderPoint }} units</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">First Order Date</div>
                <div class="text-sm font-medium text-slate-900">{{ formatDate(product.firstOrderDate) }}</div>
              </div>
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold uppercase tracking-wider text-slate-500">Stock Status</div>
                <div><span :class="['badge', getStockBadgeClass(product.stockLevel)]">{{ product.stockLevel }}</span></div>
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
  product: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close'])

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

const getStockBadgeClass = (stockLevel) => {
  if (stockLevel === 'In Stock') return 'success'
  if (stockLevel === 'Low Stock') return 'warning'
  if (stockLevel === 'Out of Stock') return 'danger'
  return 'info'
}
</script>

<style scoped>
.modal-enter-active, .modal-leave-active { transition: all 0.2s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(0.95); }
</style>
