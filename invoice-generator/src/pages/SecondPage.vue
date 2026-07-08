<template>
  <q-page class="q-pa-lg bg-grey-2">
    <div class="row q-col-gutter-lg">

      <div class="col-12 col-md-5">
        <q-card flat bordered class="q-pa-md bg-white">
          <div class="text-h6 q-mb-md">Enter Order Details</div>

          <q-input v-model="invoice.number" label="Invoice / Order #" outlined dense class="q-mb-sm" />
          <q-input v-model="invoice.customer" label="Customer Name" outlined dense class="q-mb-sm" />
          <q-input v-model="invoice.items" label="Items purchased" outlined dense class="q-mb-sm" />
          <q-input v-model.number="invoice.total" type="number" label="Total Amount ($)" outlined dense class="q-mb-lg" />

          <q-btn
            color="positive"
            icon="download"
            label="Save & Download JPEG"
            class="full-width"
            :loading="isGenerating"
            @click="exportAsJpeg"
          />
        </q-card>
      </div>

      <div class="col-12 col-md-7">
        <div class="text-caption text-grey-7 q-mb-xs">Live Export Preview:</div>

        <div
          ref="invoiceRef"
          class="q-pa-xl shadow-2"
          style="width: 100%; max-width: 550px; background-color: #ffffff; color: #000000;"
        >
          <div class="row justify-between items-center q-mb-xl">
            <div>
              <div class="text-h6 text-weight-bold text-primary">YOUR SMALL BUSINESS</div>
              <div class="text-caption text-grey-6">Order Confirmation Receipt</div>
            </div>
            <div class="text-right">
              <div class="text-subtitle1 text-weight-bold">ORDER SLIP</div>
              <div class="text-caption">No: {{ invoice.number || '---' }}</div>
            </div>
          </div>

          <div class="q-mb-lg">
            <div class="text-caption text-grey-6 text-uppercase">Billed To:</div>
            <div class="text-subtitle1 text-weight-bold">{{ invoice.customer || 'Guest Customer' }}</div>
          </div>

          <q-markup-table flat bordered dense class="q-mb-xl">
            <thead>
              <tr>
                <th class="text-left">Item Description</th>
                <th class="text-right">Amount</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td class="text-left">{{ invoice.items || 'No description added yet' }}</td>
                <td class="text-right text-weight-bold">${{ invoice.total || 0 }}.00</td>
              </tr>
            </tbody>
          </q-markup-table>

          <div class="row justify-end">
            <div class="col-6 bg-primary text-white q-pa-md text-right rounded-borders">
              <div class="text-caption text-uppercase style-opacity-70">Total Due</div>
              <div class="text-h5 text-weight-bold">${{ invoice.total || 0 }}.00</div>
            </div>
          </div>
        </div>

      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useQuasar } from 'quasar'
import * as htmlToImage from 'html-to-image'

const $q = useQuasar()
const invoiceRef = ref(null)
const isGenerating = ref(false)

// Component Reactive Form Data State
const invoice = reactive({
  number: 'INV-1001',
  customer: '',
  items: '',
  total: null
})

// Core JPEG Generation Method
const exportAsJpeg = async () => {
  if (!invoiceRef.value) return

  isGenerating.value = true
  const notification = $q.notify({
    type: 'ongoing',
    message: 'Rendering image wrapper canvas...',
    position: 'bottom'
  })

  try {
    // Generate JPEG data URL at crystal-clear quality
    const dataUrl = await htmlToImage.toJpeg(invoiceRef.value, {
      quality: 0.98,
      pixelRatio: 2,           // Keeps the exported texts sharp on Retina/mobile screens
      backgroundColor: '#ffffff' // Ensures no transparent layout artifacts
    })

    // Initiate local browser download pipeline automatically
    const link = document.createElement('a')
    link.download = `Receipt_${invoice.number || 'Draft'}.jpeg`
    link.href = dataUrl
    link.click()

    notification() // Close loading state
    $q.notify({ type: 'positive', message: 'JPEG downloaded! Ready to send.', position: 'bottom' })
  } catch (error) {
    notification()
    console.error('Render error layout failed:', error)
    $q.notify({ type: 'negative', message: 'Error compiling layout into picture.', position: 'bottom' })
  } finally {
    isGenerating.value = false
  }
}
</script>
