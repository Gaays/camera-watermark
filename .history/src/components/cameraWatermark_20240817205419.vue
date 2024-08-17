<template>
  <div class="content">
    <input type="file" id="fileInput" accept="image/*" @change="handleFileChange">
    <div class="show-box">
      <canvas id="imageCanvas" style="border: 1px solid black;"></canvas>
      <div class="data-box">
        <span>{{ output }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import EXIF from 'exif-js'

const output = ref('')
// const imgSrc = ref()
const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();

    reader.onload = function (e) {
      const arrayBuffer = e.target.result;
      const originImg = URL.createObjectURL(file);
      // 读exif数据
      const exifData = EXIF.readFromBinaryFile(arrayBuffer);
      output.value = JSON.stringify(exifData, null, 2);

      // canvas渲染图片
      const img = new Image();
      img.onload = function () {
        const canvas = document.getElementById('imageCanvas');
        const ctx = canvas.getContext('2d');

        // 设置 canvas 大小为图片大小
        canvas.width = img.width;
        canvas.height = img.height;

        // 将图片绘制到 canvas 上
        ctx.drawImage(img, 0, 0, img.width, img.height);
      };
      img.src = originImg; // 设置图片源为读取的文件数据
    };

    reader.readAsArrayBuffer(file);
  }
}

const imgBox = ref({
  width: 0,
  height: 0
})
const initData = () => {
  const dom = document.getElementById('imageCanvas');
  const domSizeData = dom.getClientRects();
  imgBox.value = {
    width: domSizeData[0].width,
    height: domSizeData[0].height
  }
}

onMounted(() => {
  initData()
})
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

    #imageCanvas {
      width: calc(100% - 300px);
      height: 100%;
    }

    .data-box {
      width: 300px;
      height: 100%;
      overflow: auto;
    }
  }
}
</style>
