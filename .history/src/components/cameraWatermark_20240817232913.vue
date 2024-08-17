<template>
  <div class="content">
    <input type="file" id="fileInput" accept="image/*" @change="handleFileChange">
    <div class="show-box">
      <canvas id="imageCanvas" style="border: 1px solid black;"></canvas>
      <div class="data-box">
        <span>{{ output }}</span>
        <div class="btn-box">
          <button @click="handleDownload">下载</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import EXIF from 'exif-js'
import CanonImg from '../assets/canon.png'
import NikonImg from '../assets/nikon.png'
import SonyImg from '../assets/sony.png'

const borderInfo = {
  left: 20,
  top: 20,
  right: 20,
  bottom: 80,
}

const markMap = {
  Canon: CanonImg,
  Nikon: NikonImg,
  Sony: SonyImg
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
      const data = exifData[item.key];
      item.value = data;
      // 单独处理快门速度，把值改为分数显示
      if (item.key === 'ExposureTime') {
        if (data > 1) {
          item.value = data + 's'
        } else {
          item.value = '1/' + 1 / data + 's'
        }
      } else if (item.key === 'FNumber') {
        // 单独处理光圈
        item.value = 'f/' + data
      } else if (item.key === 'FocalLength') {
        // 单独处理焦段
        item.value = data + 'mm'
      } else if (item.key === 'ISOSpeedRatings') {
        item.value = 'ISO ' + data
      }
    } else {
      item.value = '-'
    }
  })
}

let imgInfo = {
  fileName: '',
  src: null,
  width: 0,
  height: 0
}
const handleImg = async (file) => {
  const originImg = URL.createObjectURL(file);
  const canvas = document.getElementById('imageCanvas');
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, canvasBox.width, canvasBox.height);

  const img = new Image();
  img.onload = function () {
    // 获取图片宽高
    imgInfo.width = img.width
    imgInfo.height = img.height
    imgInfo.src = originImg

    const aspectRatio = img.width / img.height;
    let imgWidth = 0
    let imgHeight = 0
    if (img.width >= img.height) {
      imgWidth = imgBox.width;
      imgHeight = imgWidth / aspectRatio;
    } else {
      imgHeight = imgBox.height;
      imgWidth = imgHeight * aspectRatio;
    }
    ctx.strokeRect(0, 0, imgWidth + borderInfo.left + borderInfo.right, imgHeight + borderInfo.top + borderInfo.bottom);
    // 将图片绘制到 canvas 上
    ctx.drawImage(img, borderInfo.left, borderInfo.top, imgWidth, imgHeight);

    // 绘制水印
    handleDrawExif(ctx, imgWidth, imgHeight);
  };
  img.src = originImg; // 设置图片源为读取的文件数据
}

const handleDrawExif = async (ctx, width, height) => {
  ctx.font = "18px bold Arial";
  ctx.fillStyle = "rgb(0, 0, 0)";
  // 品牌
  const brand = output.value[0].value;
  // 快门速度
  const exposureTime = output.value[3].value;
  // 光圈
  const fNumber = output.value[4].value;
  // ISO
  const iso = output.value[5].value;
  // 焦段
  const focalLength = output.value[6].value;

  const list = [focalLength, fNumber, exposureTime, iso]
  const text = list.join(' ')
  const textWidth = ctx.measureText(text).width;
  const textLeft = (width - textWidth + borderInfo.left);
  const textTop = (height + borderInfo.top + 50);

  ctx.fillText(text, textLeft, textTop)

  // 绘制品牌
  await drawBrand(brand, ctx, width, height)
}

// 绘制品牌
const drawBrand = async (brand, ctx, width, height) => {
  if (brand && markMap[brand]) {
    const originImg = markMap[brand];
    const img = new Image();
    img.onload = function () {
      const aspectRatio = img.width / img.height;
      const imgHeight = 30
      const imgWidth = imgHeight * aspectRatio
      const imgTop = height + borderInfo.top + 20
      const imgLeft = borderInfo.left

      ctx.drawImage(img, imgLeft, imgTop, imgWidth, imgHeight);
    };
    img.src = originImg; // 设置图片源为读取的文件数据
  }
}

const handleFileChange = (event) => {
  const file = event.target.files[0];
  console.log('🚀 ~ handleFileChange ~ file:', file);
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

let imgBox = {
  width: 0,
  height: 0
}
let canvasBox = {
  width: 0,
  height: 0
}
const initData = () => {
  const dom = document.getElementById('imageCanvas');
  const domSizeData = dom.getClientRects();
  imgBox = {
    width: domSizeData[0].width,
    height: domSizeData[0].height,
  }
  const canvas = document.getElementById('imageCanvas');
  canvas.width = imgBox.width + borderInfo.left + borderInfo.right;
  canvas.height = imgBox.height + borderInfo.top + borderInfo.bottom;
  canvasBox = {
    width: canvas.width,
    height: canvas.height,
  }
}

// 后台创建canvas尺寸和图片尺寸一致
const handleDownload = async () => {
  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d');

  const img = new Image();
  img.onload = function () {
    canvas.width = imgInfo.width + borderInfo.left + borderInfo.right;
    canvas.height = imgInfo.height + borderInfo.top + borderInfo.bottom;
    // 将图片绘制到 canvas 上
    ctx.drawImage(img, borderInfo.left, borderInfo.top, imgInfo.width, imgInfo.height);

    // 绘制水印
    handleDrawExif(ctx, imgInfo.width, imgInfo.height);
  };
  img.src = imgInfo.src; // 设置图片源为读取的文件数据

  // 下载图片
  const jpg = canvas.toDataURL('image/jpeg', 1);
  const link = document.createElement('a');
  link.href = jpg;
  link.download = '';
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
