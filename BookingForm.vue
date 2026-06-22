<template>
  <form @submit.prevent="onSubmit" class="space-y-3" aria-live="polite">
    <div>
      <label class="block text-sm font-medium">Student Name</label>
      <input v-model="form.studentName" required class="mt-1 block w-full border rounded p-2" />
    </div>
    <div>
      <label class="block text-sm font-medium">Student ID</label>
      <input v-model="form.studentId" required class="mt-1 block w-full border rounded p-2" />
    </div>
    <div>
      <label class="block text-sm font-medium">Room</label>
      <select v-model="form.roomId" required class="mt-1 block w-full border rounded p-2">
        <option disabled value="">Select a room</option>
        <option v-for="r in rooms" :key="r.id" :value="r.id">{{ r.name }} ({{ r.location }})</option>
      </select>
    </div>
    <div>
      <label class="block text-sm font-medium">Booking Date</label>
      <input v-model="form.date" type="date" required class="mt-1 block w-full border rounded p-2" />
    </div>
    <div>
      <label class="block text-sm font-medium">Timeslot</label>
      <select v-model="form.timeslot" required class="mt-1 block w-full border rounded p-2">
        <option value="08:00-10:00">08:00-10:00</option>
        <option value="10:00-12:00">10:00-12:00</option>
        <option value="14:00-16:00">14:00-16:00</option>
      </select>
    </div>

    <div class="flex items-center gap-3">
      <button :disabled="submitting || !isValid()" class="px-3 py-2 bg-green-600 text-white rounded disabled:opacity-50">Submit Booking</button>
      <span class="text-sm" aria-live="polite">{{ status }}</span>
    </div>
  </form>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'
import { storeToRefs } from 'pinia'
import { useRoomsStore } from '../stores/rooms'
import { useBookingsStore } from '../stores/bookings'

const roomsStore = useRoomsStore()
const bookingsStore = useBookingsStore()
const { rooms } = storeToRefs(roomsStore)

const form = reactive({ studentName: '', studentId: '', roomId: '', date: '', timeslot: '08:00-10:00' })
const status = ref('')
const submitting = ref(false)

const isValid = () => {
  return form.studentName && form.studentId && form.roomId && form.date && form.timeslot
}

const onSubmit = async () => {
  if (!isValid()) {
    status.value = 'Please fill all required fields.'
    return
  }

  submitting.value = true
  status.value = ''
  try {
    await bookingsStore.createBooking({
      studentName: form.studentName,
      studentId: form.studentId,
      roomId: form.roomId,
      date: form.date,
      timeslot: form.timeslot
    })
    status.value = 'Booking successful.'
    form.studentName = ''
    form.studentId = ''
    form.roomId = ''
    form.date = ''
  } catch (e) {
    // show detailed error when available
    const detail = e?.response?.data || e?.message || 'Unknown error'
    try {
      status.value = typeof detail === 'string' ? detail : JSON.stringify(detail)
    } catch (_) {
      status.value = 'Error creating booking.'
    }
  } finally {
    submitting.value = false
  }
}

onMounted(async () => {
  if (rooms.length === 0) await roomsStore.fetchRooms()
  await bookingsStore.fetchBookings()
})
</script>
