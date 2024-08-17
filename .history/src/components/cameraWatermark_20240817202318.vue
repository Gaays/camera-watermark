<template>
  <div class="content">
    <input type="file" id="fileInput" accept="image/*" @change="handleFileChange">
    <span>{{ output }}</span>
  </div>
</template>

<script setup>
import { ref } from 'vue'
// import EXIF from 'exif-js'
import sharp from 'sharp'

const output = ref('')
const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();

    reader.onload = function (e) {
      // console.log('🚀 ~ handleFileChange ~ e:', e);
      const arrayBuffer = e.target.result;
      console.log('🚀 ~ handleFileChange ~ arrayBuffer:', arrayBuffer);
      // const exifData = EXIF.readFromBinaryFile(arrayBuffer);
      const exifData = sharp(arrayBuffer)
      console.log('🚀 ~ handleFileChange ~ exifData:', exifData);
      // // 显示元信息
      // output.value = JSON.stringify(exifData, null, 2);
    };

    reader.readAsArrayBuffer(file);
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped></style>
