<template>
  <div>
    <table class="w-full table-auto" role="table">
      <thead>
        <tr>
          <th class="text-left p-2">Student</th>
          <th class="text-left p-2">Student ID</th>
          <th class="text-left p-2">Room</th>
          <th class="text-left p-2">Date</th>
          <th class="text-left p-2">Timeslot</th>
          <th class="text-left p-2">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="b in bookings" :key="b.id" class="border-t">
          <td class="p-2">{{ b.studentName }}</td>
          <td class="p-2">{{ b.studentId }}</td>
          <td class="p-2">{{ roomName(b.roomId) }}</td>
          <td class="p-2">{{ b.date }}</td>
          <td class="p-2">{{ b.timeslot }}</td>
          <td class="p-2 space-x-2">
            <button @click="onCancel(b.id)" class="px-2 py-1 bg-red-600 text-white rounded">Cancel</button>
            <button @click="onRelease(b.id)" class="px-2 py-1 bg-yellow-600 text-white rounded">Release</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { storeToRefs } from 'pinia'
import { useBookingsStore } from '../stores/bookings'
import { useRoomsStore } from '../stores/rooms'

const bookingsStore = useBookingsStore()
const roomsStore = useRoomsStore()
const { bookings } = storeToRefs(bookingsStore)
const { rooms } = storeToRefs(roomsStore)

const refresh = async () => {
  await roomsStore.fetchRooms()
  await bookingsStore.fetchBookings()
}

const onCancel = async (id) => {
  if (!confirm('Cancel this booking?')) return
  await bookingsStore.cancelBooking(id)
}

const onRelease = async (id) => {
  if (!confirm('Release this booking?')) return
  await bookingsStore.releaseBooking(id)
}

const roomName = (roomId) => {
  const r = roomsStore.rooms.find(r => r.id === roomId)
  return r ? r.name : '—'
}

onMounted(refresh)
</script>
