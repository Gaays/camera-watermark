<template>
  <div class="content">
    <input type="file" id="fileInput" @change="handleFileChange">
  </div>
</template>

<script setup>
import { ref } from 'vue'

const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();

    reader.onload = function (e) {
      const arrayBuffer = e.target.result;
      const exifData = EXIF.readFromBinaryFile(arrayBuffer);

      // 显示元信息
      document.getElementById('output').textContent = JSON.stringify(exifData, null, 2);
    };

    reader.readAsArrayBuffer(file);
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped></style>
