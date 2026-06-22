<template>
  <form @submit.prevent="onSubmit" class="space-y-3" novalidate>
    <div>
      <label class="block text-sm font-medium">Name</label>
      <input v-model="form.name" required class="mt-1 block w-full border rounded p-2" />
    </div>
    <div>
      <label class="block text-sm font-medium">Capacity</label>
      <input v-model.number="form.capacity" type="number" min="1" required class="mt-1 block w-full border rounded p-2" />
    </div>
    <div>
      <label class="block text-sm font-medium">Location</label>
      <input v-model="form.location" required class="mt-1 block w-full border rounded p-2" />
    </div>
    <div class="flex items-center gap-3">
      <button class="px-3 py-2 bg-blue-600 text-white rounded">Add Room</button>
      <span role="status" aria-live="polite" class="text-sm text-green-600">{{ msg }}</span>
    </div>
  </form>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { useRoomsStore } from '../stores/rooms'

const form = reactive({ name: '', capacity: 1, location: '', available: true })
const msg = ref('')
const store = useRoomsStore()

const onSubmit = async () => {
  try {
    await store.addRoom({ ...form })
    msg.value = 'Room added.'
    form.name = ''
    form.capacity = 1
    form.location = ''
  } catch (e) {
    msg.value = 'Error adding room.'
  }
}
</script>
