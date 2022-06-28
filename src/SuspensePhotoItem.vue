<script setup>
import niceBytes from "nice-bytes";
import { computed, ref } from "vue";

const props = defineProps({ file: File });
const filename = computed(() => props.file.name);
const filesize = computed(() => niceBytes(props.file.size).text);

</script>


<template>
  <div class="photo-item">
    <h3 class="filename">{{ filename }}</h3>
    <small style="opacity: 0.6">{{ filesize }}</small>
  </div>
</template>

<style lang="scss">
.photo-item {
  border: 1px solid rgba(white, 0.1);
  padding: 0.6em 0.8em;
  background-color: rgba(white, 0.05);
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.filename {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  max-width: 500px;
}
</style>
