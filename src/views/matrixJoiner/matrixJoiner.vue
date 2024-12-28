<template>
  <div class="min-h-screen bg-[#2a2a2c] text-white p-6">
    <nav class="flex items-center gap-2 text-gray-400 mb-6">
      <span>Home</span>
      <span class="text-gray-600">›</span>
      <span>My Strategies</span>
    </nav>

    <div class="mb-6">
      <div class="flex items-center justify-between mb-4">
        <div class="text-2xl ">My Strategies</div>
      </div>
      
      <!-- Search and Filter Bar -->
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4  bg-[#1d1d20] p-4 rounded-t-lg">
        <div class="relative w-full sm:w-64">
          <input 
            type="search" 
            placeholder="Search" 
            v-model="searchQuery"
            @input="handleSearch"
            class="w-full bg-[#1d1d20] border border-gray-700 rounded-lg pl-10 pr-4 py-2 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-gray-700"
          />
          <SearchIcon class="absolute left-3 top-2.5 h-5 w-5 text-gray-400" />
        </div>
        
        <div class="flex items-center gap-3">
          <span class="text-gray-400">Showing My Strategies type:</span>
          <div class="flex items-center gap-4">
            <label class="flex items-center gap-2 cursor-pointer">
              <input 
                type="checkbox" 
                v-model="filters.active"
                @change="handleFilterChange"
                class="form-checkbox bg-[#262626] border-gray-700 rounded text-[#7C3AED]" 
              />
              <span>Active</span>
            </label>
            <label class="flex items-center gap-2 cursor-pointer">
              <input 
                type="checkbox" 
                v-model="filters.inactive"
                @change="handleFilterChange"
                class="form-checkbox bg-[#262626] border-gray-700 rounded text-[#7C3AED]" 
              />
              <span>Inactive</span>
            </label>
          </div>
        </div>
      </div>

      <!-- Table -->
      <div class="overflow-x-auto bg-[#1d1d20] rounded-b-lg">
        <table class="w-full">
          <thead>
            <tr class="text-left border-b border-gray-700">
              <th class="py-3 px-4 text-gray-400 font-medium">Strategy</th>
              <th class="py-3 px-4 text-gray-400 font-medium">Broker</th>
              <th class="py-3 px-4 text-gray-400 font-medium">Lot size</th>
              <th class="py-3 px-4 text-gray-400 font-medium">ReEntry</th>
              <th class="py-3 px-4 text-gray-400 font-medium">ReEntry Triggered</th>
              <th class="py-3 px-4 text-gray-400 font-medium">Active</th>
              <th class="py-3 px-4 text-gray-400 font-medium">Joined at</th>
              <th class="py-3 px-4 text-gray-400 font-medium">Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="matrixJoiner in filteredMatrixJoiners" :key="matrixJoiner.id" class="border-b border-gray-800">
              <td class="py-4 px-4">
                <div class="flex items-center gap-2">
                  <component 
                    v-if="containsIcon(matrixJoiner)"
                    :is="matrixJoiner.strategy.image_url" 
                    class="h-5 w-5"
                    :style="{ color: matrixJoiner.strategy.color }"
                  />
                  <span>{{ `${matrixJoiner.strategy.name}(${matrixJoiner.strategy.script})` }}</span>
                </div>
              </td>
              <td class="py-4 px-4">
                <div class="flex items-center gap-2">
                  <img 
                    v-if="matrixJoiner.broker" 
                    :src="images[matrixJoiner.broker.broker_name]" 
                    class="h-5 w-5"
                  />
                  <span>{{ `${matrixJoiner.broker.broker_name} (${matrixJoiner.broker.broker_userid})` }}</span>
                </div>
              </td>
              <td class="py-4 px-4">{{ matrixJoiner.lots }}</td>
              <td class="py-4 px-4">{{ matrixJoiner.re_entry }}</td>
              <td class="py-4 px-4">{{ matrixJoiner.re_entry_triggered }}</td>
              <td class="py-4 px-4">
                <ButtonSwitch 
                  v-model="matrixJoiner.is_active"
                  @change="() => handleActiveChange(matrixJoiner)"
                />
              </td>
              <td class="py-4 px-4">{{ formatDate(matrixJoiner.created_at) }}</td>
              <td class="py-4 px-4">
                <div class="flex items-center gap-3">
                  <button 
                    @click="showEdit(matrixJoiner)"
                    class="flex items-center gap-1 text-gray-400 hover:text-white"
                  >
                    <PencilIcon class="h-4 w-4" />
                    <span>Edit</span>
                  </button>
                  <button 
                    @click="deleteModal(matrixJoiner.id)"
                    class="flex items-center gap-1 text-gray-400 hover:text-red-500"
                  >
                    <TrashIcon class="h-4 w-4" />
                    <span>Delete</span>
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Pagination -->
      <!-- Add pagination component here if needed -->

    </div>
  </div>
  <AddEditMatrixJoiner />
  <DeleteMatrixJoiner />
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { storeToRefs } from 'pinia'
import { useMatrixJoinersStore } from '@/stores/matrix/matrixJoiner'
import { SearchIcon, PencilIcon, TrashIcon } from 'lucide-vue-next'
import { images } from '@/assets/img'
import AddEditMatrixJoiner from './addEditMatrixJoiner.vue'
import DeleteMatrixJoiner from './deleteMatrixJoiner.vue'

const matrixJoinersStore = useMatrixJoinersStore()
const { showAddEditModal, addEditMatrixJoinerData, showDeleteConfirmationModal, idForDelete } = storeToRefs(matrixJoinersStore)
const { addEditMatrixJoiner } = matrixJoinersStore

const searchQuery = ref('')
const filters = ref({
  active: true,
  inactive: true
})

const matrixJoiners = computed(() => {
  return matrixJoinersStore.matrixJoiners.sort((a: any, b: any) => b.id - a.id)
})

const filteredMatrixJoiners = computed(() => {
  return matrixJoiners.value.filter((joiner: any) => {
    const matchesSearch = 
      joiner.strategy.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      joiner.strategy.script.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      joiner.broker.broker_name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      joiner.broker.broker_userid.toLowerCase().includes(searchQuery.value.toLowerCase())

    const matchesFilter = 
      (filters.value.active && joiner.is_active) ||
      (filters.value.inactive && !joiner.is_active)

    return matchesSearch && matchesFilter
  })
})

const handleSearch = () => {
  // The filtering is handled by the computed property
}

const handleFilterChange = () => {
  // If both checkboxes are unchecked, check both of them
  if (!filters.value.active && !filters.value.inactive) {
    filters.value.active = true
    filters.value.inactive = true
  }
}

const handleActiveChange = async (matrixJoiner: any) => {
  await addEditMatrixJoiner(matrixJoiner.id, {
    is_active: !matrixJoiner.is_active
  })
}

const containsIcon = (data: any) => {
  return data.strategy?.image_url ? true : false
}

const showEdit = (data: any) => {
  showAddEditModal.value = true
  addEditMatrixJoinerData.value = data
}

const deleteModal = (id: number) => {
  showDeleteConfirmationModal.value = true
  idForDelete.value = id
}

const formatDate = (date: string) => {
  if (!date) return ''
  return date.replace('T', ' ').replace('Z', '')
}
</script>

<style scoped>
:deep(.form-checkbox) {
  @apply rounded border-gray-700 text-[#7C3AED] focus:ring-[#7C3AED];
}
</style>

