<template>
  <EditModal/>
  <div class="w-full h-full md:h-full bg-[#2a2a2c]">
    <nav class="flex items-center gap-2 text-gray-400 p-6">
      <span>Home</span>
      <span class="text-gray-600">›</span>
      <span>Strategy</span>
    </nav>
    <h1 class="text-white text-2xl ml-8">Strategy</h1>
    <div class="flex flex-col rounded-lg ml-2 mr-2 sm:ml-4 sm:mr-4 mt-2">
      <!-- Filters Container -->
      <div class="flex justify-between items-start p-4">
        <!-- Strategy Type Filters -->
        <div class="flex items-center">
          <span class="text-gray-300 mr-4">Showing strategies type:</span>
          <div class="flex space-x-3">
            <label class="flex items-center space-x-2">
              <input type="checkbox" 
                     v-model="filters.all" 
                     @change="handleAllChange"
                     class="form-checkbox rounded border-gray-600 bg-transparent" />
              <span class="text-gray-300 text-sm">All</span>
            </label>
            <label class="flex items-center space-x-2">
              <input type="checkbox" 
                     v-model="filters.myStrategies" 
                     @change="handleStrategyTypeChange"
                     class="form-checkbox rounded border-gray-600 bg-transparent" />
              <span class="text-gray-300 text-sm">My strategies</span>
            </label>
            <label class="flex items-center space-x-2">
              <input type="checkbox" 
                     v-model="filters.otherStrategies" 
                     @change="handleStrategyTypeChange"
                     class="form-checkbox rounded border-gray-600 bg-transparent" />
              <span class="text-gray-300 text-sm">Other strategies</span>
            </label>
          </div>
        </div>

        <!-- Symbol Filters -->
        <div class="flex items-center">
          <span class="text-gray-300 mr-4">Symbols:</span>
          <div class="flex space-x-3">
            <label class="flex items-center space-x-2">
              <input type="checkbox" 
                     v-model="filters.symbols.ALL" 
                     @change="handleSymbolAllChange"
                     class="form-checkbox rounded border-gray-600 bg-transparent" />
              <span class="text-gray-300 text-sm">All</span>
            </label>
            <label v-for="symbol in availableSymbols" 
                   :key="symbol"
                   class="flex items-center space-x-2">
              <input type="checkbox" 
                     v-model="filters.symbols[symbol]" 
                     @change="handleSymbolChange"
                     class="form-checkbox rounded border-gray-600 bg-transparent" />
              <span class="text-gray-300 text-sm">{{ symbol }}</span>
            </label>
          </div>
        </div>
      </div>

      <!-- Strategy Count -->
      <div class="px-4 text-gray-400 text-sm">
        {{ filteredStrategies.length }} Strategies
      </div>

      <!-- Strategy Cards Grid -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 p-4">
        <div v-for="strategy in filteredStrategies" 
             :key="strategy.id" 
             class="bg-[#1d1d20] rounded-lg p-4">
             
          <!-- Tags -->
          <div class="flex gap-2 mb-4">
            <span :class="[
              'px-3 py-1 text-xs rounded-xl font-medium',
              getSymbolClass(strategy.script)
            ]">
              {{ strategy.script }}
            </span>
            <span :class="[
              'px-3 py-1 text-xs rounded-xl font-medium',
              strategy.risk === 'Low risk' ? 'bg-[#2E7D32] text-white' : 'bg-[#D32F2F] text-white'
            ]">
              {{ strategy.risk }}
            </span>
          </div>

          <!-- Strategy Name and Toggle -->
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-xl font-medium text-white">{{ strategy.name }}</h3>
            <div class="flex items-center">
              <button v-if="strategy.is_deployed" @click="showEdit(strategy)">
                <img src="/public/edit.svg" alt="Edit">
              </button>
            </div>
          </div>

          <!-- Strategy Details -->
          <div class="grid grid-cols-2 gap-4 mb-6 bg-[#1d1d20] border border-gray-600 rounded-lg p-8">
            <div class="flex flex-col items-center border-r border-gray-600 pr-4">
              <div class="w-12 h-12 p-2 mb-2 border border-gray-600 rounded-xl flex items-center justify-center">
                <Monitor class="w-6 h-6 text-gray-400" />
              </div>
              <span class="text-sm text-gray-400">Option</span>
              <span :class="[ 
                'text-base font-medium', 
                strategy.option_type === 'Buy' ? 'text-green-500' : 'text-red-500' 
              ]">{{ strategy.option_type }}</span>
            </div>
            <div class="flex flex-col items-center">
              <div class="w-12 h-12 p-2 mb-2 border border-gray-600 rounded-xl flex items-center justify-center">
                <IndianRupee class="w-6 h-6 text-gray-400" />
              </div>
              <span class="text-sm text-gray-400">Capital required</span>
              <span class="text-base font-medium text-white">{{ strategy.capital_required }}K</span>
            </div>
          </div>

          <!-- Action Buttons -->
          <div class="flex gap-2">
            <button 
              v-if="!strategy.is_deployed"
              @click="deployStrategy(strategy)"
              class="w-full py-2.5 rounded-md text-sm font-medium bg-[#4A4AFF] hover:bg-[#3A3AFF] text-white transition-colors"
            >
              Deploy
            </button>
            <template v-else>
              <button 
                @click="removeStrategy(strategy)"
                class="w-full py-2.5 rounded-md text-sm font-medium border border-[#4A4AFF] text-[#4A4AFF] hover:bg-[#4A4AFF] hover:text-white transition-colors"
              >
                Remove
              </button>
              <button 
                disabled
                class="w-full py-2.5 rounded-md text-sm font-medium bg-gray-700 text-gray-400 cursor-not-allowed"
              >
                Deployed
              </button>
            </template>
          </div>
        </div>
      </div>
    </div>
  </div>

  <AddMatrixJoiner />
  <Message />
</template>

<script setup lang="ts">
import { ref, computed } from "vue"
import { storeToRefs } from "pinia"
import { useStrategiesStore } from '@/stores/matrix/strategy'
import { useProfileStore } from "@/stores/matrix/profile"
import { Monitor, IndianRupee } from 'lucide-vue-next'
import AddMatrixJoiner from ".././matrixJoiner/addEditMatrixJoiner.vue"
import Message from './message.vue'
import { makeRequest } from "@/request/request"

// Store initialization
const strategiesStore = useStrategiesStore()
const profileStore = useProfileStore()
const { strategies, plans, stratgyJoinedPlans } = storeToRefs(strategiesStore)

// Available trading symbols (without ALL)
const availableSymbols = ['NIFTY', 'BANKNIFTY', 'STOCKS', 'SENSEX']

import { useMatrixJoinersStore } from '@/stores/matrix/matrixJoiner'
import EditModal from "./EditModal.vue"

const matrixJoinersStore = useMatrixJoinersStore()
const { showAddEditModal, addEditMatrixJoinerData, showDeleteConfirmationModal, idForDelete } = storeToRefs(matrixJoinersStore)

const showEdit = (data: any) => {
  showAddEditModal.value = true
  addEditMatrixJoinerData.value = data
}

// Filter state with ALL symbol enabled by default
const filters = ref({
  all: true,
  myStrategies: false,
  otherStrategies: false,
  symbols: {
    ALL: true,
    ...availableSymbols.reduce((acc, symbol) => {
      acc[symbol] = false
      return acc
    }, {} as Record<string, boolean>)
  }
})

// Handle "All Strategies" checkbox change
const handleAllChange = () => {
  if (filters.value.all) {
    filters.value.myStrategies = false
    filters.value.otherStrategies = false
  }
}

// Handle strategy type checkbox changes
const handleStrategyTypeChange = () => {
  if (filters.value.myStrategies || filters.value.otherStrategies) {
    filters.value.all = false
  }
  
  if (!filters.value.myStrategies && !filters.value.otherStrategies) {
    filters.value.all = true
  }
}

// Handle "All Symbols" checkbox change
const handleSymbolAllChange = () => {
  if (filters.value.symbols.ALL) {
    // If ALL is checked, uncheck all other symbols
    availableSymbols.forEach(symbol => {
      filters.value.symbols[symbol] = false
    })
  }
}

// Handle individual symbol checkbox changes
const handleSymbolChange = () => {
  // If any individual symbol is checked, uncheck ALL
  const hasIndividualSelection = availableSymbols.some(symbol => filters.value.symbols[symbol])
  if (hasIndividualSelection) {
    filters.value.symbols.ALL = false
  }
  
  // If no individual symbols are checked, check ALL
  const noSymbolsSelected = availableSymbols.every(symbol => !filters.value.symbols[symbol])
  if (noSymbolsSelected) {
    filters.value.symbols.ALL = true
  }
}

// Get user ID for filtering
const userId = computed(() => profileStore.profile?.id)

// Merged strategies with joined plan details
const mergedStrategies = computed(() => {
  return strategies.value.map(strategy => {
    const joinedStrategy = stratgyJoinedPlans.value.find(
      (joined: any) => joined.strategy_id === strategy.id
    )
    if (joinedStrategy) {
      return {
        ...strategy,
        lots: joinedStrategy.lots,
        re_entry: joinedStrategy.re_entry,
        is_deployed: true,
        is_active: joinedStrategy.is_active
      }
    }
    return {
      ...strategy,
      is_deployed: false,
      is_active: false
    }
  })
})

// Filtered strategies based on all filters
const filteredStrategies = computed(() => {
  let filtered = mergedStrategies.value

  // Apply strategy type filters
  if (!filters.value.all) {
    if (filters.value.myStrategies) {
      filtered = filtered.filter(strategy => isDeplo)
    }
    if (filters.value.otherStrategies) {
      filtered = filtered.filter(strategy => strategy.user_id !== userId.value)
    }
    if (!filters.value.myStrategies && !filters.value.otherStrategies) {
      filtered = []
    }
  }

  // Apply symbol filters
  if (!filters.value.symbols.ALL) {
    filtered = filtered.filter(strategy => 
      Object.entries(filters.value.symbols)
        .some(([symbol, isSelected]) => 
          symbol !== 'ALL' && isSelected && strategy.script === symbol
        )
    )
  }

  return filtered
})

// Get symbol class for styling
const getSymbolClass = (symbol: string) => {
  const symbolClasses: Record<string, string> = {
    'NIFTY': 'bg-[#8B5E34] text-white',
    'BANKNIFTY': 'bg-[#4A4AFF] text-white',
    'STOCKS': 'bg-[#2E7D32] text-white',
    'SENSEX': 'bg-[#D32F2F] text-white'
  }
  return symbolClasses[symbol] || 'bg-gray-600 text-white'
}

// Handle strategy deployment
const deployStrategy = async (strategy: any) => {
  try {
    await strategiesStore.addEditMatrixJoiner(0, {
      strategy_id: strategy.id,
      is_deployed: true,
      is_active: true,
      lots: strategy.lots || 1,  // Default to 1 if not specified
      re_entry: strategy.re_entry || false  // Default to false if not specified
    })
    await strategiesStore.getStrategies()
  } catch (error) {
    console.error('Failed to deploy strategy:', error)
  }
}

// Handle strategy removal
const removeStrategy = async (strategy: any) => {
  try {
    const joinedStrategy = stratgyJoinedPlans.value.find(
      (joined: any) => joined.strategy_id === strategy.id
    )
    if (joinedStrategy) {
      await strategiesStore.deleteMatrixJoiner(joinedStrategy.id)
      await strategiesStore.getStrategies()
    }
  } catch (error) {
    console.error('Failed to remove strategy:', error)
  }
}

// Handle strategy activation/deactivation
const handleStrategyToggle = async (strategy: any) => {
  try {
    const res = await makeRequest('plan_strategy', 'PUT', {is_active: !strategy.is_active}, {}, {}, 0, strategy.id)
    console.log(res)
    strategy.is_active = !strategy.is_active
  } catch (error) {
    console.error('Failed to update strategy:', error)
    strategy.is_active = !strategy.is_active
  }
}

// Initialize data
strategiesStore.getStrategies()
</script>

<style scoped>
.mobile-device-table {
  @apply h-[calc(100vh-194px)] w-full overflow-scroll;
}
</style>