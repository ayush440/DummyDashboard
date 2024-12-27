<template>
  <div class="bg-[#1C1C1F] rounded-lg p-6">
    <div class="flex items-center justify-between mb-6">
      <h2 class="text-xl text-white font-medium">Personal detail</h2>
      <button 
        @click="toggleEdit" 
        class="px-4 py-2 rounded-lg bg-[#7C3AED] text-white hover:bg-[#6D28D9] transition-colors"
      >
        {{ isEditing ? "Editing" : "Edit" }}
      </button>
    </div>

    <form @submit.prevent="submitForm" class="space-y-6">
      <div>
        <label for="name" class="block text-sm font-medium text-gray-200 mb-2">Full name</label>
        <input
          type="text"
          id="name"
          v-model="profileData.name"
          :disabled="!isEditing"
          class="w-full  border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-[#7C3AED]"
          :placeholder="profile?.name"
        />
      </div>

      <div>
        <label for="number" class="block text-sm font-medium text-gray-200 mb-2">Number</label>
        <input
          type="text"
          id="number"
          v-model="profileData.mobile"
          :disabled="!isEditing"
          class="w-full bg-[#2C2C30] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-[#7C3AED]"
        />
      </div>

      <div>
        <label for="email" class="block text-sm font-medium text-gray-200 mb-2">Email</label>
        <input
          type="email"
          id="email"
          v-model="profileData.email"
          :disabled="!isEditing"
          class="w-full bg-[#2C2C30] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-[#7C3AED]"
        />
      </div>

      <div>
        <label for="address" class="block text-sm font-medium text-gray-200 mb-2">Address</label>
        <textarea
          id="address"
          v-model="profileData.address"
          :disabled="!isEditing"
          rows="3"
          class="w-full bg-[#2C2C30] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-[#7C3AED]"
        ></textarea>
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-200 mb-2">Profile Picture</label>
        <input
          type="file"
          accept="image/*"
          @change="handleFileChange"
          :disabled="!isEditing"
          class="w-full bg-[#2C2C30] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:bg-[#7C3AED] file:text-white hover:file:bg-[#6D28D9]"
        />
      </div>

      <div class="flex justify-end gap-4" v-if="isEditing">
        <button
          type="button"
          @click="toggleEdit"
          class="px-6 py-2 rounded-lg border border-[#3C3C40] text-white hover:bg-[#2C2C30]"
        >
          Cancel
        </button>
        <button
          type="submit"
          class="px-6 py-2 rounded-lg bg-[#7C3AED] text-white hover:bg-[#6D28D9]"
        >
          Update
        </button>
      </div>
    </form>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { makeRequest } from '@/request/request'

const props = defineProps({
  profile: Object
})

const isEditing = ref(false)
const profileData = ref({
  name: '',
  mobile: '',
  email: '',
  address: '',
  image: null as File | null
})

watch(() => props.profile, (newProfile) => {
  if (newProfile) {
    profileData.value = {
      name: newProfile.name || '',
      mobile: newProfile.mobile || '',
      email: newProfile.email || '',
      address: newProfile.address || '',
      image: null
    }
  }
}, { immediate: true })

const toggleEdit = () => {
  isEditing.value = !isEditing.value
  if (!isEditing.value) {
    // Reset form when canceling edit
    if (props.profile) {
      profileData.value = {
        name: props.profile.name || '',
        mobile: props.profile.mobile || '',
        email: props.profile.email || '',
        address: props.profile.address || '',
        image: null
      }
    }
  }
}

const handleFileChange = (event: Event) => {
  const target = event.target as HTMLInputElement
  if (target.files?.length) {
    profileData.value.image = target.files[0]
  }
}

const submitForm = async () => {
  try {
    const formData = new FormData()
    Object.entries(profileData.value).forEach(([key, value]) => {
      if (value !== null) {
        formData.append(key, value)
      }
    })
    
    await makeRequest('updateUser', 'PUT', formData)
    isEditing.value = false
  } catch (error) {
    console.error('Error updating profile:', error)
  }
}
</script>