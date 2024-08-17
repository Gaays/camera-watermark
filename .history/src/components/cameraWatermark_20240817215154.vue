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
import CanonImg from '../assets/canon.jpg'

const borderInfo = {
  left: 20,
  top: 20,
  right: 20,
  bottom: 80,
}

const markMap = {
  Canon: CanonImg
}

const output = ref([
  {
    name: '相机品牌', key: 'Make', value: ''
  },
  {
    name: '相机型号', key: 'Model', value: ''
  },
  {
    name: '原始拍摄时间', key: 'DateTimeOriginal', value: ''
  },
  {
    name: '快门速度', key: 'ExposureTime', value: ''
  },
  {
    name: '光圈', key: 'FNumber', value: ''
  },
  {
    name: 'ISO', key: 'ISOSpeedRatings', value: ''
  },
  {
    name: '焦段', key: 'FocalLength', value: ''
  },
])

const handleExif = (arrayBuffer) => {
  const exifData = EXIF.readFromBinaryFile(arrayBuffer);
  output.value.forEach(item => {
    if (exifData[item.key]) {
      item.value = exifData[item.key];
      // 单独处理快门速度，把值改为分数显示
      if (item.key === 'ExposureTime') {
        const data = exifData[item.key];
        if (data > 1) {
          item.value = data + 's'
        } else {
          item.value = '1/' + 1 / data + 's'
        }
      } else if (item.key === 'FNumber') {
        // 单独处理光圈
        const data = exifData[item.key];
        item.value = 'f/' + data
      }
    }
  })
}

const handleImg = (file) => {
  const originImg = URL.createObjectURL(file);
  const img = new Image();
  img.onload = function () {
    const canvas = document.getElementById('imageCanvas');
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, imgBox.value.width, imgBox.value.height);
    const aspectRatio = img.width / img.height;
    let imgWidth = 0
    let imgHeight = 0
    if (img.width >= img.height) {
      imgWidth = imgBox.value.width;
      imgHeight = imgWidth / aspectRatio;
    } else {
      imgHeight = imgBox.value.height;
      imgWidth = imgHeight * aspectRatio;
    }

    // 将图片绘制到 canvas 上
    ctx.drawImage(img, borderInfo.left, borderInfo.top, imgWidth, imgHeight);
  };
  img.src = originImg; // 设置图片源为读取的文件数据
}

const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();

    reader.onload = function (e) {
      const arrayBuffer = e.target.result;
      // 读exif数据
      handleExif(arrayBuffer)

      // canvas渲染图片
      handleImg(file)

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
    height: domSizeData[0].height,
  }
  const canvas = document.getElementById('imageCanvas');
  canvas.width = imgBox.value.width + borderInfo.left + borderInfo.right;
  canvas.height = imgBox.value.height + borderInfo.top + borderInfo.bottom;
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
