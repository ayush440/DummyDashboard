<template>
    <!-- logo-topbar -->
    <div class="logo-topbar bg-[#1d1d20] flex justify-between items-center relative p-2 pb-1">

        <div class="flex items-center ">
            <div class="flex items-center">
                <!-- <DarkModeSwicher /> -->
                <div class="cursor-pointer ml-2 hover:bg-third-bold rounded-lg transition duration-200 dark:text-tableText"
                    :class="{ 'rotate-0': !toggle, 'rotate-[180deg]': toggle }">
                    <MenuIcon v-if="!toggle" class="m-2 w-6 h-6 transition transition-delay-300 hover:scale-110"
                        @click="toggle = !toggle, mobileToggle = !mobileToggle" />
                    <CloseIcon v-else @click="toggle = !toggle, mobileToggle = !mobileToggle"
                        class="m-2 w-6 h-6  transition transition-delay-300 hover:scale-110"
                        :class="{ 'rotate-0': toggle, 'rotate-[180deg]': !toggle }" />
                </div>
            </div>
            <RouterLink to="/">
                <img src="/public/InfinityParker.png" @load="handleImageLoad" @error="handleImageError" class="w-24" />
                <img v-if="imgName" :src="'' + imgName + '.png'" @load="handleImageLoad" @error="handleImageError"
                    class="w-[228px] h-16" :class="{ 'hidden': !imgLoad }" />
                <div v-if="!imgLoad"
                    class="flex h-16 w-full font-bold justify-center  border border-red-600 items-center mt-[-5px] text-3xl text-gray-600 dark:text-tableText">
                    {{ domainName }}
                </div>
            </RouterLink>
        </div>



        <div class="flex gap-2">
            <button @click="() => showAddEditModal = true"
                class="flex items-center gap-1 bg-[#1d1d20] border border-gray-600 hover:bg-[#7A6DFF] text-white px-2 rounded-lg transition-colors duration-200">
                <PlusIcon class="w-4 h-4" />
                <span class="text-sm font-medium">Add broker</span>
            </button>

            <div>
                <img class="w-12 h-12 rounded-full"
                    :src="profile.profile_picture_url ? profile.profile_picture_url : images['profile']" alt="" />
            </div>
        </div>
    </div>

    <AddEdit v-if="showAddEditModal" />
    <teleport to="body">
        <div class="visible lg:hidden h-full bg-[#1d1d20] text-[#888888]">
            <transition name="slide-fade">
                <div v-if="toggle" @click.self="closeLogoTopbar"
                    class="fixed h-full w-full menuBar overflow-y-scroll top-0 left-0"
                    :style="{ zIndex: zIndexValue + 2 }">
                    <div ref="logoTopbar"
                        class="bg-[#1d1d20] overflow-y-auto overflow-x-hidden h-full menuBar-shadow w-[250px] transform"
                        :class="{ 'translate-x-0': toggle, '-translate-x-full': !toggle }">

                        <!-- profile and notification menu -->


                        <!-- <div class="flex justify-between mr-2">  -->
                        <div class="pb-4  ml-1 flex flex-col shadow">

                            <div class="unselectable flex flex-col p-1 pt-8 pl-2">
                                <div class="flex items-center cursor-pointer" >
                                    <img class="w-12 h-12 rounded-full"
                                        :src="profile.profile_picture_url ? profile.profile_picture_url : images['profile']"
                                        alt="" />
                                    <div class="ml-2">
                                        <div class="font-bold text-[#EFEFEF]">{{ profile?.name }}</div>
                                        <div class=" dark:text-tableText">
                                            {{ profile?.user_role }}
                                            
                                        </div>
                                    </div>
                                </div>
                            </div>

                        </div>

                        <ul class="flex flex-col p-1 mt-4 pl-2" v-if="navlinks.length > 0 && allRoutes.length > 0">

                            <routeName v-for="(navData, index) in navlinks" :key="index" :item="navData"
                            :route="findRoutes(navData.name)" />

                        </ul>



                    </div>
                </div>
            </transition>
        </div>
    </teleport>
</template>


<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { logout } from '@/request/request'
import router from '@/router/index'
import { ref, computed, watchEffect, onBeforeMount } from 'vue'
import { images } from '@/assets/img'
import DarkModeSwicher from '@/components/darkModeSwicher.vue'
import { useNavlinksStore } from '@/stores/navlinks'
import routeName from './routeName.vue'
import { useProfileStore } from '@/stores/matrix/profile'
import { useNotificationsStore } from '@/stores/matrix/notification';
import { useStrategiesStore } from '@/stores/matrix/strategy'
import { useLogoStore } from '@/stores/utils/logo'
import { PlusIcon } from 'lucide-vue-next'
import AddEdit from '@/views/broker/addEdit.vue'
import { useBrokersStore } from '@/stores/matrix/broker'

const notificationStore = useNotificationsStore()
const strategiesStore = useStrategiesStore()
const brokerStore = useBrokersStore()
const { showAddEditModal } = storeToRefs(brokerStore)

let { notificationsData } = storeToRefs(notificationStore)

const { strategies } = storeToRefs(strategiesStore)

const navlinkStore = useNavlinksStore()
const { navlinks, mobileToggle } = storeToRefs(navlinkStore)
const profileStore = useProfileStore()
const { profile } = storeToRefs(profileStore)

const logoStore = useLogoStore()
const { logoData } = storeToRefs(logoStore)


const toggle = ref(false)
const showProfileModal = ref(false);
const showNotificationModal = ref(false);

const allRoutes = computed(() => router.getRoutes())


const findRoutes = (name: string) => {
    return allRoutes.value.find((route) => route.name === name)
}


onBeforeMount(() => {
    const url = window.location.href
    getDomain(url)
})

const imgName = ref('')
const imgLoad = ref(false)


const domainName = computed(() => {
    return logoData.value
})


function handleImageLoad() {
    imgLoad.value = true
}
function handleImageError(error: any) {
    imgLoad.value = false
}

const getDomain = (url: string) => {
    const parsedUrl = new URL(url);
    // const parsedUrl = new URL('https://www.admin.xyz.in.netlify.app/');
    let count = 0, newHostname = '';
    let urlHostNameArray = parsedUrl.hostname.split(".");

    for (let i = 0; i < urlHostNameArray.length; i++) {
        if (urlHostNameArray[i] == "www" && count == 0) {
            continue;
        } else if (count == 0) {
            count = 1;
        } else {
            newHostname += urlHostNameArray[i] + "_"
        }
    }

    imgName.value = newHostname.slice(0, -1);
}

watchEffect(() => {
    if (!mobileToggle.value) {
        toggle.value = false
    }
})

let notifications = computed<any>(() => {
    if (
        notificationsData.value &&
        strategies.value &&
        strategies.value.length > 0 &&
        notificationsData.value.length > 0
    ) {
        // Map strategy IDs to their corresponding strategies for efficient lookup
        const strategyMap = new Map(strategies.value.map((strategy: { id: any; }) => [strategy.id, strategy]));

        // Associate each notification with its corresponding strategy
        const notificationsWithStrategies = notificationsData.value.map((notification: { strategy_id: unknown; }) => {
            const strategy = strategyMap.get(notification.strategy_id);
            return {
                ...notification,
                strategy: strategy || null,
            };
        });
        return notificationsWithStrategies;
    }

    return [];
});



const clickProfile = () => {
    router.push({ name: 'profile' })
    showProfileModal.value = true
    closeLogoTopbar()
}

const clickRefresh = () => {
    window.location.reload()
}

const closeProfileModal = () => {
    showProfileModal.value = false
}

const toggleProfileModal = () => {
    showProfileModal.value = !showProfileModal.value
}

const closeNotificationModal = () => {
    showNotificationModal.value = false
}

const toggleNotificationModal = () => {
    showNotificationModal.value = !showNotificationModal.value
}



function getHighestZIndex() {
    const elements = [...document.querySelectorAll('body *')] as HTMLElement[]
    const zIndexes = elements.map(el => parseFloat(window.getComputedStyle(el).zIndex))
        .filter(zIndex => !isNaN(zIndex))
    return Math.max(0, ...zIndexes)
}

// Compute the zIndex value dynamically
const zIndexValue = computed(() => getHighestZIndex() + 1)


const closeLogoTopbar = () => {
    toggle.value = false
    mobileToggle.value = false
}

</script>


<style lang="scss" scoped>
@media (max-width: 1020px) {
    .menuBar {
        display: block;
    }
}

.menuBar-shadow {
    box-shadow: 0 1px 30px 1px rgba(0, 0, 0, 0.4);
}

.profile-option {
    @apply hover:bg-primary hover:bg-opacity-10 dark:hover:bg-opacity-40 cursor-pointer flex bg-secondary dark:bg-primary dark:bg-opacity-50 dark:shadow-slate-600 rounded p-1 my-1 shadow-sm;
}

.icon-size {
    @apply m-1 h-4 w-4;
}


.menuBar {
    transition: transform 0.7s ease-in-out;
}

.slide-fade-enter-active,
.slide-fade-leave-active {
    transition: transform 0.7s ease-in-out;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
    transform: translateX(-100%);
}
</style>