<script setup>
import { ref, reactive } from 'vue';
import axios from 'axios';

const selectedFiles = reactive([]);
const isDragging = ref(false);
const activeIndex = ref(0);
const fileInput = ref(null);

const handleFileUpload = (event, index) => {
  const file = event.target.files[0];
  if (index === undefined) {
    addNewFile(file);
  } else {
    selectedFiles[index].file = file;
    selectedFiles[index].imagePreview = URL.createObjectURL(file);
    selectedFiles[index].uploaded = false;
    selectedFiles[index].predictionMessage = '';
  }
};

const handleDragOver = (event) => {
  event.preventDefault();
  isDragging.value = true;
};

const handleDragLeave = (event) => {
  event.preventDefault();
  isDragging.value = false;
};

const handleDrop = (event) => {
  event.preventDefault();
  isDragging.value = false;
  const files = event.dataTransfer.files;
  if (files.length > 0) {
    addNewFile(files[0]);
  }
};

const handleNewFileSelection = (event) => {
  const file = event.target.files[0];
  if (file) {
    selectedFiles.push({
      file,
      imagePreview: URL.createObjectURL(file),
      uploaded: false,
      isScanning: false,
      predictionMessage: '',
    });
    activeIndex.value = selectedFiles.length - 1;
  }
};

const addNewFile = (file = null) => {
  selectedFiles.push({
    file,
    imagePreview: file ? URL.createObjectURL(file) : null,
    uploaded: false,
    isScanning: false,
    predictionMessage: '',
  });
  activeIndex.value = selectedFiles.length - 1;
};

const uploadImage = async (index) => {
  const selectedFile = selectedFiles[index];
  if (!selectedFile.file) {
    return;
  }

  const formData = new FormData();
  formData.append('image', selectedFile.file);

  try {
    selectedFile.isScanning = true;
    // const response = await axios.post('/api/upload', formData, {
    //   headers: {
    //     'Content-Type': 'multipart/form-data'
    //   }
    // });
    // if (response.status !== 200) {
    //   throw new Error(`HTTP error! status: ${response.status}`);
    // }

    // Simulating API response
    await new Promise(resolve => setTimeout(resolve, 2000));

    selectedFile.predictionMessage = `Negative 85% sure`;
    selectedFile.uploaded = true;
  } catch (error) {
    console.error('Error uploading image:', error);
  } finally {
    selectedFile.isScanning = false;
  }
};

const resetFileInput = (index) => {
  selectedFiles[index].file = null;
  selectedFiles[index].imagePreview = null;
  selectedFiles[index].predictionMessage = '';
  selectedFiles[index].uploaded = false;
};

const deleteFile = (index) => {
  selectedFiles.splice(index, 1);
}

</script>

<template>
  <div>
    <div v-if="selectedFiles.length === 0" class="drop-zone" @dragover="handleDragOver" @dragleave="handleDragLeave"
      @drop="handleDrop" :class="{ 'dragging': isDragging }">
      <label for="file-input-main" class="drop-zone-label">
        <p>Drag and drop an image here, or click to select a file</p>
      </label>
      <input id="file-input-main" type="file" @change="(event) => handleFileUpload(event)" accept="image/*">
    </div>

    <div v-if="selectedFiles.length > 0" class="image-carousel">
      <div class="tabs">
        <button v-for="(file, index) in selectedFiles" :key="index" @click="activeIndex = index"
          :class="{ active: activeIndex === index }">
          <img :src="selectedFiles[index].imagePreview" alt="Uploaded image"
          style="width: 50px; height: 50px;">
        </button>
      </div>

      <div class="image-details">
        <div class="image-preview" v-if="selectedFiles[activeIndex].imagePreview">
          <div class="image-container" :class="{ 'scanning': selectedFiles[activeIndex].isScanning }">
            <img :src="selectedFiles[activeIndex].imagePreview" alt="Uploaded image"
              style="max-width: 300px; max-height: 200px;">
            <div class="scan-line" v-if="selectedFiles[activeIndex].isScanning"></div>
          </div>
          <div v-if="selectedFiles[activeIndex].predictionMessage" class="prediction-message">
            {{ selectedFiles[activeIndex].predictionMessage }}
          </div>
        </div>
        <div class="buttons">
          <button v-if="!selectedFiles[activeIndex].uploaded" @click="uploadImage(activeIndex)"
            :disabled="!selectedFiles[activeIndex].file">Upload</button>
          <button v-else @click="resetFileInput(activeIndex)">Redo</button>
          <label :for="'file-input-' + activeIndex" class="add-icon">
            <p>+</p>
          </label>
          <input :id="'file-input-' + activeIndex" type="file" @change="(event) => handleFileUpload(event, activeIndex)"
            accept="image/*" style="display: none;">
          <button @click="deleteFile(activeIndex)">Delete</button>
        </div>
      </div>
    </div>

    <div v-if="selectedFiles.length > 0" class="plus-button">
      <label for="file-input-main" class="drop-zone-label">
        <p>+</p>
      </label>
      <input id="file-input-main" type="file" @change="(event) => handleFileUpload(event)" accept="image/*"
        style="display: none;">
    </div>
  </div>
</template>

<style scoped>
.drop-zone {
  width: 300px;
  height: 200px;
  border: 2px dashed #ccc;
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  cursor: pointer;
  transition: border .3s ease;
}

.drop-zone.dragging {
  border-color: #000;
}

.drop-zone input[type="file"] {
  display: none;
}

.drop-zone-label {
  cursor: pointer;
}

.drop-zone p {
  margin: 0;
  text-align: center;
}

.image-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-top: 20px;
}

.image-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.image-preview {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.image-scanning {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.scan-line {
  position: absolute;
  width: 100%;
  height: 2px;
  background: rgba(255, 0, 0, 0.5);
  animation: line-up 3s linear infinite;
}

.scanning .scan-line {
  animation: line-up 3s linear;
}

.buttons {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.add-icon {
  cursor: pointer;
}

.prediction-message {
  margin-top: 10px;
  animation: show-fade 2s ease-in-out;
}

.image-carousel {
  margin-top: 20px;
}

.tabs {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.tabs button {
  padding: 5px 10px;
  border: 1px solid #ccc;
  background: #f0f0f0;
  cursor: pointer;
}

.tabs button.active {
  background: #007bff;
  color: white;
}

.image-details {
  border: 1px solid #ccc;
  padding: 20px;
  border-radius: 5px;
}

.add-new {
  margin-top: 20px;
}

@keyframes line-up {
  0% {
    top: 200px;
    opacity: 0;
  }

  10% {
    top: 200px;
    opacity: 1;
  }

  85% {
    top: -5px;
    opacity: 1;
  }

  100% {
    opacity: 0;
  }
}

@keyframes show-fade {
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
}
</style>
