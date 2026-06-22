<template>
  <section class="dashboard page">
    <div class="hero-card">
      <div>
        <p class="eyebrow">Library Dashboard</p>
        <h2>Everything your library needs in one place</h2>
        <p>Manage inventory, track borrowing, and keep overdue books under control with an elegant dashboard.</p>
      </div>
      <div class="hero-actions">
        <button @click="$emit('navigate', 'books')">Manage Books</button>
        <button class="secondary" @click="$emit('navigate', 'borrows')">Manage Borrows</button>
      </div>
    </div>

    <div class="dashboard-grid">
      <article class="metric-card blue-card">
        <BookOpen class="metric-icon" />
        <p>Total Books</p>
        <strong>{{ totalBooks }}</strong>
      </article>
      <article class="metric-card green-card">
        <CheckCircle class="metric-icon" />
        <p>Available Books</p>
        <strong>{{ availableBooks }}</strong>
      </article>
      <article class="metric-card orange-card">
        <ArrowRightLeft class="metric-icon" />
        <p>Borrowed Books</p>
        <strong>{{ borrowedBooks }}</strong>
      </article>
      <article class="metric-card purple-card">
        <FileText class="metric-icon" />
        <p>Borrow Records</p>
        <strong>{{ totalBorrows }}</strong>
      </article>
      <article class="metric-card red-card">
        <AlertTriangle class="metric-icon" />
        <p>Overdue Books</p>
        <strong>{{ overdueBooks }}</strong>
      </article>
      <article class="metric-card teal-card">
        <Plus class="metric-icon" />
        <p>Recent Additions</p>
        <strong>{{ recentBooks }}</strong>
      </article>
    </div>

    <div class="grid-two board-grid">
      <div class="card recent-card">
        <div class="section-header">
          <h3>Recent Transactions</h3>
          <button class="secondary small">View All</button>
        </div>
        <div v-if="loading">Loading transactions...</div>
        <ul v-else class="transaction-list">
          <li v-for="borrow in recentTransactions" :key="borrow.id">
            <div>
              <strong>{{ borrow.memberName }}</strong>
              <p>{{ bookTitle(borrow.bookId) }}</p>
            </div>
            <div>
              <span class="status-pill">Borrowed</span>
              <small>{{ borrow.borrowDate }}</small>
            </div>
          </li>
        </ul>
      </div>

      <div class="card category-card">
        <div class="section-header">
          <h3>Book Categories</h3>
          <span>{{ totalCategories }} categories</span>
        </div>
        <ul class="category-list">
          <li v-for="category in bookCategoryList" :key="category.name">
            <div class="category-row">
              <span>{{ category.name }}</span>
              <strong>{{ category.count }}</strong>
            </div>
            <div class="category-bar-wrap">
              <div class="category-bar" :style="{ width: category.percent + '%' }"></div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import { BookOpen, CheckCircle, ArrowRightLeft, FileText, AlertTriangle, Plus } from 'lucide-vue-next';

const props = defineProps({});
const emit = defineEmits(['navigate']);
const books = ref([]);
const borrows = ref([]);
const loading = ref(true);
const apiBase = import.meta.env.VITE_API_BASE || 'http://localhost:4000';

const fetchBooks = async () => {
  const response = await fetch(`${apiBase}/api/books`);
  if (!response.ok) throw new Error('Failed to load books');
  books.value = await response.json();
};

const fetchBorrows = async () => {
  const response = await fetch(`${apiBase}/api/borrows`);
  if (!response.ok) throw new Error('Failed to load borrow records');
  borrows.value = await response.json();
};

onMounted(async () => {
  loading.value = true;
  try {
    await Promise.all([fetchBooks() || [] , fetchBorrows() || []]);
  } catch (error) {
    console.error(error);
  } finally {
    loading.value = false;
  }
});

const totalBooks = computed(() => books.value.length);
const availableBooks = computed(() => books.value.filter(book => book.available).length);
const borrowedBooks = computed(() => totalBooks.value - availableBooks.value);
const totalBorrows = computed(() => borrows.value.length);
const overdueBooks = computed(() => {
  const today = new Date();
  return borrows.value.filter((borrow) => {
    const due = new Date(borrow.returnDate);
    return due < today;
  }).length;
});
const recentBooks = computed(() => books.value.slice(-3).length);
const totalCategories = computed(() => bookCategoryList.value.length);
const bookCategoryList = computed(() => {
  const counts = {};
  books.value.forEach((book) => {
    const category = book.category || 'Uncategorized';
    counts[category] = (counts[category] || 0) + 1;
  });

  const entries = Object.entries(counts).map(([name, count]) => ({ name, count }));
  const max = Math.max(...entries.map((item) => item.count), 1);
  return entries.map((item) => ({
    ...item,
    percent: Math.round((item.count / max) * 100)
  }));
});
const recentTransactions = computed(() => {
  return [...borrows.value]
    .sort((a, b) => new Date(b.borrowDate) - new Date(a.borrowDate))
    .slice(0, 5);
});

const bookTitle = (id) => {
  const book = books.value.find((item) => item.id === id);
  return book ? `${book.title} (${book.author})` : 'Unknown book';
};
</script>
 