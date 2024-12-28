<template>
  <div class="min-h-screen bg-[#2a2a2c] p-4 sm:p-6">
    <!-- Breadcrumb -->
    <nav class="flex items-center gap-2 text-gray-400 mb-4 sm:mb-6 text-sm sm:text-base">
      <span>Home</span>
      <span class="text-gray-600">›</span>
      <span>Positions</span>
    </nav>

    <!-- Header Section -->
    <h1 class="text-white text-xl sm:text-2xl mb-4 sm:mb-6">Positions</h1>
    
    <!-- Search and Filters -->
    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center p-4 rounded-t-lg bg-[#1c1c1f] gap-4 sm:gap-0">
      <!-- Search -->
      <div class="relative w-full sm:w-64">
        <input
          type="text"
          placeholder="Search"
          v-model="searchQuery"
          class="w-full bg-[#1c1c1f] text-white pl-10 pr-4 py-2 rounded-md border border-gray-700 focus:outline-none focus:border-[#7C3AED]"
        />
        <SearchIcon class="absolute left-3 top-1/2 -translate-y-1/2 h-5 w-5 text-gray-400" />
      </div>

      <!-- Filters -->
      <div class="flex flex-col sm:flex-row items-start sm:items-center gap-2 sm:gap-6 w-full sm:w-auto">
        <span class="text-gray-400 text-sm">Showing position type:</span>
        <div class="flex flex-wrap gap-4">
          <label class="flex items-center gap-2 cursor-pointer">
            <input 
              type="checkbox" 
              v-model="filters.all" 
              @change="handleFilterChange('all')"
              class="form-checkbox" 
            />
            <span class="text-gray-400 text-sm">All</span>
          </label>
          <label class="flex items-center gap-2 cursor-pointer">
            <input 
              type="checkbox" 
              v-model="filters.open"
              @change="handleFilterChange('open')"
              class="form-checkbox" 
            />
            <span class="text-gray-400 text-sm">Open</span>
          </label>
          <label class="flex items-center gap-2 cursor-pointer">
            <input 
              type="checkbox" 
              v-model="filters.closed"
              @change="handleFilterChange('closed')"
              class="form-checkbox" 
            />
            <span class="text-gray-400 text-sm">Closed</span>
          </label>
        </div>
      </div>
    </div>

    <!-- Table -->
    <div class="bg-[#1d1d20] rounded-b-lg overflow-x-auto">
      <table class="w-full border-collapse">
        <thead class="hidden sm:table-header-group">
          <tr class="text-gray-400 text-sm border-b border-gray-700">
            <th class="py-4 px-6 text-left font-medium">Strategy</th>
            <th class="py-4 px-6 text-left font-medium">Side</th>
            <th class="py-4 px-6 text-center font-medium">QTY</th>
            <th class="py-4 px-6 text-right font-medium">PNL</th>
            <th class="py-4 px-6 text-center font-medium">Status</th>
            <th class="py-4 px-6 text-right font-medium">Action</th>
          </tr>
        </thead>
        <tbody>
          <template v-for="position in filteredPositions" :key="position.id">
            <!-- Mobile View -->
            <tr class="sm:hidden">
              <td colspan="6" class="p-0">
                <div class="p-4 border-b border-gray-700/50">
                  <div class="flex justify-between items-start mb-2">
                    <div>
                      <h3 class="text-white font-medium">{{ position.strategy?.name || 'Multidisciplinary' }}</h3>
                      <p class="text-gray-400 text-sm">{{ position.side }} • {{ position.quantity }}/25</p>
                    </div>
                    <div :class="calculatePnL(position) >= 0 ? 'text-green-400' : 'text-red-400'">
                      {{ calculatePnL(position) >= 0 ? '+' : '' }}{{ calculatePnL(position) }}
                    </div>
                  </div>
                  <div class="flex justify-between items-center mt-4">
                    <span 
                      class="px-3 py-1 rounded-full text-xs font-medium"
                      :class="position.status === 'OPEN' ? 'bg-green-400/20 text-green-400' : 'bg-red-400/20 text-red-400'"
                    >
                      {{ position.status.toLowerCase() }}
                    </span>
                    <button
                      v-if="position.status === 'OPEN'"
                      @click.stop="openSquareOffModal(position)"
                      class="inline-flex items-center gap-2 px-3 py-1 rounded-full border border-[#7C3AED] bg-[#1d1d20]/10 text-[#7C3AED] hover:bg-[#7C3AED]/20 transition-colors"
                    >
                      <span>Square Off</span>
                      <img src="/public/off.png" alt="Square Off" class="w-4 h-4">
                    </button>
                    <span v-else class="text-gray-500 text-sm">
                      Closed at {{ new Date(position.updated_at).toLocaleTimeString() }}
                    </span>
                  </div>
                </div>
              </td>
            </tr>
            <!-- Desktop View -->
            <tr class="hidden sm:table-row">
              <td colspan="6" class="p-0">
                <div 
                  class="grid grid-cols-6 cursor-pointer hover:bg-[#262626] transition-colors" 
                  :class="{'bg-[#262626]': expandedRow === position.id}"
                  @click="toggleExpand(position.id)"
                >
                  <div class="py-4 px-6 text-white">
                    {{ position.strategy?.name || 'Multidisciplinary' }}
                  </div>
                  <div class="py-4 px-6 text-white">
                    {{ position.side }}
                  </div>
                  <div class="py-4 px-6 text-center text-white">
                    {{ position.quantity }}/25
                  </div>
                  <div class="py-4 px-6 text-right" :class="calculatePnL(position) >= 0 ? 'text-green-400' : 'text-red-400'">
                    {{ calculatePnL(position) >= 0 ? '+' : '' }}{{ calculatePnL(position) }}
                  </div>
                  <div class="py-4 px-6 flex justify-center">
                    <span 
                      class="px-3 py-1 rounded-full text-xs font-medium inline-block min-w-[4rem] text-center"
                      :class="position.status === 'OPEN' ? 'bg-green-400/20 text-green-400' : 'bg-red-400/20 text-red-400'"
                    >
                      {{ position.status.toLowerCase() }}
                    </span>
                  </div>
                  <div class="py-4 px-6 text-right">
                    <button
                      v-if="position.status === 'OPEN'"
                      @click.stop="openSquareOffModal(position)"
                      class="inline-flex items-center gap-2 px-3 py-1 rounded-full border border-[#7C3AED] bg-[#1d1d20]/10 text-[#7C3AED] hover:bg-[#7C3AED]/20 transition-colors"
                    >
                      <span>Square Off</span>
                      <img src="/public/off.png" alt="Square Off" class="w-4 h-4">
                    </button>
                    <span v-else class="text-gray-500 text-sm">
                      Closed at {{ new Date(position.updated_at).toLocaleTimeString() }}
                    </span>
                  </div>
                </div>
              </td>
            </tr>
            <!-- Expanded Details (Desktop Only) -->
            <tr v-if="expandedRow === position.id" class="hidden sm:table-row bg-[#262626] border-t border-gray-700/50">
              <td colspan="6" class="py-4">
                <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-5 px-6 gap-4 sm:gap-6">
                  <div class="space-y-1">
                    <div class="text-gray-400 text-sm">Script</div>
                    <div class="text-white">{{ position.strategy?.script || 'Nifty36FDWTE72' }}</div>
                  </div>
                  <div class="space-y-1">
                    <div class="text-gray-400 text-sm">Broker</div>
                    <div class="text-white">{{ position.broker?.broker_name || 'DHAN2DS2E28G' }}</div>
                  </div>
                  <div class="space-y-1">
                    <div class="text-gray-400 text-sm">Buy price</div>
                    <div class="text-white">₹{{ position.buy_price }} ({{ formatTime(position.buy_time) }})</div>
                  </div>
                  <div class="space-y-1">
                    <div class="text-gray-400 text-sm">Sell price</div>
                    <div class="text-white">₹{{ position.sell_price }} ({{ formatTime(position.sell_time) }})</div>
                  </div>
                  <div class="space-y-1">
                    <div class="text-gray-400 text-sm">LTP</div>
                    <div class="text-white">{{ getLtp(position) }}</div>
                  </div>
                </div>
              </td>
            </tr>
          </template>
        </tbody>
      </table>
    </div>
  </div>

  <sqOffPosition />
  <sqoffManual />
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { storeToRefs } from 'pinia'
import { SearchIcon } from 'lucide-vue-next'
import { usePositionsStore } from '@/stores/matrix/position'
import { useManualPositionsStore } from '@/stores/groups/manualPosition'
import { useProfileStore } from '@/stores/matrix/profile'
import { usePaperPositionsStore } from "@/stores/matrix/paperPositions"
import { useTickerStore } from '@/stores/matrix/ticker/ticker'
import sqOffPosition from './sqOffPosition.vue'
import sqoffManual from './sqoffManual.vue'

// Store references
const profileStore = useProfileStore()
const positionsStore = usePositionsStore()
const manualPositionsStore = useManualPositionsStore()
const paperPositionsStore = usePaperPositionsStore()
const tickerStore = useTickerStore()

// State
const searchQuery = ref('')
const expandedRow = ref<number | null>(null)
const filters = ref({
  all: true,
  open: false,
  closed: false
})

// Store state
const { profile } = storeToRefs(profileStore)
const { positions } = storeToRefs(positionsStore)
const { manualPositions } = storeToRefs(manualPositionsStore)
const { paperPositions } = storeToRefs(paperPositionsStore)
const { showSqOffModal } = storeToRefs(positionsStore)

// Methods
function handleFilterChange(type: 'all' | 'open' | 'closed') {
  if (type === 'all') {
    filters.value.open = false
    filters.value.closed = false
  } else {
    filters.value.all = false
  }
}

function toggleExpand(positionId: number) {
  expandedRow.value = expandedRow.value === positionId ? null : positionId
}

function openSquareOffModal(position: any) {
  showSqOffModal.value = true
  positionsStore.selectedPosition = position
}

// Computed
const filteredPositions = computed(() => {
  let filtered = [...positions.value, ...manualPositions.value]
    .sort((a, b) => new Date(b.updated_at).getTime() - new Date(a.updated_at).getTime())

  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    filtered = filtered.filter(position => 
      position.strategy?.name?.toLowerCase().includes(query) ||
      position.broker?.broker_name?.toLowerCase().includes(query)
    )
  }

  if (!filters.value.all) {
    if (filters.value.open) filtered = filtered.filter(p => p.status === 'OPEN')
    if (filters.value.closed) filtered = filtered.filter(p => p.status === 'CLOSED')
  }

  return filtered
})

function getLtp(position: any) {
  const ltp = tickerStore.getLastPrice(position.instrument_token)
  return ltp || position.last_price
}

function calculatePnL(position: any) {
  const ltp = getLtp(position)
  let pnl = 0

  if (position.status === "CLOSED") {
    if (position.sell_price && position.buy_price) {
      pnl = position.sell_price * position.quantity - position.buy_price * position.quantity
    }
  } else if (position.status === "OPEN") {
    if (position.side === 'BUY' && position.buy_price && ltp) {
      pnl = (ltp - position.buy_price) * position.quantity
    } else if (position.side === 'SELL' && position.sell_price && ltp) {
      pnl = (position.sell_price - ltp) * position.quantity
    }
  }

  return pnl.toFixed(2)
}

function formatTime(time: string) {
  if (!time) return '09:20AM'
  return new Date(time).toLocaleTimeString('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    hour12: true
  })
}
</script>

<style scoped>
.form-checkbox {
  @apply rounded border-gray-600 text-[#7C3AED] focus:ring-[#7C3AED] focus:ring-offset-0 focus:ring-offset-transparent;
}
</style>