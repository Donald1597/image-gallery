<template>
  <div
    class="container mx-auto p-4 bg-gradient-to-r from-gray-100 to-gray-200 min-h-screen"
  >
    <div
      @dragover.prevent="dragoverHandler"
      @dragleave.prevent="dragleaveHandler"
      @drop.prevent="dropHandler"
      class="drop-area mb-6 border-4 border-dashed border-gray-300 p-6 text-center bg-white rounded-lg shadow-lg transition-transform duration-300 hover:shadow-2xl"
    >
      <p class="mb-2 text-gray-600 text-lg font-semibold">
        Drag & Drop Images Here or
      </p>
      <input type="file" @change="uploadImage" multiple class="hidden" />
      <button
        @click.prevent="openFileDialog"
        class="bg-blue-600 text-white py-2 px-6 rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-transform duration-300 hover:scale-105"
      >
        Select Images
      </button>
    </div>
    <div class="flex flex-col md:flex-row md:justify-between mb-6">
      <input
        type="text"
        v-model="searchQuery"
        placeholder="Search by title..."
        class="border border-gray-300 p-3 rounded-md w-full md:w-1/3 text-gray-700 shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-500 focus:ring-opacity-50 transition-transform duration-300"
      />
      <select
        v-model="sortBy"
        class="border border-gray-300 p-3 rounded-md mt-4 md:mt-0 md:ml-4 text-gray-700 shadow-sm transition-transform duration-300"
      >
        <option value="date">Sort by Date</option>
        <option value="title">Sort by Title</option>
      </select>
    </div>
    <div
      class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6"
    >
      <div
        v-for="image in filteredAndSortedImages"
        :key="image.id"
        class="relative group bg-white rounded-lg shadow-lg overflow-hidden transition-transform duration-300 hover:scale-105 hover:shadow-xl"
      >
        <img
          :src="image.url"
          @click="viewImage(image)"
          class="cursor-pointer object-cover w-full h-[20rem] transition-transform duration-300"
          :style="{
            transform: `rotate(${image.rotate}deg)`,
          }"
        />
        <div
          class="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black to-transparent text-white p-3 flex items-center justify-between"
        >
          <input
            v-model="imageEdit[image.id]"
            @blur="updateTitle(image.id)"
            placeholder="Add a title"
            class="bg-transparent border-none w-full text-white focus:outline-none"
          />
          <div class="flex space-x-2">
            <button
              @click.stop="rotateImage(image)"
              class="bg-blue-600 text-white py-1 px-2 rounded-lg hover:bg-blue-700 transition-transform duration-300"
              title="Rotate"
            >
              🔄
            </button>
            <button
              @click.stop="deleteImage(image.id)"
              class="bg-red-600 text-white py-1 px-2 rounded-lg hover:bg-red-700 transition-transform duration-300"
              title="Delete"
            >
              🗑️
            </button>
          </div>
        </div>
      </div>
    </div>
    <div
      v-if="selectedImage"
      class="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center z-50"
    >
      <div
        class="bg-white p-6 max-w-screen-lg mx-auto rounded-lg relative shadow-2xl"
      >
        <img :src="selectedImage.url" class="w-full h-[30rem] object-contain" />
        <p class="text-center mt-3 text-lg font-semibold">
          {{ selectedImage.title }}
        </p>
        <button
          @click="closeImage"
          class="absolute top-4 right-4 bg-red-600 text-white py-2 px-4 rounded-lg hover:bg-red-700 transition-transform duration-300"
        >
          Close
        </button>
      </div>
    </div>
    <div class="flex justify-center mt-6">
      <button
        v-for="page in totalPages"
        :key="page"
        @click="currentPage = page"
        class="mx-1 px-4 py-2 border border-gray-300 rounded-lg shadow-md transition-colors duration-300"
        :class="{
          'bg-blue-600 text-white': currentPage === page,
          'bg-white text-gray-700': currentPage !== page,
        }"
      >
        {{ page }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";
import { initialImages } from "@/data/imageData";

const images = ref([...initialImages]);
const selectedImage = ref(null);
const searchQuery = ref("");
const sortBy = ref("title");
const currentPage = ref(1);
const itemsPerPage = 12;
const imageEdit = ref({});

// Initialize imageEdit for existing images
images.value.forEach((image) => {
  imageEdit.value[image.id] = image.title;
});

const openFileDialog = () => {
  const input = document.querySelector('input[type="file"]');
  if (input) {
    input.click();
  }
};

const uploadImage = (event) => {
  const files = event.target.files;
  for (let i = 0; i < files.length; i++) {
    const reader = new FileReader();
    reader.onload = (e) => {
      images.value.push({
        id: Date.now() + i,
        url: e.target.result,
        title: "",
        rotate: 0,
      });
      // Initialize title in local state
      imageEdit.value[Date.now() + i] = "";
    };
    reader.readAsDataURL(files[i]);
  }
};

const updateTitle = (imageId) => {
  const image = images.value.find((img) => img.id === imageId);
  if (image) {
    image.title = imageEdit.value[imageId] || "";
  }
};

const deleteImage = (id) => {
  images.value = images.value.filter((image) => image.id !== id);
  delete imageEdit.value[id]; // Remove from local state
};

const viewImage = (image) => {
  selectedImage.value = image;
};

const closeImage = () => {
  selectedImage.value = null;
};

const dragoverHandler = (event) => {
  event.preventDefault();
  event.currentTarget.classList.add("dragover");
};

const dragleaveHandler = (event) => {
  event.preventDefault();
  event.currentTarget.classList.remove("dragover");
};

const dropHandler = (event) => {
  event.preventDefault();
  const files = event.dataTransfer.files;
  for (let i = 0; i < files.length; i++) {
    const reader = new FileReader();
    reader.onload = (e) => {
      images.value.push({
        id: Date.now() + i,
        url: e.target.result,
        title: "",
        rotate: 0,
      });
      // Initialize title in local state
      imageEdit.value[Date.now() + i] = "";
    };
    reader.readAsDataURL(files[i]);
  }
  event.currentTarget.classList.remove("dragover");
};

const rotateImage = (image) => {
  image.rotate = (image.rotate + 90) % 360;
};

// Filter images based on search query
const filteredImages = computed(() => {
  return images.value.filter((image) =>
    image.title.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

// Sort images based on sortBy value
const sortedImages = computed(() => {
  return filteredImages.value.slice().sort((a, b) => {
    if (sortBy.value === "title") {
      return a.title.localeCompare(b.title);
    } else {
      return b.id - a.id;
    }
  });
});

// Pagination logic
const totalPages = computed(() => {
  return Math.ceil(sortedImages.value.length / itemsPerPage);
});

const paginatedImages = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return sortedImages.value.slice(start, end);
});

// Update filtered and sorted images based on current page and sortBy
const filteredAndSortedImages = computed(() => {
  return paginatedImages.value;
});

// Watcher to react to changes in sortBy
watch(
  [sortBy, filteredImages],
  () => {
    filteredAndSortedImages.value = paginatedImages.value;
  },
  { deep: true }
);
</script>

<style scoped>
.drop-area {
  border-color: #d1d5db;
  background-color: #f9fafb;
}

.drop-area.dragover {
  border-color: #3b82f6;
}

.drop-area.dragover p {
  color: #3b82f6;
}

img {
  transition: transform 0.3s ease, width 0.3s ease, height 0.3s ease;
}

img:hover {
  filter: brightness(0.9);
}

button {
  cursor: pointer;
}

button:hover {
  opacity: 0.9;
}

button[title] {
  position: relative;
}

button[title]::after {
  content: attr(title);
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: black;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  white-space: nowrap;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.2s ease;
}

button[title]:hover::after {
  opacity: 1;
  visibility: visible;
}
</style>
