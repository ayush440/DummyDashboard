<template>
  <div :class="[isDashboard ? 'flex' : 'hidden md:flex']" class="border border-gray-600 bg-[#1D1D20] w-auto justify-between items-center flex-wrap space-y-1 pl-4 md:pl-10 pr-2 md:pr-4 py-3 md:py-4">
    <!-- Page Name -->
    <div class="unselectable capitalize font-bold text-lg sm:text-2xl text-white dark:text-tableText"> 
      <span v-if="!isDashboard">{{ title }}</span>
      <span v-if="isDashboard">Hello {{ profile?.name }} 👋</span>
    </div>

    <div class="flex items-center gap-4 pt-0 pr-6 2xl:pr-2">
      <!-- Add Broker Button -->
      <button 
        @click="showAddEditModal=true"
        class="flex items-center gap-2 bg-[#8B7EFF] hover:bg-[#7A6DFF] text-white px-4 py-2 rounded-lg transition-colors duration-200"
      >
        <PlusIcon class="w-4 h-4" />
        <span class="text-sm font-medium">Add broker</span>
      </button>

      <!-- Toggle Switch -->
      <div v-if="isDashboard">
        <div 
          class="relative inline-flex items-center rounded-full cursor-pointer w-[160px] h-[32px] bg-[#1D1D20] border border-gray-500" 
          @click="toggleMode"
        >
          <div 
            class="absolute w-[80px] h-[32px] bg-[#8B7EFF] rounded-full transition-transform duration-300 ease-in-out" 
            :class="[isTabActive === 'live' ? 'translate-x-[80px]' : '']"
          ></div>
          <span 
            class="relative z-10 w-[80px] text-center text-sm font-medium transition-colors duration-300" 
            :class="[isTabActive === 'paper' ? 'text-white' : 'text-gray-500']"
          >
            Paper
          </span>
          <span 
            class="relative z-10 w-[80px] text-center text-sm font-medium transition-colors duration-300" 
            :class="[isTabActive === 'live' ? 'text-white' : 'text-gray-500']"
          >
            Live
          </span>
        </div>
      </div>

      <div class="hidden md:flex items-center gap-2">
        <!-- Notification -->
        <PopupDropdown :isOpen="showNotificationModal" @close="closeNotificationModal" dropdownClass="dropdown-1 top-[100%] shadow-lg border border-third-light dark:border-third">
          <template #dropdown-toggle>
            <div class="p-2 2xl:p-2.5 mx-1 bg-[#1D1D20] border border-gray-500 rounded-lg" @click="toggleNotificationModal">
              <BellIcon class="w-4 h-4 2xl:w-5 2xl:h-5 cursor-pointer text-white" />
            </div>
          </template>

          <template #dropdown-body>
            <div class="max-h-[320px] overflow-y-scroll text-gray-600 dark:text-white text-center p-2">
              <div v-if="notifications.length > 0" class="cursor-pointer relative" v-for="notification in notifications" :key="notification.id">
                <div class="ml-2 overflow-hidden">
                  <div class="flex items-center">
                    <a href="javascript:;" class="font-medium truncate mr-5">{{notification.heading}}</a>
                    <div class="text-xs text-slate-400 ml-auto whitespace-nowrap">
                      {{notification.time}}
                    </div>
                  </div>
                  <div class="text-left truncate text-slate-400 mt-0.5">
                    Strategy: {{ notification.strategy ? notification.strategy.name : 'N/A' }}
                  </div>
                  <div class="text-left truncate text-slate-400 mt-0.5">
                    Exchange: {{ notification.exchange }}
                  </div>
                  <div class="text-left truncate text-slate-400 mt-0.5">
                    Side: {{ notification.side }}
                  </div>
                  <div class="text-left truncate text-slate-400 mt-0.5">
                    Symbol: {{ notification.tradingsymbol }}
                  </div>
                  <div class="text-left truncate text-slate-400 mt-0.5">
                    {{notification.message}}
                  </div>
                </div>
              </div>
              <span v-else>
                You have no notifications
              </span>
            </div>
          </template>
        </PopupDropdown>

        <!-- Profile -->
        <PopupDropdown :isOpen="showProfileModal" @close="closeProfileModal" dropdownClass="dropdown-1 max-w-56 top-[100%] right-[70%] shadow-lg border border-third-light dark:border-third">
          <template #dropdown-toggle>
            <div class="flex">
              <div class="p-2 2xl:p-2.5 mx-1 bg-[#1D1D20] border border-gray-500 rounded-lg" @click="toggleProfileModal">
                <UserIcon class="w-4 h-4 2xl:w-5 2xl:h-5 cursor-pointer text-white" />
              </div>
            </div>
          </template>
          <template #dropdown-body>
            <div class="font-bold text-xl">Profile</div>
            <div class="font-medium capitalize pb-2">
              {{ profile?.name }} <br>
              <span class="text-sm lowercase">{{ profile?.email }}</span>
            </div>
            <div @click="clickProfile" class="profile-option">
              <UserIcon class="icon-size ml-0" />My Profile
            </div>
            <div @click="clickRefresh" class="profile-option">
              <RefreshCwIcon class="icon-size p-[1px] ml-0" />Refresh
            </div>
            <div class="profile-option">
              <HelpCircleIcon class="icon-size ml-0" />Help
            </div>
            <div class="profile-option" @click="logout">
              <LockIcon class="icon-size ml-0" />Logout
            </div>
          </template>
        </PopupDropdown>
      </div>
    </div>
  </div>
  
  <!-- Add Broker Modal -->
  <AddEdit v-if="showAddEditModal" />
</template>

<script setup lang="ts">
import { logout } from '@/request/request'
import { ref, computed } from 'vue'
import { storeToRefs } from 'pinia'
import { useRouter, useRoute } from 'vue-router'
import { useProfileStore } from '@/stores/matrix/profile'
import { useNotificationsStore } from '@/stores/matrix/notification'
import { useStrategiesStore } from '@/stores/matrix/strategy'
import { useNavlinksStore } from '@/stores/navlinks'
import { usePositionsStore } from '@/stores/matrix/position'
import { useBrokersStore } from '@/stores/matrix/broker'
import AddEdit from '@/views/broker/addEdit.vue'
import PopupDropdown from '@/components/PopupDropdown.vue'
import { BellIcon, UserIcon, RefreshCwIcon, HelpCircleIcon, LockIcon, PlusIcon } from 'lucide-vue-next'

const router = useRouter()
const route = useRoute()
const notificationStore = useNotificationsStore()
const strategiesStore = useStrategiesStore()
const navlinksStore = useNavlinksStore()
const profileStore = useProfileStore()
const positionsStore = usePositionsStore()
const brokersStore = useBrokersStore()

const { navlinks } = storeToRefs(navlinksStore)
const { notificationsData } = storeToRefs(notificationStore)
const { strategies } = storeToRefs(strategiesStore)
const { profile } = storeToRefs(profileStore)
const { isTabActive } = storeToRefs(positionsStore)
const { showAddEditModal } = storeToRefs(brokersStore)

const openAddBrokerModal = () => {
  brokersStore.setShowAddEditModal(true)
}

const toggleMode = () => {
  if(isTabActive.value === 'live') {
    isTabActive.value = 'paper'
  } else {
    isTabActive.value = 'live'
  }
}

const title = computed(() => {
  let name = route.name?.toString() || ''
  if (name) {
    let titleData = navlinks.value.find((item: any) => item.name === name)
    return titleData && titleData.title ? titleData.title : name
  }
  return ''
})

const isDashboard = computed(() => {
  return route.name === 'dashboard'
})

const showNotificationModal = ref(false)
const showProfileModal = ref(false)

const notifications = computed(() => {
  if (
    notificationsData.value &&
    strategies.value &&
    strategies.value.length > 0 &&
    notificationsData.value.length > 0
  ) {
    const strategyMap = new Map(strategies.value.map((strategy: { id: any }) => [strategy.id, strategy]))
    return notificationsData.value.map((notification: { strategy_id: unknown }) => ({
      ...notification,
      strategy: strategyMap.get(notification.strategy_id) || null,
    }))
  }
  return []
})

const closeNotificationModal = () => {
  showNotificationModal.value = false
}

const toggleNotificationModal = () => {
  showNotificationModal.value = !showNotificationModal.value
}

const closeProfileModal = () => {
  showProfileModal.value = false
}

const toggleProfileModal = () => {
  showProfileModal.value = !showProfileModal.value
}

const clickProfile = () => {
  router.push({ name: 'profile' })
  showProfileModal.value = false
}

const clickRefresh = () => {
  window.location.reload()
}
</script>

<style scoped>
.profile-option {
  @apply flex items-center gap-2 px-4 py-2 text-sm text-gray-600 dark:text-white hover:bg-gray-100 dark:hover:bg-gray-800 cursor-pointer;
}

.icon-size {
  @apply w-4 h-4;
}
</style>