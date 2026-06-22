<template>
  <div class="min-h-screen bg-gray-50 text-gray-900">
    <header class="bg-white shadow">
      <div class="max-w-5xl mx-auto p-4 flex items-center justify-between">
        <h1 class="text-2xl font-semibold">Room Booking</h1>
        <NavBar />
      </div>
    </header>

    <main class="max-w-5xl mx-auto p-4">
      <router-view />
    </main>

    <footer class="max-w-5xl mx-auto p-4 text-sm text-gray-600">
      <p>Room Booking Dashboard — manage rooms, bookings, and availability in one place.</p>
    </footer>
  </div>
</template>

<script setup>
import NavBar from './components/NavBar.vue'
</script>
