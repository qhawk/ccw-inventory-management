<template>
  <div class="bg-slate-50 min-h-full p-6">
    <div class="page-header">
      <h2>{{ t('finance.title') }}</h2>
      <p>{{ t('finance.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <!-- Revenue & Financial KPIs -->
      <div class="stats-grid mb-8">
        <div class="stat-card border-l-4 border-indigo-500">
          <div class="text-sm font-medium text-slate-500 mb-1">{{ t('finance.totalRevenue') }}</div>
          <div class="text-2xl font-bold text-slate-900">{{ formatCurrency(revenueMetrics.totalRevenue) }}</div>
          <div class="mt-3 text-sm font-semibold text-emerald-600 flex items-center gap-1">
            <span class="font-bold text-base">↑</span>
            {{ t('finance.fromOrders', { count: revenueMetrics.orderCount }) }}
          </div>
        </div>
        <div class="stat-card border-l-4 border-red-500">
          <div class="text-sm font-medium text-slate-500 mb-1">{{ t('finance.totalCosts') }}</div>
          <div class="text-2xl font-bold text-slate-900">{{ formatCurrency(totalCosts) }}</div>
          <div class="mt-2 text-xs text-slate-500">{{ t('finance.costBreakdown') }}</div>
        </div>
        <div class="stat-card border-l-4 border-emerald-500">
          <div class="text-sm font-medium text-slate-500 mb-1">{{ t('finance.netProfit') }}</div>
          <div class="text-2xl font-bold text-slate-900">{{ formatCurrency(netProfit) }}</div>
          <div class="mt-2 text-xs text-slate-500">{{ profitMargin }}% {{ t('finance.margin') }}</div>
        </div>
        <div class="stat-card border-l-4 border-slate-900">
          <div class="text-sm font-medium text-slate-500 mb-1">{{ t('finance.avgOrderValue') }}</div>
          <div class="text-2xl font-bold text-slate-900">{{ formatCurrency(revenueMetrics.avgOrderValue) }}</div>
          <div class="mt-2 text-xs text-slate-500">{{ t('finance.perOrderRevenue') }}</div>
        </div>
      </div>

      <!-- Monthly Revenue vs Cost Chart -->
      <div class="card mb-7">
        <div class="card-header">
          <h3 class="card-title">{{ t('finance.revenueVsCosts.title') }}</h3>
          <div class="flex gap-6 text-sm">
            <span class="flex items-center gap-2 text-slate-500">
              <span class="inline-block w-3 h-3 rounded-sm bg-slate-900"></span>
              {{ t('finance.revenueVsCosts.revenue') }}
            </span>
            <span class="flex items-center gap-2 text-slate-500">
              <span class="inline-block w-3 h-3 rounded-sm bg-red-500"></span>
              {{ t('finance.revenueVsCosts.costs') }}
            </span>
          </div>
        </div>
        <div class="px-6 py-6">
          <div class="flex gap-6 h-[350px]">
            <div class="flex flex-col justify-between pr-4 text-xs text-slate-400 border-r border-slate-200">
              <span>{{ currencySymbol }}{{ maxRevenueValue }}K</span>
              <span>{{ currencySymbol }}{{ Math.round(maxRevenueValue * 0.75) }}K</span>
              <span>{{ currencySymbol }}{{ Math.round(maxRevenueValue * 0.5) }}K</span>
              <span>{{ currencySymbol }}{{ Math.round(maxRevenueValue * 0.25) }}K</span>
              <span>{{ currencySymbol }}0</span>
            </div>
            <div class="flex-1 flex items-end justify-around gap-2">
              <div v-for="month in monthlyRevenue" :key="month.month" class="flex flex-col items-center flex-1 h-full">
                <div class="w-full max-w-[80px] flex gap-1.5 justify-center items-end h-full pb-8">
                  <div
                    class="w-1/2 max-w-[30px] rounded-t-md bg-slate-900 transition-all duration-300 cursor-pointer min-h-[4px] hover:opacity-80"
                    :style="{ height: getRevenueBarHeight(month.revenue) + '%' }"
                    :title="`Revenue: ${currencySymbol}${month.revenue.toLocaleString()}`"
                  ></div>
                  <div
                    class="w-1/2 max-w-[30px] rounded-t-md bg-red-500 transition-all duration-300 cursor-pointer min-h-[4px] hover:opacity-80"
                    :style="{ height: getRevenueBarHeight(month.costs) + '%' }"
                    :title="`Costs: ${currencySymbol}${month.costs.toLocaleString()}`"
                  ></div>
                </div>
                <span class="mt-2 text-xs font-semibold text-slate-500">{{ translateMonth(month.month) }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Monthly Cost Flow Chart -->
      <div class="card mb-7">
        <div class="card-header">
          <h3 class="card-title">{{ t('finance.monthlyCostFlow.title') }}</h3>
          <div class="flex gap-6 text-sm">
            <span class="flex items-center gap-2 text-slate-500">
              <span class="inline-block w-3 h-3 rounded-sm bg-indigo-500"></span>
              {{ t('finance.monthlyCostFlow.procurement') }}
            </span>
            <span class="flex items-center gap-2 text-slate-500">
              <span class="inline-block w-3 h-3 rounded-sm bg-violet-500"></span>
              {{ t('finance.monthlyCostFlow.operational') }}
            </span>
            <span class="flex items-center gap-2 text-slate-500">
              <span class="inline-block w-3 h-3 rounded-sm bg-emerald-500"></span>
              {{ t('finance.monthlyCostFlow.labor') }}
            </span>
            <span class="flex items-center gap-2 text-slate-500">
              <span class="inline-block w-3 h-3 rounded-sm bg-amber-500"></span>
              {{ t('finance.monthlyCostFlow.overhead') }}
            </span>
          </div>
        </div>
        <div class="px-6 py-6">
          <div class="flex gap-6 h-[350px]">
            <div class="flex flex-col justify-between pr-4 text-xs text-slate-400 border-r border-slate-200">
              <span>{{ currencySymbol }}25K</span>
              <span>{{ currencySymbol }}20K</span>
              <span>{{ currencySymbol }}15K</span>
              <span>{{ currencySymbol }}10K</span>
              <span>{{ currencySymbol }}5K</span>
              <span>{{ currencySymbol }}0</span>
            </div>
            <div class="flex-1 flex items-end justify-around gap-2">
              <div v-for="month in monthlySpending" :key="month.month" class="flex flex-col items-center flex-1 h-full">
                <div
                  class="w-full max-w-[60px] flex flex-col-reverse items-stretch h-full pb-8 cursor-pointer transition-opacity duration-200 hover:opacity-85"
                  @click="showCostDetail(month)"
                >
                  <div class="bar-segment-procurement w-full transition-all duration-300 cursor-pointer block rounded-b-md" :style="{ height: getBarHeight(month.procurement) + '%' }" :title="`Procurement: ${currencySymbol}${month.procurement.toLocaleString()}`"></div>
                  <div class="bar-segment-operational w-full transition-all duration-300 cursor-pointer block" :style="{ height: getBarHeight(month.operational) + '%' }" :title="`Operational: ${currencySymbol}${month.operational.toLocaleString()}`"></div>
                  <div class="bar-segment-labor w-full transition-all duration-300 cursor-pointer block" :style="{ height: getBarHeight(month.labor) + '%' }" :title="`Labor: ${currencySymbol}${month.labor.toLocaleString()}`"></div>
                  <div class="bar-segment-overhead w-full transition-all duration-300 cursor-pointer block rounded-t-md" :style="{ height: getBarHeight(month.overhead) + '%' }" :title="`Overhead: ${currencySymbol}${month.overhead.toLocaleString()}`"></div>
                </div>
                <span class="mt-2 text-xs font-semibold text-slate-500">{{ translateMonth(month.month) }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="grid gap-7" style="grid-template-columns: repeat(auto-fit, minmax(450px, 1fr))">
        <!-- Category Spending Breakdown -->
        <div class="card">
          <div class="card-header">
            <h3 class="card-title">{{ t('finance.categorySpending.title') }}</h3>
          </div>
          <div class="flex flex-col gap-6 px-6 py-5">
            <div v-for="category in categorySpending" :key="category.category" class="flex flex-col gap-2">
              <div class="flex justify-between items-center">
                <div class="font-semibold text-slate-900">{{ translateCategory(category.category) }}</div>
                <div class="font-bold text-indigo-600 text-lg">{{ currencySymbol }}{{ category.amount.toLocaleString() }}</div>
              </div>
              <div class="w-full h-2 bg-slate-100 rounded overflow-hidden">
                <div
                  class="h-full rounded transition-[width] duration-500"
                  :style="{ width: category.percentage + '%', background: 'linear-gradient(90deg, #6366f1 0%, #4f46e5 100%)' }"
                ></div>
              </div>
              <div class="flex justify-between text-xs font-medium">
                <span class="text-slate-500">{{ category.percentage }}% {{ t('finance.categorySpending.ofTotal') }}</span>
                <span
                  class="font-semibold"
                  :class="category.change > 0 ? 'text-emerald-600' : category.change < 0 ? 'text-red-600' : 'text-slate-500'"
                >
                  {{ category.change > 0 ? '+' : '' }}{{ category.change }}%
                </span>
              </div>
            </div>
          </div>
        </div>

        <!-- Recent Transactions -->
        <div class="card flex flex-col">
          <div class="card-header">
            <h3 class="card-title">{{ t('finance.transactions.title') }}</h3>
          </div>
          <div class="overflow-y-auto max-h-[400px]">
            <table class="w-full border-collapse">
              <thead class="sticky top-0 bg-slate-50 z-10">
                <tr>
                  <th class="text-left px-3 py-2.5 font-semibold text-slate-500 text-xs uppercase tracking-wider border-b border-slate-200">{{ t('finance.transactions.id') }}</th>
                  <th class="text-left px-3 py-2.5 font-semibold text-slate-500 text-xs uppercase tracking-wider border-b border-slate-200">{{ t('finance.transactions.description') }}</th>
                  <th class="text-left px-3 py-2.5 font-semibold text-slate-500 text-xs uppercase tracking-wider border-b border-slate-200">{{ t('finance.transactions.vendor') }}</th>
                  <th class="text-left px-3 py-2.5 font-semibold text-slate-500 text-xs uppercase tracking-wider border-b border-slate-200">{{ t('finance.transactions.date') }}</th>
                  <th class="text-right px-3 py-2.5 font-semibold text-slate-500 text-xs uppercase tracking-wider border-b border-slate-200">{{ t('finance.transactions.amount') }}</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="transaction in recentTransactions"
                  :key="transaction.id"
                  class="cursor-pointer transition-colors hover:bg-slate-50/70 border-b border-slate-50"
                  @click="handleTransactionClick(transaction)"
                >
                  <td class="px-3 py-3 text-slate-500 font-medium font-mono text-xs">{{ transaction.id.toString().padStart(3, '0') }}</td>
                  <td class="px-3 py-3 text-sm text-slate-900 font-medium">{{ transaction.description }}</td>
                  <td class="px-3 py-3 text-sm text-slate-500">{{ transaction.vendor }}</td>
                  <td class="px-3 py-3 text-xs text-slate-500">{{ formatDateShort(transaction.date) }}</td>
                  <td class="px-3 py-3 text-sm font-bold text-slate-900 text-right">{{ currencySymbol }}{{ transaction.amount.toLocaleString() }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <CostDetailModal
      :is-open="showCostModal"
      :cost-data="selectedCostData"
      @close="showCostModal = false"
    />
  </div>
</template>

<script>
import { ref, onMounted, watch, computed } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'
import { formatCurrency as formatCurrencyUtil } from '../utils/currency'
import CostDetailModal from '../components/CostDetailModal.vue'

export default {
  name: 'Spending',
  components: {
    CostDetailModal
  },
  setup() {
    const { t, currentCurrency } = useI18n()
    const loading = ref(true)
    const error = ref(null)
    const allMonthlySpending = ref([])
    const allCategorySpending = ref([])
    const allTransactions = ref([])
    const summaryData = ref({})
    const allOrders = ref([])

    // Modal state
    const showCostModal = ref(false)
    const selectedCostData = ref(null)

    // Use shared filters
    const { selectedPeriod, getCurrentFilters } = useFilters()

    // Monthly spending chart always shows all months (not filtered)
    const monthlySpending = computed(() => {
      return allMonthlySpending.value
    })

    // Filtered monthly spending for summary calculations only
    const filteredMonthlySpending = computed(() => {
      if (selectedPeriod.value === 'all') {
        return allMonthlySpending.value
      }

      // Extract month name from YYYY-MM format
      const monthMap = {
        '01': 'Jan', '02': 'Feb', '03': 'Mar', '04': 'Apr',
        '05': 'May', '06': 'Jun', '07': 'Jul', '08': 'Aug',
        '09': 'Sep', '10': 'Oct', '11': 'Nov', '12': 'Dec'
      }
      const selectedMonth = monthMap[selectedPeriod.value.split('-')[1]]
      return allMonthlySpending.value.filter(m => m.month === selectedMonth)
    })

    const categorySpending = computed(() => {
      return allCategorySpending.value
    })

    const recentTransactions = computed(() => {
      if (selectedPeriod.value === 'all') {
        return allTransactions.value
      }
      // Filter transactions by selected month
      return allTransactions.value.filter(t => {
        const transactionMonth = new Date(t.date).toISOString().slice(0, 7)
        return transactionMonth === selectedPeriod.value
      })
    })

    const summary = computed(() => {
      // Recalculate summary based on filteredMonthlySpending (not the chart data)
      if (filteredMonthlySpending.value.length === 0) {
        return summaryData.value
      }

      const totals = filteredMonthlySpending.value.reduce((acc, month) => ({
        procurement: acc.procurement + month.procurement,
        operational: acc.operational + month.operational,
        labor: acc.labor + month.labor,
        overhead: acc.overhead + month.overhead
      }), { procurement: 0, operational: 0, labor: 0, overhead: 0 })

      return {
        total_procurement_cost: totals.procurement,
        total_operational_cost: totals.operational,
        total_labor_cost: totals.labor,
        total_overhead: totals.overhead,
        procurement_change: summaryData.value.procurement_change || 0,
        operational_change: summaryData.value.operational_change || 0,
        labor_change: summaryData.value.labor_change || 0,
        overhead_change: summaryData.value.overhead_change || 0
      }
    })

    // Filtered orders based on selected period
    const filteredOrders = computed(() => {
      if (selectedPeriod.value === 'all') {
        return allOrders.value
      }

      // Filter orders by selected month
      return allOrders.value.filter(order => {
        const orderMonth = new Date(order.order_date).toISOString().slice(0, 7)
        return orderMonth === selectedPeriod.value
      })
    })

    // Revenue metrics from filtered orders
    const revenueMetrics = computed(() => {
      const totalRevenue = filteredOrders.value.reduce((sum, order) => sum + (order.total_value || 0), 0)
      const orderCount = filteredOrders.value.length
      const avgOrderValue = orderCount > 0 ? totalRevenue / orderCount : 0

      return {
        totalRevenue,
        orderCount,
        avgOrderValue,
        revenueGrowth: 15.3 // Placeholder - could calculate from historical data
      }
    })

    // Total costs from summary
    const totalCosts = computed(() => {
      return summary.value.total_procurement_cost +
             summary.value.total_operational_cost +
             summary.value.total_labor_cost +
             summary.value.total_overhead
    })

    // Net profit
    const netProfit = computed(() => {
      return revenueMetrics.value.totalRevenue - totalCosts.value
    })

    // Profit margin percentage
    const profitMargin = computed(() => {
      if (revenueMetrics.value.totalRevenue === 0) return 0
      return ((netProfit.value / revenueMetrics.value.totalRevenue) * 100).toFixed(1)
    })

    // Monthly revenue data for chart
    const monthlyRevenue = computed(() => {
      const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

      // Initialize all months
      const revenueByMonth = monthNames.map(month => ({
        month,
        revenue: 0,
        costs: 0
      }))

      // Calculate revenue from orders
      allOrders.value.forEach(order => {
        const orderDate = new Date(order.order_date)
        const monthIndex = orderDate.getMonth()
        if (monthIndex >= 0 && monthIndex < 12) {
          revenueByMonth[monthIndex].revenue += order.total_value || 0
        }
      })

      // Add costs from spending data
      allMonthlySpending.value.forEach(spending => {
        const monthIndex = monthNames.indexOf(spending.month)
        if (monthIndex >= 0) {
          revenueByMonth[monthIndex].costs = spending.procurement + spending.operational + spending.labor + spending.overhead
        }
      })

      return revenueByMonth
    })

    // Max value for chart scaling
    const maxRevenueValue = computed(() => {
      const maxRevenue = Math.max(...monthlyRevenue.value.map(m => m.revenue))
      const maxCost = Math.max(...monthlyRevenue.value.map(m => m.costs))
      const max = Math.max(maxRevenue, maxCost)
      return Math.ceil(max / 1000) // Return in K
    })

    const loadData = async () => {
      try {
        loading.value = true
        const [summaryRes, monthlyRes, categoryRes, transactionsRes, ordersRes] = await Promise.all([
          api.getSpendingSummary(),
          api.getMonthlySpending(),
          api.getCategorySpending(),
          api.getTransactions(),
          api.getOrders()
        ])

        summaryData.value = summaryRes
        allMonthlySpending.value = monthlyRes
        allCategorySpending.value = categoryRes
        allTransactions.value = transactionsRes
        allOrders.value = ordersRes
      } catch (err) {
        error.value = 'Failed to load financial data: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Watch for period filter changes
    watch([selectedPeriod], () => {
      // Data will automatically update via computed properties
    })

    const formatCurrency = (value) => {
      return formatCurrencyUtil(value, currentCurrency.value)
    }

    const currencySymbol = computed(() => {
      return currentCurrency.value === 'JPY' ? '¥' : '$'
    })

    const getBarHeight = (value) => {
      const maxValue = 25000
      return (value / maxValue) * 100
    }

    const getRevenueBarHeight = (value) => {
      const maxValue = maxRevenueValue.value * 1000
      return (value / maxValue) * 100
    }

    const formatDate = (dateString) => {
      return new Date(dateString).toLocaleDateString('en-US', {
        month: 'short',
        day: 'numeric'
      })
    }

    const formatDateShort = (dateString) => {
      const date = new Date(dateString)
      const month = (date.getMonth() + 1).toString().padStart(2, '0')
      const day = date.getDate().toString().padStart(2, '0')
      const year = date.getFullYear().toString().slice(-2)
      return `${month}/${day}/${year}`
    }

    const translateMonth = (month) => {
      const monthMap = {
        'Jan': t('months.jan'),
        'Feb': t('months.feb'),
        'Mar': t('months.mar'),
        'Apr': t('months.apr'),
        'May': t('months.may'),
        'Jun': t('months.jun'),
        'Jul': t('months.jul'),
        'Aug': t('months.aug'),
        'Sep': t('months.sep'),
        'Oct': t('months.oct'),
        'Nov': t('months.nov'),
        'Dec': t('months.dec')
      }
      return monthMap[month] || month
    }

    const translateCategory = (category) => {
      // First try spending categories
      const spendingCategoryMap = {
        'Raw Materials': t('spendingCategories.rawMaterials'),
        'Components': t('spendingCategories.components'),
        'Equipment': t('spendingCategories.equipment'),
        'Consumables': t('spendingCategories.consumables')
      }

      // Then try product categories
      const productCategoryMap = {
        'Circuit Boards': t('categories.circuitBoards'),
        'Sensors': t('categories.sensors'),
        'Actuators': t('categories.actuators'),
        'Controllers': t('categories.controllers'),
        'Power Supplies': t('categories.powerSupplies')
      }

      return spendingCategoryMap[category] || productCategoryMap[category] || category
    }

    const handleTransactionClick = (transaction) => {
      console.log('Transaction clicked:', transaction)
      alert(`Transaction Details:\n\nID: ${transaction.id}\nDescription: ${transaction.description}\nVendor: ${transaction.vendor}\nDate: ${formatDateShort(transaction.date)}\nAmount: $${transaction.amount.toLocaleString()}`)
    }

    const showCostDetail = (monthData) => {
      selectedCostData.value = monthData
      showCostModal.value = true
    }

    onMounted(loadData)

    return {
      t,
      loading,
      error,
      summary,
      monthlySpending,
      categorySpending,
      recentTransactions,
      revenueMetrics,
      totalCosts,
      netProfit,
      profitMargin,
      monthlyRevenue,
      maxRevenueValue,
      formatCurrency,
      currencySymbol,
      getBarHeight,
      getRevenueBarHeight,
      formatDate,
      formatDateShort,
      translateMonth,
      translateCategory,
      handleTransactionClick,
      showCostModal,
      selectedCostData,
      showCostDetail,
      Math
    }
  }
}
</script>

<style scoped>
/* Stacked bar segment colors — cannot use Tailwind bg-* on dynamic segments
   because the stacked bar uses flex-col-reverse which requires precise color assignment */
.bar-segment-procurement { background: #6366f1; }
.bar-segment-operational { background: #8b5cf6; }
.bar-segment-labor      { background: #10b981; }
.bar-segment-overhead   { background: #f59e0b; }
</style>
