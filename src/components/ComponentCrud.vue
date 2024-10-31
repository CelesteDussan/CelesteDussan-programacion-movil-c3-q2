<template>
  <ion-page>
    <!-- Menú lateral de navegación -->
    <ion-menu content-id="main-content">
      <ion-header>
        <ion-toolbar>
          <ion-title>Categorías</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
        <ion-list>
          <ion-item button @click="setCategory('Ficción')">Ficción</ion-item>
          <ion-item button @click="setCategory('No Ficción')">No Ficción</ion-item>
          <ion-item button @click="setCategory('Ciencia')">Ciencia</ion-item>
          <ion-item button @click="setCategory('Historia')">Historia</ion-item>
        </ion-list>
        <ion-menu-toggle>
          <ion-button expand="block">Cerrar Menú</ion-button>
        </ion-menu-toggle>
      </ion-content>
    </ion-menu>

    <!-- Contenido principal -->
    <div class="ion-page" id="main-content">
      <ion-header>
        <ion-toolbar>
          <ion-title>Biblioteca</ion-title>
          <ion-buttons slot="end">
            <ion-menu-toggle>
              <ion-button>Menú</ion-button>
            </ion-menu-toggle>
          </ion-buttons>
        </ion-toolbar>
        <ion-toolbar>
          <ion-searchbar v-model="searchQuery" placeholder="Buscar título o autor"></ion-searchbar>
        </ion-toolbar>
      </ion-header>

      <ion-content>
        <!-- Segmento para vista de lista, favoritos y prestados -->
        <ion-segment v-model="currentSegment">
          <ion-segment-button value="todos">Todos</ion-segment-button>
          <ion-segment-button value="prestados">Mis Prestados</ion-segment-button>
          <ion-segment-button value="favoritos">Favoritos</ion-segment-button>
        </ion-segment>

        <!-- Contador de libros -->
        <div class="book-count">
          <p>Total en esta vista: {{ filteredBooks.length }}</p>
        </div>

        <!-- Lista de libros filtrados por categoría y búsqueda -->
        <ion-list>
          <ion-item v-for="(book, index) in filteredBooks" :key="index" button @click="viewBookDetails(book)">
            <ion-label>
              <h2>{{ book.title }}</h2>
              <p>{{ book.author }} - {{ book.category }}</p>
              <!-- Indicador de libro prestado -->
              <p v-if="!book.isAvailable">📕 Libro prestado</p>
            </ion-label>
            <ion-badge :color="book.isAvailable ? 'success' : 'danger'">
              {{ book.isAvailable ? 'Disponible' : 'Prestado' }}
            </ion-badge>
            <ion-button @click.stop="toggleFavorite(book)">
              <ion-icon :name="isFavorite(book) ? 'heart' : 'heart-outline'"></ion-icon>
            </ion-button>
          </ion-item>
        </ion-list>

        <!-- Botón para agregar un nuevo libro -->
        <ion-fab vertical="bottom" horizontal="end" slot="fixed">
          <ion-fab-button @click="showAddBookModal = true">
            <ion-icon name="add"></ion-icon>
          </ion-fab-button>
        </ion-fab>

        <!-- Modal para agregar libro -->
        <ion-modal :isOpen="showAddBookModal" @ionModalDidDismiss="showAddBookModal = false">
          <ion-header>
            <ion-toolbar>
              <ion-title>Nuevo Libro</ion-title>
              <ion-button slot="end" @click="showAddBookModal = false">Cerrar</ion-button>
            </ion-toolbar>
          </ion-header>
          <ion-content>
            <ion-item>
              <ion-label>Título</ion-label>
              <ion-input placeholder="Título" v-model="newBook.title"></ion-input>
            </ion-item>
            <ion-item>
              <ion-label>Autor</ion-label>
              <ion-input placeholder="Autor" v-model="newBook.author"></ion-input>
            </ion-item>
            <ion-item>
              <ion-label>Categoría</ion-label>
              <ion-input placeholder="Categoría" v-model="newBook.category"></ion-input>
            </ion-item>
            <ion-button expand="block" @click="addBook">Agregar Libro</ion-button>
          </ion-content>
        </ion-modal>
      </ion-content>
    </div>
  </ion-page>
</template>

<script setup>
import { ref, computed } from 'vue';
import {
  IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonItem, IonLabel, IonInput,
  IonList, IonSegment, IonSegmentButton, IonFab, IonFabButton, IonModal, IonButton,
  IonBadge, IonMenu, IonMenuToggle, IonSearchbar, IonIcon
} from '@ionic/vue';

const searchQuery = ref('');
const category = ref('');
const currentSegment = ref('todos');
const showBookDetails = ref(false);
const books = ref([
  { title: '1984', author: 'George Orwell', category: 'Ficción', isAvailable: true },
  { title: 'Cien años de soledad', author: 'Gabriel García Márquez', category: 'Ficción', isAvailable: false },
  { title: 'El origen de las especies', author: 'Charles Darwin', category: 'Ciencia', isAvailable: true },
]);
const favoriteBooks = ref([]);
const borrowedBooks = ref([]);
const showAddBookModal = ref(false);
const showToast = ref(false);
const newBook = ref({ title: '', author: '', category: '', isAvailable: true });
const selectedBook = ref(null);

const filteredBooks = computed(() => {
  const filteredList = currentSegment.value === 'favoritos' ? favoriteBooks.value
                   : currentSegment.value === 'prestados' ? borrowedBooks.value
                   : books.value;
  
  return filteredList.filter(book =>
    (category.value === '' || book.category === category.value) &&
    (book.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
     book.author.toLowerCase().includes(searchQuery.value.toLowerCase()))
  );
});

function toggleFavorite(book) {
  const index = favoriteBooks.value.findIndex(favBook => favBook.title === book.title);
  if (index !== -1) {
    favoriteBooks.value.splice(index, 1);
  } else {
    favoriteBooks.value.push(book);
  }
}

function isFavorite(book) {
  return favoriteBooks.value.some(favBook => favBook.title === book.title);
}

function addBook() {
  books.value.push({ ...newBook.value });
  newBook.value = { title: '', author: '', category: '', isAvailable: true };
  showAddBookModal.value = false;
  showToast.value = true;
}

function setCategory(cat) {
  category.value = cat;
}

function viewBookDetails(book) {
  selectedBook.value = book;
  showBookDetails.value = true;
}
</script>

<style scoped>
ion-item {
  margin-bottom: 10px;
}
.book-count {
  padding: 10px;
  text-align: center;
  font-weight: bold;
}
</style>
  