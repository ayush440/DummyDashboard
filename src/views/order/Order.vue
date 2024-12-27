<template>
  <PlaceOrder :isOpen="isOpen" :toggleModal="toggleModal"/>
  <div class="min-h-screen bg-[#2a2a2c]">
    <!-- Breadcrumb -->
    <div class="flex items-center gap-2 text-gray-400 px-6 py-3 mt-3">
      <span>Home</span>
      <span>›</span>
      <span>Order</span>
    </div>

    <!-- Header -->
    <div class="flex justify-between items-center px-6 py-4">
      <h1 class="text-2xl text-white">Order</h1>
      <button  @click="toggleModal" class="bg-[#8b7eff] text-white px-4 py-1 rounded-lg">
        Place order
      </button>
    </div>

    <!-- Main Content -->
    <div class="px-6">
      <!-- Search and Filters -->
      <div class="flex justify-between items-center rounded-t-lg mx-auto pr-6 py-4 bg-[#1d1d20]">
        <div class="relative w-64">
          <input 
            v-model="searchQuery"
            type="search" 
            placeholder="Search" 
            class="w-full bg-[#1d1d20] text-white border ml-5 border-gray-600 rounded-lg pl-10 pr-4 py-2 focus:outline-none"
          />
          <Search class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 ml-5 text-gray-400" />
        </div>
        
        <div class="flex items-center gap-4">
          <span class="text-gray-400">Showing order type:</span>
          <div class="flex gap-3">
            <label class="flex items-center gap-2 cursor-pointer">
              <input v-model="filters.successful" type="checkbox" class="form-checkbox rounded-lg bg-[#2C2C2E] border-gray-600 text-[#7C5CFC]" />
              <span class="text-white">Successful</span>
            </label>
            <label class="flex items-center gap-2 cursor-pointer">
              <input v-model="filters.pending" type="checkbox" class="form-checkbox rounded-lg bg-[#2C2C2E] border-gray-600 text-[#7C5CFC]" />
              <span class="text-white">Pending</span>
            </label>
            <label class="flex items-center gap-2 cursor-pointer">
              <input v-model="filters.rejected" type="checkbox" class="form-checkbox rounded-lg bg-[#2C2C2E] border-gray-600 text-[#7C5CFC]" />
              <span class="text-white">Rejected</span>
            </label>
          </div>
        </div>
      </div>

      <!-- Table -->
      <div class="bg-[#2C2C2E] rounded-b-lg  overflow-x-auto">
        <table class="w-full">
          <thead>
            <tr class="text-gray-400 bg-[#262626]">
              <th class="text-left p-4 font-normal">Strategy</th>
              <th class="text-left p-4 font-normal">Broker</th>
              <th class="text-left p-4 font-normal">Time</th>
              <th class="text-left p-4 font-normal">Script</th>
              <th class="text-left p-4 font-normal">Side</th>
              <th class="text-left p-4 font-normal">QTY</th>
              <th class="text-left p-4 font-normal">Price</th>
              <th class="text-left p-4 font-normal">Status</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-800">
            <template v-if="filteredOrders.length > 0">
              <tr 
                v-for="(order, i) in filteredOrders" 
                :key="order.serialNo" 
                class="text-white bg-[#1d1d20]"
              >
                <td class="p-4">{{ order.strategy?.name || '--' }}</td>
                <td class="p-4">{{ order.broker?.broker_name || '--' }}</td>
                <td class="p-4">{{ formatTime(order.created_at) || '--' }}</td>
                <td class="p-4">{{ order.tradingsymbol || '--' }}</td>
                <td class="p-4">{{ order.transaction_type || '--' }}</td>
                <td class="p-4">{{ order.quantity || '0' }}/25</td>
                <td class="p-4">₹{{ order.status === 'COMPLETE' && (order.order_type === 'MARKET' || order.order_type === 'MKT') ? 
                  order.average_price : order.price }}</td>
                <td class="p-4">
                  <span :class="{
                    'px-3 py-1 rounded-full text-sm': true,
                    'bg-green-500/20 text-green-500': order.status === 'COMPLETE',
                    'bg-[#92400E]/20 text-[#F59E0B]': order.status === 'PENDING',
                    'bg-red-500/20 text-red-500': order.status === 'REJECTED'
                  }">
                    {{ order.status || '--' }}
                  </span>
                </td>
              </tr>
            </template>
            
            <tr v-else>
              <td colspan="8" class="p-4 text-center text-gray-400">
                No Orders Found
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { Search } from 'lucide-vue-next'
import { ref, computed } from 'vue'
import { useOrdersStore } from '@/stores/matrix/order'
import { useManualOrdersStore } from '@/stores/groups/manualOrders'
import PlaceOrder from '@/components/PlaceOrder.vue'

// Store initialization
const ordersStore = useOrdersStore()
const manualOrdersStore = useManualOrdersStore()
const isOpen = ref(false)
const toggleModal = () => isOpen.value = !isOpen.value
// Search and filter state
const searchQuery = ref('')
const filters = ref({
  successful: true,
  pending: true,
  rejected: true
})

// Computed properties
const orders = computed(() => {
  return ordersStore.orders.map((order, index) => ({
    ...order,
    serialNo: `order-${index}`
  }))
})

const manualOrders = computed(() => {
  return manualOrdersStore.manualOrders.map((manualOrder, index) => ({
    ...manualOrder,
    serialNo: `manualOrder-${index}`
  }))
})

const mergedOrders = computed(() => {
  const allOrders = [...orders.value, ...manualOrders.value]
  if (!allOrders.length) {
    return []
  }
  return allOrders.sort((a, b) => {
    const dateA = new Date(a.updated_at).getTime()
    const dateB = new Date(b.updated_at).getTime()
    return dateB - dateA
  })
})

const filteredOrders = computed(() => {
  return mergedOrders.value.filter(order => {
    const matchesSearch = !searchQuery.value || 
      order.tradingsymbol?.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      order.broker?.broker_name?.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      order.strategy?.name?.toLowerCase().includes(searchQuery.value.toLowerCase())
    
    const matchesFilter = 
      (order.status === 'COMPLETE' && filters.value.successful) ||
      (order.status === 'PENDING' && filters.value.pending) ||
      (order.status === 'REJECTED' && filters.value.rejected)
    
    return matchesSearch && matchesFilter
  })
})

// Utility functions
const formatTime = (dateString: string) => {
  if (!dateString) return '--'
  const date = new Date(dateString)
  return date.toLocaleTimeString('en-US', {
    hour12: false,
    hour: '2-digit',
    minute: '2-digit'
  })
}
</script>

<style scoped>
.overflow-x-auto {
  overflow-x: auto;
}

::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: #1C1C1E;
}

::-webkit-scrollbar-thumb {
  background: #4A4A4A;
  border-radius: 4px;
}
</style>