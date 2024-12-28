<template>
  <div class="w-full bg-[#1DID20] p-6 rounded-lg">
    <div class="flex justify-between items-center p-4 rounded-t-lg bg-[#1c1c1f] mb-1">
      <div class="flex items-center gap-4">
        <!-- Search with button -->
        <div class="relative flex items-center">
          <input
            type="text"
            placeholder="Search strategy"
            v-model="searchQuery"
            @input="handleSearch"
            class="w-64 bg-[#1c1c1f] text-white pl-10 pr-4 py-2 rounded-md border border-gray-700 focus:outline-none focus:border-[#7C3AED]"
          />
          <SearchIcon class="absolute left-3 h-5 w-5 text-gray-400" />
        </div>

        <!-- Strategy Dropdown -->
        <div class="relative">
          <button 
            @click="toggleStrategyDropdown"
            class="flex items-center gap-2 px-4 py-2 bg-[#1c1c1f] text-white border border-gray-700 rounded-md"
          >
            {{ selectedStrategy || 'Strategies' }}
            <ChevronDownIcon class="h-4 w-4" :class="{ 'rotate-180': showStrategyDropdown }" />
          </button>
          <div v-if="showStrategyDropdown" class="absolute z-10 mt-2 w-56 bg-[#1c1c1f] border border-gray-700 rounded-md shadow-lg">
            <div class="py-1">
              <button
                v-for="strategy in uniqueStrategies"
                :key="strategy"
                @click="selectStrategy(strategy)"
                class="block w-full px-4 py-2 text-left text-white hover:bg-[#2a2a2c]"
              >
                {{ strategy }}
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Status Filters -->
      <div class="flex items-center gap-6">
        <span class="text-gray-400">Showing position type:</span>
        <div class="flex gap-4">
          <label class="flex items-center gap-2 cursor-pointer">
            <input 
              type="checkbox" 
              v-model="filters.all" 
              @change="handleFilterChange('all')"
              class="form-checkbox" 
            />
            <span class="text-gray-400">All</span>
          </label>
          <label class="flex items-center gap-2 cursor-pointer">
            <input 
              type="checkbox" 
              v-model="filters.open"
              @change="handleFilterChange('open')"
              class="form-checkbox" 
            />
            <span class="text-gray-400">Open</span>
          </label>
          <label class="flex items-center gap-2 cursor-pointer">
            <input 
              type="checkbox" 
              v-model="filters.closed"
              @change="handleFilterChange('closed')"
              class="form-checkbox" 
            />
            <span class="text-gray-400">Closed</span>
          </label>
        </div>
      </div>
    </div>

    <!-- Desktop View -->
    <div class="hidden sm:block space-y-2">
      <!-- Header row -->
      <div class="flex justify-between items-center px-4 py-3 font-medium text-gray-700">
        <div class="w-[20%]">STRATEGY NAME</div>
        <div class="w-[10%]">QUANTITY</div>
        <div class="w-[10%]">SIDE</div>
        <div class="w-[10%]">TIME</div>
        <div class="w-[10%]">P&L</div>
        <div class="w-[10%]">STATUS</div>
        <div class="w-[10%]"></div>
        <div class="w-[2%]"></div>
      </div>

      <!-- Strategy groups -->
      <template v-if="filteredPositions.length > 0">
        <div v-for="(group, strategyId) in groupedFilteredPositions" :key="strategyId" class="mb-4">
          <div class="flex justify-between items-center px-4 py-2 bg-[#262626] rounded-t-lg text-white">
            <h3 class="font-semibold text-lg">{{ group.name }}</h3>
            <div :class="getPnLColor(group.pnl)" class="font-bold">
              {{ formatPnL(group.pnl) }}
            </div>
          </div>
          <div v-for="position in group.positions" :key="position.id" class="relative">
            <div
              class="cursor-pointer transition-colors duration-150 ease-in-out overflow-hidden text-white bg-[#1d1d20]"
              :class="{ 'rounded-b-none': expandedRows[position.id] }"
              @click="toggleExpand(position.id)"
            >
              <div class="flex justify-between px-4 py-3 text-white">
                <div class="w-[20%]">{{ group.name }}</div>
                <div class="w-[10%]">{{ position.quantity }}</div>
                <div class="w-[10%]">{{ position.side }}</div>
                <div class="w-[10%]">{{ new Date(position.updated_at).toLocaleTimeString() }}</div>
                <div class="w-[10%]" :class="getPnLColor(position.pnl)">{{ formatPnL(position.pnl) }}</div>
                <div class="w-[10%]">
                  <span 
                    class="px-3 py-1 rounded-full text-xs font-medium text-center"
                    :class="position.status === 'OPEN' ? 'bg-green-400/20 text-green-400' : 'bg-red-400/20 text-red-400'"
                  >
                    {{ position.status.toLowerCase() }}
                  </span>
                </div>
                <div class="w-[10%]">
                  <button
                    v-if="position.status === 'OPEN'"
                    @click.stop="openSquareOffModal(position)"
                    class="inline-flex items-center gap-2 px-3  rounded-full border border-[#7C3AED] bg-[#1d1d20]/10 text-[#7C3AED] hover:bg-[#7C3AED]/20 transition-colors"
                  >
                    Square Off
                   <img src="/public/off.png" alt="">
                  </button>
                  <span v-else class="text-gray-500 text-sm">
                    Closed at {{ formatTime(position.updated_at) }}
                  </span>
                </div>
                <div class="text-right w-[2%]">
                  <ChevronDownIcon v-if="expandedRows[position.id]" class="w-5 h-5 text-white inline-block" />
                  <ChevronRightIcon v-else class="w-5 h-5 text-white inline-block" />
                </div>
              </div>
            </div>
            <div v-if="expandedRows[position.id]" class="bg-[#262626] rounded-b-lg">
              <div class="flex justify-between py-6 px-4">
                <div class="truncate-text w-[20%]">
                  <p class="text-sm font-medium text-gray-500">Strategy</p>
                  <p class="mt-1 text-sm text-white">{{ position.tradingsymbol }}</p>
                </div>
                <div class="w-[10%]">
                  <p class="text-sm font-medium text-gray-500">Broker</p>
                  <p class="mt-1 text-sm text-white">{{ position.broker?.broker_name }}</p>
                </div>
                <div class="w-[10%]">
                  <p class="text-sm font-medium text-gray-500">Scripts</p>
                  <p class="mt-1 text-sm text-white">{{ group.script }}</p>
                </div>
                <div class="w-[10%]">
                  <p class="text-sm font-medium text-gray-500">Buy Price</p>
                  <p class="mt-1 text-sm text-white">{{ position.buy_price }}</p>
                </div>
                <div class="w-[10%]">
                  <p class="text-sm font-medium text-gray-500">Sell Price</p>
                  <p class="mt-1 text-sm text-white">{{ position.sell_price }}</p>
                </div>
                <div class="w-[10%]">
                  <p class="text-sm font-medium text-gray-500">P&L</p>
                  <p :class="['mt-1 text-sm text-white', getPnLColor(position.pnl)]">{{ formatPnL(position.pnl) }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </template>
      <div v-else-if="!showTableData" class="flex justify-center py-4">
        <LoadingIcon icon="puff" class="w-8 h-8" />
      </div>
      <div v-else class="text-center py-4">
        Data not found!!
      </div>
    </div>

    <!-- Mobile View -->
    <div class="sm:hidden space-y-4">
      <div class="bg-third text-tabletext dark:bg-primary py-3 px-4 rounded-lg">
        <div class="grid grid-cols-5 font-bold text-xs">
          <div>Strategy</div>
          <div>Qty</div>
          <div>Side</div>
          <div>P&L</div>
          <div>Type</div>
        </div>
      </div>

      <template v-if="filteredPositions.length > 0">
        <div v-for="(group, strategyId) in groupedFilteredPositions" :key="strategyId" class="mb-4">
          <div v-for="position in group.positions" :key="position.id" 
               class="bg-[#1d1d20] p-4 rounded-lg space-y-2">
            <div class="grid grid-cols-5 items-center text-xs">
              <div class="truncate">{{ position.tradingsymbol }}</div>
              <div>{{ position.quantity }}</div>
              <div>{{ position.side }}</div>
              <div :class="getPnLColor(position.pnl)">{{ formatPnL(position.pnl) }}</div>
              <div>{{ position.type }}</div>
            </div>
            <div @click="toggleExpand(position.id)" class="cursor-pointer text-sm text-gray-600 flex items-center justify-between mt-2">
              <span>Details</span>
              <ChevronDownIcon v-if="expandedRows[position.id]" class="w-4 h-4" />
              <ChevronRightIcon v-else class="w-4 h-4" />
            </div>
            <div v-if="expandedRows[position.id]" class="text-xs space-y-2 mt-2 bg-[#262626] p-4 rounded-lg">
              <div class="grid grid-cols-2 gap-2">
                <div>
                  <p class="font-medium text-gray-500">Broker</p>
                  <p class="text-white">{{ position.broker?.broker_name }}</p>
                </div>
                <div>
                  <p class="font-medium text-gray-500">Scripts</p>
                  <p class="text-white">{{ group.script }}</p>
                </div>
                <div>
                  <p class="font-medium text-gray-500">Buy Price</p>
                  <p class="text-white">{{ position.buy_price }}</p>
                </div>
                <div>
                  <p class="font-medium text-gray-500">Sell Price</p>
                  <p class="text-white">{{ position.sell_price }}</p>
                </div>
                <div>
                  <p class="font-medium text-gray-500">Time</p>
                  <p class="text-white">{{ new Date(position.updated_at).toLocaleTimeString() }}</p>
                </div>
              </div>
            </div>
            <button 
              @click.stop="squareOff(position)" 
              class="w-full bg-[#1d1d20] text-indigo-600 px-4 py-2 rounded-lg text-xs border border-indigo-200 mt-2 flex items-center justify-center space-x-1"
            >
              <svg width="12" height="12" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg" class="mr-1">
                <path d="M13.3334 4L6.00008 11.3333L2.66675 8" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              <span>Square off</span>
            </button>
          </div>
        </div>
      </template>
      <div v-else-if="!showTableData" class="flex justify-center py-4">
        <LoadingIcon icon="puff" class="w-8 h-8" />
      </div>
      <div v-else class="text-center py-4 text-sm">
        Data not found!!
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import { storeToRefs } from 'pinia'
import { ChevronRightIcon, ChevronDownIcon, RefreshCwIcon, SearchIcon } from 'lucide-vue-next'
import { usePositionsStore } from '@/stores/matrix/position'
import { useProfileStore } from '@/stores/matrix/profile'
import { usePaperPositionsStore } from "@/stores/matrix/paperPositions"
import { useManualPositionsStore } from'@/stores/groups/manualPosition'
import { useStrategiesStore } from '@/stores/matrix/strategy'

const props = defineProps<{
  positions: any[]
  mode: 'paper' | 'live' | 'both'
}>()

const paperPositionsStore = usePaperPositionsStore()
const profileStore = useProfileStore()
const positionsStore = usePositionsStore()
const manualPositionsStore = useManualPositionsStore()
const strategiesStore = useStrategiesStore()

const { profile } = storeToRefs(profileStore)
const { positions, strategiesPositions } = storeToRefs(positionsStore)
const { paperPositions } = storeToRefs(paperPositionsStore)
const { showSqOffModal, dataForSqOff, showCloseModal } = storeToRefs(positionsStore)
const { showManualSqOffModal, manualDataForSqOff, showManualCloseModal } = storeToRefs(manualPositionsStore)
const { strategies } = storeToRefs(strategiesStore)

const expandedRows = ref<any>({})
const searchQuery = ref('')
const showStrategyDropdown = ref(false)
const selectedStrategy = ref('')
const filters = ref({
  all: true,
  open: false,
  closed: false
})

const uniqueStrategies = computed(() => {
  const strategies = new Set(props.positions.map(p => p.strategy?.name || 'Multidisciplinary'))
  return Array.from(strategies)
})

const filteredPositions = computed(() => {
  let filtered = [...props.positions]

  if (selectedStrategy.value) {
    filtered = filtered.filter(p => 
      (p.strategy?.name || 'Multidisciplinary') === selectedStrategy.value
    )
  }

  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    filtered = filtered.filter(p => 
      (p.strategy?.name || '').toLowerCase().includes(query) ||
      (p.broker?.broker_name || '').toLowerCase().includes(query) ||
      (p.tradingsymbol || '').toLowerCase().includes(query)
    )
  }

  if (!filters.value.all) {
    if (filters.value.open) filtered = filtered.filter(p => p.status === 'OPEN')
    if (filters.value.closed) filtered = filtered.filter(p => p.status === 'CLOSED')
  }

  return filtered
})

const groupedFilteredPositions = computed(() => {
  return groupPositionsByStrategyId(filteredPositions.value)
})

const showTableData = computed<boolean>(() => {
  const state = positionsStore.state[positionsStore.endpoint]
  return state && state.loading === false
})

function handleFilterChange(type: 'all' | 'open' | 'closed') {
  if (type === 'all') {
    filters.value.open = false
    filters.value.closed = false
  } else {
    filters.value.all = false
  }
}

function selectStrategy(strategy: string) {
  selectedStrategy.value = strategy
  showStrategyDropdown.value = false
}

function handleSearch() {
  // The filtering is handled by the computed property
}

function toggleStrategyDropdown() {
  showStrategyDropdown.value = !showStrategyDropdown.value
}

function toggleExpand(id: number) {
  expandedRows.value[id] = !expandedRows.value[id]
}

function formatPnL(pnl: number) {
  if(!pnl) return '₹0.00'
  return pnl > 0 ? `+₹${pnl.toFixed(2)}` : `₹${pnl.toFixed(2)}`
}

function getPnLColor(pnl: number) {
  if (pnl > 0) return 'text-green-600'
  if (pnl < 0) return 'text-red-600'
  return ''
}

function squareOff(position: any) {
  if (!position || typeof position !== 'object') {
    console.error('Invalid position object:', position)
    return
  }

  if (position.id && position.strategy_id && position.broker_id) {
    showSqOffModal.value = true
    dataForSqOff.value = {
      position_id: position.id,
      strategy_id: position.strategy_id,
      broker_id: position.broker_id
    }
  } else if (position.id) {
    showManualSqOffModal.value = true
    manualDataForSqOff.value = {
      position_id: position.id
    }
  } else {
    console.warn('Unable to determine position type:', position)
  }
}

function openSquareOffModal(position: any) {
  showSqOffModal.value = true
  positionsStore.selectedPosition = position
}

function formatTime(time: string) {
  if (!time) return '09:20 AM'
  return new Date(time).toLocaleTimeString('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    hour12: true
  })
}

function groupPositionsByStrategyId(positions: any[]) {
  const groupedPositions: { [key: string]: any } = {}

  positions.forEach((position: any) => {
    const { strategy_id } = position
    const strategy = position.strategy || strategies.value.find((s: any) => s.id === strategy_id)

    if (!groupedPositions[strategy_id]) {
      groupedPositions[strategy_id] = {
        color: strategy?.color || '#000000',
        color2: strategy?.color2 || '#FFFFFF',
        pnl: 0,
        name: strategy?.name || 'Unknown Strategy',
        image_url: strategy?.image_url || '',
        script: strategy?.script || '',
        positions: [],
      }
    }

    groupedPositions[strategy_id].pnl += position.pnl
    groupedPositions[strategy_id].positions.push(position)
  })

  return groupedPositions
}

watch(() => props.positions, (newPositions) => {
  expandedRows.value = {}
}, { deep: true })

const totalPnL = computed(() => {
  return filteredPositions.value.reduce((sum: number, position: any) => sum + position.pnl, 0)
})

const positionCount = computed(() => {
  return filteredPositions.value.length
})

defineExpose({
  totalPnL,
  positionCount
})
</script>

<style scoped>
.truncate-text {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.hover\:shadow-glow:hover {
  box-shadow: 0 0 5px rgba(37, 99, 235, 1), 0 0 10px rgba(37, 99, 235, 1);
  border-color: rgba(37, 99, 235, 1);
}

.position-row {
  transition: background-color 0.2s ease;
}

.position-row:hover {
  background-color: rgba(243, 244, 246, 0.5);
}

.strategy-header {
  background-color: #f3f4f6;
  border-bottom: 1px solid #e5e7eb;
  padding: 8px 16px;
  font-weight: 600;
}

.expanded-details {
  background-color: #f9fafb;
  border-top: 1px solid #e5e7eb;
  padding: 12px;
}

.square-off-button {
  transition: all 0.2s ease;
}

.square-off-button:hover {
  background-color: #dbeafe;
  border-color: #3b82f6;
}

@media (max-width: 640px) {
  .strategy-header {
    padding: 6px 12px;
  }

  .expanded-details {
    padding: 8px;
  }
}
</style>