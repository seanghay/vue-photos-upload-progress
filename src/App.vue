<script setup>
import SuspensePhotoItem from "./SuspensePhotoItem.vue";
import { ref } from "vue";
import axios from "axios";
const filesList = ref([]);

const buttonText = ref("Disabled");
const loading = ref(false);
const progress = ref(0)

async function fileUpload(file, progress) {
  let valueUp = 0;
  let valueDown = 0;

  const value = () => Math.floor((valueDown + valueUp) * 100);

  progress(value());
  const form = new FormData();
  form.append("file", file);
  const response = await axios.post("https://httpbin.org/post", form, {
    headers: {
      "Content-Type": "multipart/form-data",
    },
    onDownloadProgress: (e) => {
      valueDown = (e.loaded / e.total) * 0.5;
      progress(value());
    },
    onUploadProgress: (e) => {
      valueUp = (e.loaded / e.total) * 0.5;
      progress(value());
    },
  });

  return response.data;
}

async function upload() {
  loading.value = true;
  buttonText.value = "Uploading...";
  const files = Array.from(filesList.value);
  let size = files.length;

  let i = 0;
  try {
    for (const file of files) {
      i++;
      buttonText.value = `Uploading (${i}/${size})`;
      await fileUpload(file, (p) => {
        progress.value = p;
        buttonText.value = `Uploading ${p}% (${i}/${size})`;
      });
      filesList.value = [...filesList.value.filter((it) => it !== file)];
    }
  } catch (e) {
    console.error(e);
  }

  buttonText.value = "Disabled";
  loading.value = false;

  filesList.value = [];
}
</script>

<template>
  <div class="flex-column">
    <h2>Photos Upload</h2>
    <p class="subtitle">Choose your photos or drop</p>
    <input
      multiple
      accept="image/*"
      type="file"
      name="files"
      id="files"
      @change="(el) => (filesList = [...el.target.files])"
    />
    <div class="files-list">
      <SuspensePhotoItem
        v-for="(file, index) in filesList"
        :file="file"
        :key="index"
      />
    </div>
    <button
      @click="upload"
      :disabled="filesList.length == 0 || loading"
      class="button"
    >
      <template v-if="filesList.length && !loading"> Upload </template>
      <template v-else>{{ buttonText }}</template>
    </button>
    <progress v-if="loading" max="100" :value="progress"></progress>
  </div>
</template>

<style lang="scss">
body {
  color: white;
  background-color: rgb(21, 21, 21);
  font-size: 14px;
  display: grid;
  place-items: center;
  min-height: 100vh;
}

#app {
  padding: 1.8em 2.4em;
  border: 1px solid rgba(white, 0.12);
  background-color: rgba(white, 0.1);
  max-width: 512px;
  min-width: 512px;

  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
}

h1,
h2,
h3,
h4,
h5,
h6,
p {
  margin: 0;
  padding: 0;
}

.subtitle {
  color: rgba(white, 0.7);
}

.flex-column {
  display: flex;
  flex-direction: column;
  gap: 0.8em;
}

.files-list {
  margin: 1em 0;
  display: flex;
  flex-direction: column;
  gap: 0.8em;
}

.button {
  background-color: rgba(white, 0.1);
  font-family: inherit;
  color: white;
  border: none;
  padding: 0.8em 1em;
  font-weight: bold;
  border: 1px solid rgba(white, 0.2);

  &:not(:disabled):hover {
    background-color: rgba(white, 0.12);
  }

  &:not(:disabled):active {
    background-color: rgba(white, 0.14);
  }

  &:disabled {
    border: 1px solid rgba(white, 0.1);
    color: rgba(white, 0.6);
    cursor: not-allowed;
  }
}
</style>
