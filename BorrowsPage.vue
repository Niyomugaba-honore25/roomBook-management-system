<template>
  <section class="page">
    <div class="page-top">
      <div>
        <p class="eyebrow">Borrow Center</p>
        <h2>Issue and return books quickly</h2>
      </div>
      <span class="tag">Live borrowing feed</span>
    </div>

    <div class="stats-row">
      <article class="mini-card">
        <p>Total borrows</p>
        <strong>{{ borrows.length }}</strong>
      </article>
      <article class="mini-card">
        <p>Available books</p>
        <strong>{{ availableBookCount }}</strong>
      </article>
      <article class="mini-card">
        <p>Borrowed books</p>
        <strong>{{ borrowedBooks }}</strong>
      </article>
    </div>

    <div class="card form-card">
      <div class="card-header">
        <h3>Borrow a Book</h3>
        <span class="tag">Fast loan</span>
      </div>
      <form @submit.prevent="createBorrow">
        <label>
          Member Name
          <input v-model="borrowForm.memberName" required />
        </label>
        <label>
          Member ID
          <input v-model="borrowForm.memberId" required />
        </label>
        <label>
          Book
          <select v-model.number="borrowForm.bookId" required>
            <option value="" disabled>Select a book</option>
            <option v-for="book in availableBookList" :key="book.id" :value="book.id">
              {{ book.title }} — {{ book.author }}
            </option>
          </select>
        </label>
        <label>
          Borrow Date
          <input type="date" v-model="borrowForm.borrowDate" required />
        </label>
        <label>
          Return Date
          <input type="date" v-model="borrowForm.returnDate" required />
        </label>
        <button type="submit">Submit Borrow</button>
      </form>
      <p v-if="message" class="message">{{ message }}</p>
    </div>

    <div class="card list-card">
      <div class="card-header">
        <h3>Borrow Records</h3>
        <span class="tag">{{ borrows.length }} active</span>
      </div>
      <div v-if="loading">Loading records...</div>
      <div v-else-if="borrows.length === 0">No borrow records found.</div>
      <table v-else>
        <thead>
          <tr>
            <th>Member</th>
            <th>Member ID</th>
            <th>Book</th>
            <th>Borrow Date</th>
            <th>Return Date</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="borrow in borrows" :key="borrow.id">
            <td>{{ borrow.memberName }}</td>
            <td>{{ borrow.memberId }}</td>
            <td>{{ bookTitle(borrow.bookId) }}</td>
            <td>{{ borrow.borrowDate }}</td>
            <td>{{ borrow.returnDate }}</td>
            <td>
              <button @click="returnBorrow(borrow.id)">Return</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

const borrows = ref([]);
const books = ref([]);
const loading = ref(false);
const message = ref('');
const borrowForm = ref({ memberName: '', memberId: '', bookId: '', borrowDate: '', returnDate: '' });
const apiBase = import.meta.env.VITE_API_BASE || 'http://localhost:4000';

const availableBookCount = computed(() => books.value.filter(book => book.available).length);
const availableBooks = computed(() => books.value.filter(book => book.available));
const borrowedBooks = computed(() => books.value.length - availableBookCount.value);

const fetchBooks = async () => {
  try {
    const response = await fetch(`${apiBase}/api/books`);
    if (!response.ok) throw new Error('Failed to load books');
    books.value = await response.json();
  } catch (error) {
    message.value = `Error: ${error.message}`;
  }
};

const fetchBorrows = async () => {
  loading.value = true;
  message.value = '';
  try {
    const response = await fetch(`${apiBase}/api/borrows`);
    if (!response.ok) throw new Error('Failed to load borrow records');
    borrows.value = await response.json();
  } catch (error) {
    message.value = `Error: ${error.message}`;
  } finally {
    loading.value = false;
  }
};

const availableBookList = computed(() => books.value.filter(book => book.available));

const bookTitle = (id) => {
  const book = books.value.find((item) => item.id === id);
  return book ? `${book.title} (${book.author})` : 'Unknown book';
};

const createBorrow = async () => {
  message.value = '';
  try {
    const response = await fetch(`${apiBase}/api/borrows`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(borrowForm.value)
    });
    if (response.status !== 201) {
      const errorText = await response.text();
      throw new Error(errorText || 'Could not create borrow record');
    }
    message.value = 'Borrow record created successfully.';
    borrowForm.value = { memberName: '', memberId: '', bookId: '', borrowDate: '', returnDate: '' };
    await fetchBooks();
    await fetchBorrows();
  } catch (error) {
    message.value = `Error: ${error.message}`;
  }
};

const returnBorrow = async (id) => {
  message.value = '';
  try {
    const response = await fetch(`${apiBase}/api/borrows/${id}`, {
      method: 'DELETE'
    });
    if (response.status !== 204) {
      const errorText = await response.text();
      throw new Error(errorText || 'Could not return borrow record');
    }
    message.value = 'Book returned successfully.';
    await fetchBooks();
    borrows.value = borrows.value.filter((borrow) => borrow.id !== id);
  } catch (error) {
    message.value = `Error: ${error.message}`;
  }
};

onMounted(async () => {
  await fetchBooks();
  await fetchBorrows();
});
</script>
