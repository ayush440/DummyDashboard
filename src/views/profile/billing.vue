<template>
  <div class="bg-[#1C1C1F] rounded-lg p-6">
    <div class="flex items-center justify-between mb-6">
      <h2 class="text-xl text-white font-medium">Billing info</h2>
      <button 
        @click="showEdit"
        class="px-4 py-2 rounded-lg bg-[#7C3AED] text-white hover:bg-[#6D28D9] transition-colors"
      >
        Create Invoice
      </button>
    </div>

    <div class="space-y-6">
      <div class="bg-[#2C2C30] rounded-lg p-6">
        <div class="flex justify-center mb-6">
          <div class="w-32 h-32">
            <img 
              src="/public/qrcode.png" 
              alt="QR Code"
              class="w-full h-full"
            />
          </div>
        </div>

        <div class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-200 mb-2">
              Select plan
            </label>
            <select 
              class="w-full bg-[#1C1C1F] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white appearance-none focus:outline-none focus:ring-2 focus:ring-[#7C3AED]"
            >
              <option value="" disabled selected>Select a plan</option>
              <option v-for="plan in plans" :key="plan.id" :value="plan.id">
                {{ plan.name }}
              </option>
            </select>
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-200 mb-2">
              Promo code
            </label>
            <input
              type="text"
              class="w-full bg-[#1C1C1F] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-[#7C3AED]"
              placeholder="Enter promo code"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-200 mb-2">
              Add payment proof
            </label>
            <input
              type="file"
              accept="image/*"
              class="w-full bg-[#1C1C1F] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:bg-[#7C3AED] file:text-white hover:file:bg-[#6D28D9]"
            />
          </div>

          <div class="grid grid-cols-3 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-200 mb-2">
                Total amount
              </label>
              <input
                type="text"
                readonly
                :value="totalAmount"
                class="w-full bg-[#1C1C1F] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white"
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-200 mb-2">
                Discount
              </label>
              <input
                type="text"
                readonly
                :value="discount"
                class="w-full bg-[#1C1C1F] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white"
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-200 mb-2">
                Payable amount
              </label>
              <input
                type="text"
                readonly
                :value="payableAmount"
                class="w-full bg-[#1C1C1F] border border-[#3C3C40] rounded-lg px-4 py-2.5 text-white"
              />
            </div>
          </div>
        </div>
      </div>

      <div class="space-y-4">
        <h3 class="text-lg font-medium text-white">Recent Invoices</h3>
        <div v-if="invoices.length > 0" class="space-y-4">
          <div 
            v-for="invoice in invoices" 
            :key="invoice.id"
            class="flex items-center justify-between p-4 bg-[#2C2C30] rounded-lg"
          >
            <div class="flex items-center gap-4">
              <FilesIcon class="w-6 h-6 text-gray-400" />
              <div>
                <p class="text-white">Due Date: {{ formatDate(invoice.due_date) }}</p>
                <p class="text-gray-400">Status: {{ invoice.payment_status }}</p>
              </div>
            </div>
            <div class="flex items-center gap-4">
              <button
                @click="showInvoice(invoice)"
                class="text-gray-400 hover:text-white"
              >
                <EyeIcon class="w-5 h-5" />
              </button>
              <button
                @click="showPaymentProof(invoice)"
                class="flex items-center gap-2 text-[#7C3AED] hover:text-[#6D28D9]"
              >
                <UploadIcon class="w-5 h-5" />
                <span>Add Payment Proof</span>
              </button>
            </div>
          </div>
        </div>
        <div v-else class="text-center text-gray-400 py-8">
          No invoices found
        </div>
      </div>
    </div>

    <createInvoiceModal />
    <invoiceReceiptModal />
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { storeToRefs } from 'pinia'
import { FilesIcon, EyeIcon, UploadIcon } from 'lucide-vue-next'
import { useInvoicesStore } from '@/stores/matrix/invoice'
import createInvoiceModal from './createInvoiceModal.vue'
import invoiceReceiptModal from './invoiceReceiptModal.vue'

const invoiceStore = useInvoicesStore()
const { showAddEditModal, showInvoiceReceiptModal, invoiceData, addEditInvoiceData } = storeToRefs(invoiceStore)

const invoices = computed(() => Object.values(invoiceStore.invoices))

const plans = [
  { id: 1, name: 'Basic Plan' },
  { id: 2, name: 'Pro Plan' },
  { id: 3, name: 'Enterprise Plan' }
]

const totalAmount = computed(() => '$0.00')
const discount = computed(() => '$0.00')
const payableAmount = computed(() => '$0.00')

const formatDate = (date: string) => {
  return date.replace('T', ' ').replace('Z', '')
}

const showInvoice = (invoice: any) => {
  invoiceData.value = invoice
  showInvoiceReceiptModal.value = true
}

const showPaymentProof = (invoice: any) => {
  addEditInvoiceData.value = invoice
  showAddEditModal.value = true
}

const showEdit = () => {
  addEditInvoiceData.value = {}
  showAddEditModal.value = true
}
</script>