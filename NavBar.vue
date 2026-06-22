<template>
  <nav aria-label="Main navigation">
    <ul class="flex gap-3" role="menubar">
      <li role="none"><RouterLink role="menuitem" to="/rooms" class="px-3 py-2 rounded hover:bg-gray-100">Rooms</RouterLink></li>
      <li role="none"><RouterLink role="menuitem" to="/booking" class="px-3 py-2 rounded hover:bg-gray-100">Book a Room</RouterLink></li>
      <li role="none"><RouterLink role="menuitem" to="/bookings" class="px-3 py-2 rounded hover:bg-gray-100">Bookings</RouterLink></li>
    </ul>
  </nav>
</template>

<script setup>
import { RouterLink } from 'vue-router'
</script>
