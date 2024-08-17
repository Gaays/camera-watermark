<template>
  <div class="content">
    <input type="file" id="fileInput" accept="image/*" @change="handleFileChange">
    <div class="show-box">
      <div class="img-box">
        <img v-if="imgSrc" :src="imgSrc">
      </div>
      <div class="data-box">
        <span>{{ output }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import EXIF from 'exif-js'

const output = ref('')
const imgSrc = ref()
const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();

    reader.onload = function (e) {
      const arrayBuffer = e.target.result;
      imgSrc.value = URL.createObjectURL(file);
      const exifData = EXIF.readFromBinaryFile(arrayBuffer);
      // // 显示元信息
      output.value = JSON.stringify(exifData, null, 2);
    };

    reader.readAsArrayBuffer(file);
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.content {
  width: 100%;
  height: 100%;
  padding: 30px 30px;

  .show-box {
    width: 100%;
    height: calc(100% - 25px);
    display: flex;

    .img-box {
      width: calc(100% - 300px);
      height: 100%;

      img {
        width: 100%;
        height: auto;
      }
    }

    .data-box {
      width: 300px;
      height: 100%;
      overflow: auto;
    }
  }
}
</style>
