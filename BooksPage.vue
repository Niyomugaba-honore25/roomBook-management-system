<template>
  <section class="books-page">
    <div class="books-header">
      <h2>Books</h2>
      <button class="primary-button" @click="openInsertDialog">Insert Book</button>
    </div>

    <div v-if="loading" class="status-message">Loading books...</div>
    <div v-else>
      <table v-if="books.length" class="books-table">
        <thead>
          <tr>
            <th>Title</th>
            <th>Author</th>
            <th>Category</th>
            <th>Available</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="book in books" :key="book.id">
            <td>{{ book.title }}</td>
            <td>{{ book.author }}</td>
            <td>{{ book.category }}</td>
            <td>{{ book.available ? 'Yes' : 'No' }}</td>
            <td class="actions-cell">
              <button class="action-button" @click="openEditDialog(book)">Edit</button>
              <button class="action-button danger" @click="deleteBook(book.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>

      <div v-else class="status-message">No books found.</div>
    </div>

    <div v-if="showDialog" class="dialog-overlay" role="dialog" aria-modal="true">
      <div class="dialog-box">
        <header class="dialog-header">
          <h3>{{ isEditMode ? 'Edit Book' : 'Insert Book' }}</h3>
          <button class="close-button" @click="closeDialog">×</button>
        </header>

        <form @submit.prevent="saveBook" class="dialog-form">
          <label>
            Title
            <input v-model="newBook.title" required />
          </label>

          <label>
            Author
            <input v-model="newBook.author" required />
          </label>

          <label>
            Category
            <input v-model="newBook.category" required />
          </label>

          <label class="checkbox-label">
            <input type="checkbox" v-model="newBook.available" />
            Available
          </label>

          <div class="dialog-actions">
            <button type="button" @click="closeDialog">Cancel</button>
            <button type="submit" class="primary-button">
              {{ isEditMode ? 'Save Changes' : 'Create Book' }}
            </button>
          </div>
        </form>

        <p v-if="message" class="message">{{ message }}</p>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import bookService from './services/bookService';

const books = ref([]);
const loading = ref(false);
const message = ref('');
const showDialog = ref(false);
const isEditMode = ref(false);
const editBookId = ref(null);
const newBook = ref({ title: '', author: '', category: '', available: true });

const fetchBooks = async () => {
  loading.value = true;
  message.value = '';

  try {
    books.value = await bookService.getAllBooks();
  } catch (error) {
    message.value = 'Unable to load books. Please try again.';
  } finally {
    loading.value = false;
  }
};

const openInsertDialog = () => {
  isEditMode.value = false;
  editBookId.value = null;
  message.value = '';
  newBook.value = { title: '', author: '', category: '', available: true };
  showDialog.value = true;
};

const openEditDialog = (book) => {
  isEditMode.value = true;
  editBookId.value = book.id;
  message.value = '';
  newBook.value = {
    title: book.title,
    author: book.author,
    category: book.category,
    available: book.available,
  };
  showDialog.value = true;
};

const closeDialog = () => {
  showDialog.value = false;
  message.value = '';
};

const saveBook = async () => {
  message.value = '';

  try {
    if (isEditMode.value && editBookId.value !== null) {
      await bookService.updateBook(editBookId.value, {
        title: newBook.value.title,
        author: newBook.value.author,
        category: newBook.value.category,
        available: newBook.value.available,
      });
      message.value = 'Book updated successfully.';
    } else {
      await bookService.createBook({
        title: newBook.value.title,
        author: newBook.value.author,
        category: newBook.value.category,
        available: newBook.value.available,
      });
      window.alert('Book created successfully.'); 
      //message.value = 'Book created successfully.';
    }

    await fetchBooks();
    closeDialog();
  } catch (error) {
    message.value = 'Unable to save book. Please try again.';
  }
};

const deleteBook = async (id) => {
  const confirmed = window.confirm('Delete this book?');
  if (!confirmed) return;

  try {
    await bookService.deleteBook(id);
    await fetchBooks();
  } catch (error) {
    message.value = 'Unable to delete book. Please try again.';
  }
};

onMounted(fetchBooks);
</script>

<style scoped>
.books-page {
  padding: 1rem;
}
.books-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}
.books-table {
  width: 100%;
  border-collapse: collapse;
}
.books-table th,
.books-table td {
  border: 1px solid #ddd;
  padding: 0.75rem;
  text-align: left;
}
.actions-cell {
  display: flex;
  gap: 0.5rem;
}
.action-button,
.primary-button {
  border: none;
  padding: 0.5rem 0.75rem;
  cursor: pointer;
}
.primary-button {
  background: #2f80ed;
  color: white;
}
.action-button.danger {
  background: #eb5757;
  color: white;
}
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
  z-index: 100;
}
.dialog-box {
  background: white;
  width: min(520px, 100%);
  border-radius: 0.5rem;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  padding: 1rem;
}
.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}
.close-button {
  background: none;
  border: none;
  font-size: 1.5rem;
  line-height: 1;
  cursor: pointer;
}
.dialog-form label {
  display: block;
  margin-bottom: 0.75rem;
}
.dialog-form input[type='text'],
.dialog-form input[type='checkbox'] {
  width: 100%;
  padding: 0.5rem;
  margin-top: 0.25rem;
  box-sizing: border-box;
}
.checkbox-label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.dialog-actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  margin-top: 1rem;
}
.status-message,
.message {
  margin-top: 1rem;
}
</style>
