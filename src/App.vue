<template>
  <div class="container mt-5">
    <h1 class="text-center mb-4">📚 Моя коллекция книг</h1>
    
    <!-- Форма добавления -->
    <div class="row">
      <div class="col-md-8 offset-md-2">
        <form @submit.prevent="addBook" class="card p-4 shadow-sm">
          <div class="mb-3">
            <label class="form-label">Название книги *</label>
            <input v-model="newBook.title" type="text" class="form-control" required placeholder="Введите название">
          </div>
          <div class="mb-3">
            <label class="form-label">Описание</label>
            <textarea v-model="newBook.description" class="form-control" rows="3" placeholder="Краткое описание"></textarea>
          </div>
          <div class="mb-3">
            <label class="form-label">URL изображения</label>
            <input v-model="newBook.image" type="url" class="form-control" placeholder="https://example.com/image.jpg">
          </div>
          
          <!-- Новые поля: Статусы -->
          <div class="row mb-3">
            <div class="col-6">
              <div class="form-check">
                <input v-model="newBook.isFavorite" class="form-check-input" type="checkbox" id="favCheck">
                <label class="form-check-label" for="favCheck">
                  ❤️ В избранное
                </label>
              </div>
            </div>
            <div class="col-6">
              <div class="form-check">
                <input v-model="newBook.isRead" class="form-check-input" type="checkbox" id="readCheck">
                <label class="form-check-label" for="readCheck">
                  ✅ Уже прочитано
                </label>
              </div>
            </div>
          </div>

          <button class="btn btn-primary w-100" type="submit">
            <i class="bi bi-plus-circle"></i> Добавить книгу
          </button>
        </form>
      </div>
    </div>

    <!-- Панель управления -->
    <div class="row mt-4">
      <div class="col-md-6">
        <input v-model="searchQuery" type="text" class="form-control" placeholder="🔍 Поиск по названию...">
      </div>
      <div class="col-md-6">
        <select v-model="sortBy" class="form-select">
          <option value="newest">Сначала новые</option>
          <option value="oldest">Сначала старые</option>
          <option value="alpha">По алфавиту</option>
          <option value="favorites">Сначала избранные</option>
        </select>
      </div>
    </div>

    <!-- Сетка карточек -->
    <div class="row mt-4">
      <div v-for="book in filteredAndSortedBooks" :key="book.id" class="col-md-4 mb-4">
        <div class="card h-100 shadow-sm" :class="{ 'border-warning': book.isFavorite }">
          <!-- Изображение -->
          <img 
            v-if="book.image" 
            :src="book.image" 
            class="card-img-top" 
            :alt="book.title"
            style="height: 200px; object-fit: cover;"
            @error="e => e.target.style.display = 'none'"
          >
          <div class="card-body d-flex flex-column">
            <!-- Заголовок и иконки действий -->
            <div class="d-flex justify-content-between align-items-start mb-2">
              <h5 class="card-title mb-0">{{ book.title }}</h5>
              <div>
                <!-- Кнопка Избранное -->
                <button 
                  @click="toggleFavorite(book.id)" 
                  class="btn btn-link p-0 me-2" 
                  :class="book.isFavorite ? 'text-danger' : 'text-muted'"
                  title="В избранное"
                >
                  <i :class="book.isFavorite ? 'bi bi-heart-fill' : 'bi bi-heart'"></i>
                </button>
                <!-- Кнопка Прочитано -->
                <button 
                  @click="toggleRead(book.id)" 
                  class="btn btn-link p-0" 
                  :class="book.isRead ? 'text-success' : 'text-muted'"
                  title="Отметить как прочитанное"
                >
                  <i :class="book.isRead ? 'bi bi-check-circle-fill' : 'bi bi-circle'"></i>
                </button>
              </div>
            </div>
            
            <p class="card-text text-muted small" v-if="book.description">
              {{ book.description }}
            </p>
            
            <!-- Бейджи статусов -->
            <div class="mb-3">
              <span v-if="book.isRead" class="badge bg-success me-1">Прочитано</span>
              <span v-if="book.isFavorite" class="badge bg-warning text-dark">Избранное</span>
            </div>

            <div class="mt-auto">
              <button @click="removeBook(book.id)" class="btn btn-outline-danger btn-sm w-100">
                <i class="bi bi-trash"></i> Удалить
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Сообщение, если коллекция пуста -->
    <div v-if="filteredAndSortedBooks.length === 0" class="text-center text-muted mt-5">
      <p class="fs-5">📭 Коллекция пуста. Добавьте первую книгу!</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const books = ref([])
// Инициализация новых полей в форме
const newBook = ref({ 
  title: '', 
  description: '', 
  image: '',
  isFavorite: false,
  isRead: false
})
const searchQuery = ref('')
const sortBy = ref('newest')

// Добавление книги
const addBook = () => {
  if (!newBook.value.title.trim()) return
  
  books.value.push({
    id: Date.now(),
    title: newBook.value.title.trim(),
    description: newBook.value.description.trim(),
    image: newBook.value.image.trim(),
    isFavorite: newBook.value.isFavorite,
    isRead: newBook.value.isRead,
    createdAt: new Date()
  })
  
  // Сброс формы (включая чекбоксы)
  newBook.value = { title: '', description: '', image: '', isFavorite: false, isRead: false }
}

// Удаление книги
const removeBook = (id) => {
  books.value = books.value.filter(book => book.id !== id)
}

// Переключение статуса "Избранное"
const toggleFavorite = (id) => {
  const book = books.value.find(b => b.id === id)
  if (book) book.isFavorite = !book.isFavorite
}

// Переключение статуса "Прочитано"
const toggleRead = (id) => {
  const book = books.value.find(b => b.id === id)
  if (book) book.isRead = !book.isRead
}

// Вычисляемое свойство: фильтрация + сортировка
const filteredAndSortedBooks = computed(() => {
  let result = [...books.value]
  
  // Фильтрация по поиску
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(book => 
      book.title.toLowerCase().includes(query) ||
      book.description?.toLowerCase().includes(query)
    )
  }
  
  // Сортировка
  switch (sortBy.value) {
    case 'newest':
      result.sort((a, b) => b.createdAt - a.createdAt)
      break
    case 'oldest':
      result.sort((a, b) => a.createdAt - b.createdAt)
      break
    case 'alpha':
      result.sort((a, b) => a.title.localeCompare(b.title))
      break
    case 'favorites':
      // Сначала избранные, потом остальные
      result.sort((a, b) => (b.isFavorite === a.isFavorite) ? 0 : b.isFavorite ? 1 : -1)
      break
  }
  
  return result
})
</script>

<style scoped>
.card {
  transition: transform 0.2s, box-shadow 0.2s;
}
.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.1) !important;
}
/* Убираем подчеркивание у кнопок-иконок */
.btn-link {
  text-decoration: none;
}
.btn-link:hover {
  text-decoration: none;
}
</style>