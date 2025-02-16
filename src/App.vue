<template>
  <div class="container">
    <div class="image-wrapper">
      <div>
        <canvas ref="canvas" id="canvas" width="300" height="300"/>
        <el-slider class="y-axis" v-model="yAxis" :reverse="true" :min="0" :max="300" height="300px" vertical
                   @input="refresh"/>
      </div>
      <el-slider class="x-axis" v-model="xAxis" :min="0" :max="300" @input="refresh"/>
    </div>

    <div>
      <el-button class="pick" size="large" type="primary" @click="download">下载图片</el-button>
      <el-popover placement="right" :width="420" trigger="click" ref="pickerPopover">
        <template #reference>
          <el-button type="primary" size="large" style="margin-right: 16px">选择图片</el-button>
        </template>
        <el-input v-model="searchText" style="width: 240px" placeholder="Please Input" :suffix-icon="Search"
                  @input="searchTabs"/>
        <el-tabs class="picker-tab" v-model="activeTab">
          <el-tab-pane v-for="tab in tabs" :label="tab" :name="tab" :key="tab.toLowerCase()">
            <div class="picker-container">
              <el-image v-if="tab === 'All'" class="picker-item" v-for="item in Array.from(imgDir.values()).flat()"
                        :src="item" alt="img"
                        @click="pickerImg(item)"/>
              <el-image v-else class="picker-item" v-for="item in imgDir.get(tab)" :src="item" alt="img"
                        @click="pickerImg(item)"/>
            </div>
          </el-tab-pane>
        </el-tabs>
      </el-popover>
    </div>

    <div class="controls">
      <div class="slider-block">
        <span class="slider-text">旋转</span>
        <el-slider v-model="rotate" :min="0" :max="360" @input="refresh"/>
      </div>

      <div class="slider-block">
        <span class="slider-text">大小</span>
        <el-slider v-model="fontSize" :min="20" :max="100" @input="refresh"/>
      </div>

      <div class="slider-block">
        <span class="slider-text">内边框</span>
        <el-slider v-model="innerBorder" :min="0" :max="20" @input="refresh"/>
      </div>

      <div class="slider-block">
        <span class="slider-text">外边框</span>
        <el-slider v-model="outerBorder" :min="0" :max="20" @input="refresh"/>
      </div>

      <div class="slider-block">
        <span class="slider-text">弯曲</span>
        <el-slider v-model="curve" :min="0" :max="360" @input="refresh"/>
      </div>

      <div class="slider-block">
        <span class="slider-text">半径</span>
        <el-slider v-model="radius" :min="0" :max="500" @input="refresh"/>
      </div>
    </div>

    <div class="picker-color">
      <div>
        <span>字体</span>
        <el-color-picker v-model="fontColor" @active-change="refreshFontColor"/>
      </div>
      <div>
        <span>内边框</span>
        <el-color-picker v-model="innerBorderColor" @active-change="refreshInterBorderColor"/>
      </div>
      <div>
        <span>外边框</span>
        <el-color-picker v-model="outerBorderColor" @active-change="refreshOuterBorderColor"/>
      </div>
    </div>

    <div class="input-block">
      <span class="input-text">文字</span>
      <el-input style="font-size: 20px" v-model="text" placeholder="Text" @input="refresh"/>
    </div>
  </div>
</template>

<script setup lang="ts">
import {ref, onMounted} from 'vue';
import {Search} from '@element-plus/icons-vue'
import characters from './assets/characters.json';

let ctx: CanvasRenderingContext2D;
let img: HTMLImageElement;

const canvas = ref();
const text = ref('哼哼!');
const xAxis = ref(90); // x轴坐标偏移
const yAxis = ref(250); // y轴坐标偏移
const rotate = ref(135); // 旋转角度
const fontSize = ref(50); // 字体大小
const fontColor = ref('#c09edd'); // 字体颜色
const curve = ref(48); // 弯曲角度
const radius = ref(220); // 弯曲半径

const innerBorder = ref(9); // 内边框粗细
const outerBorder = ref(11); // 外边框粗细
const outerBorderColor = ref('#FFFFFF'); // 外边框颜色
const innerBorderColor = ref('#7844a3'); // 内边框颜色

onMounted(async () => {
  ctx = canvas.value.getContext('2d');
  img = new Image();
  img.src = import.meta.env.BASE_URL + "img/Arcaea/luna.png"
  img.onload = async () => {
    await waitFontLoad("YurukaStd");
    await waitFontLoad("SSFangTangTi");
    refresh();
  };
});

const drawImg = () => {
  const canvasWidth = 300
  const canvasHeight = 300
  const imgWidth = img.width;
  const imgHeight = img.height;
  // 计算纵横比
  const imgAspectRatio = imgWidth / imgHeight;
  const canvasAspectRatio = canvasWidth / canvasHeight;
  let drawWidth, drawHeight, offsetX = 0, offsetY = 0;
  if (imgAspectRatio > canvasAspectRatio) {
    // 图片宽度超出，按宽度进行缩放
    drawWidth = canvasWidth;
    drawHeight = canvasWidth / imgAspectRatio;
    offsetY = (canvasHeight - drawHeight) / 2; // 垂直居中
  } else {
    // 图片高度超出，按高度进行缩放
    drawHeight = canvasHeight;
    drawWidth = canvasHeight * imgAspectRatio;
    offsetX = (canvasWidth - drawWidth) / 2; // 水平居中
  }
  // 绘制图片，保持其纵横比
  ctx.drawImage(img, offsetX, offsetY, drawWidth, drawHeight);
}

const refresh = () => {
  ctx.clearRect(0, 0, 300, 300);
  // ctx.drawImage(img, 0, 0, 300, 300);
  drawImg();
  // 设置文字样式
  ctx.font = `${fontSize.value}px YurukaStd, SSFangTangTi`;
  ctx.miterLimit = 2.5;
  ctx.fillStyle = fontColor.value;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.save();

  if (curve.value === 0) {
    ctx.translate(xAxis.value, 300 - yAxis.value);
    ctx.rotate(degreeToArc(rotate.value + 180));
    // 绘制外边框
    if (outerBorder.value > 0) {
      ctx.strokeStyle = outerBorderColor.value;
      ctx.lineWidth = innerBorder.value + outerBorder.value;
      ctx.strokeText(text.value, 0, 0);
    }
    // 绘制内边框
    if (innerBorder.value > 0) {
      ctx.strokeStyle = innerBorderColor.value;
      ctx.lineWidth = innerBorder.value;
      ctx.strokeText(text.value, 0, 0);
    }
    // 绘制文字内容
    ctx.fillText(text.value, 0, 0);
    ctx.restore();
  } else {
    ctx.translate(xAxis.value, 300 - yAxis.value);
    ctx.rotate(degreeToArc(rotate.value + 90));
    const length = ctx.measureText(text.value).width;
    const angle = degreeToArc(curve.value) / text.value.length;
    for (let i = 0; i < text.value.length; i++) {
      ctx.save();
      if (isSymbol(text.value[i])) {
        ctx.translate(
            radius.value * Math.cos(angle * (i - 0.25)) - radius.value,
            radius.value * Math.sin(angle * (i - 0.25)) - length / 2
        );
        ctx.rotate(Math.PI / 2 + angle * i);
      } else {
        ctx.translate(
            radius.value * Math.cos(angle * i) - radius.value,
            radius.value * Math.sin(angle * i) - length / 2
        );
        ctx.rotate(Math.PI / 2 + angle * i);
      }
      // 绘制外边框
      if (outerBorder.value > 0) {
        ctx.strokeStyle = outerBorderColor.value;
        ctx.lineWidth = innerBorder.value + outerBorder.value;
        ctx.strokeText(text.value[i], 0, 0);
      }
      // 绘制内边框
      if (innerBorder.value > 0) {
        ctx.strokeStyle = innerBorderColor.value;
        ctx.lineWidth = innerBorder.value;
        ctx.strokeText(text.value[i], 0, 0);
      }
      ctx.fillText(text.value[i], 0, 0);
      ctx.restore();
    }
  }
  ctx.restore();
}

const refreshFontColor = (color: string) => {
  fontColor.value = color;
  refresh();
}

const refreshInterBorderColor = (color: string) => {
  innerBorderColor.value = color;
  refresh();
}

const refreshOuterBorderColor = (color: string) => {
  outerBorderColor.value = color;
  refresh();
}

// 角度转弧度
const degreeToArc = (degree: number) => {
  return (degree * Math.PI) / 180;
}

// 判断是否为符号
const isSymbol = (char: string) => {
  return /[\u2000-\u206F\u2E00-\u2E7F\\!"#$%&'()*+,\-./:;<=>?@\[\]^_`{|}~]/.test(char);
}

// 等待字体加载
const waitFontLoad = async (fontName: string) => {
  return new Promise((resolve) => {
    const checkFont = setInterval(() => {
      if (hasFont(fontName)) {
        clearInterval(checkFont);
        resolve(1);
      }
    }, 100);
  });
}

// 检测字体是否加载
const hasFont = (fontName: string) => {
  // 创建一个比较基准的canvas元素来获取默认宽度和高度
  const canvas = document.createElement('canvas');
  const context = canvas.getContext('2d') as CanvasRenderingContext2D;

  // 比较的基准字体应该是系统中不可能存在的
  const text = 'abcdefghijklmnopqrstuvwxyz0123456789';
  context.font = '72px "font that doesnt exist", monospace';

  // 获取比较基准的尺寸
  const baselineSize = context.measureText(text).width;

  // 再次设置需要检测的字体并获取其尺寸
  context.font = `72px "${fontName}", monospace`;
  const checkSize = context.measureText(text).width;

  // 如果两次返回的宽度相同，那么指定的字体可能未安装
  return checkSize !== baselineSize;
}


// 获取 public 目录下的所有图片文件路径
const images = import.meta.glob('/public/img/**/*.{jpg,png,gif}', {eager: true});
const imgDir = new Map();
Object.keys(images).forEach(filePath => {
  const relativePath = filePath.replace('/public/', import.meta.env.BASE_URL);
  const parentName = relativePath.split('/').slice(-2, -1)[0];
  if (!imgDir.has(parentName)) {
    imgDir.set(parentName, []);
  }
  imgDir.get(parentName).push(relativePath);
});

const pickerPopover = ref();
const searchText = ref('')
const activeTab = ref("All");
const allTabs = ref(["All", ...imgDir.keys()]);
const tabs = ref(allTabs.value);
const searchTabs = (value: string) => {
  if (value === '') {
    tabs.value = allTabs.value;
  } else {
    tabs.value = allTabs.value.filter(tab => tab.toLowerCase().includes(value.toLowerCase()));
  }
  activeTab.value = tabs.value[0];
}

const pickerImg = (src: string) => {
  const endIdx = src.lastIndexOf('.');
  const startIdx = src.lastIndexOf('/', endIdx) + 1;
  const name = src.slice(startIdx, endIdx).split("_")[0];
  const chara = characters.find((item: any) => item.character.toLowerCase() == name.toLowerCase());
  if (chara) {
    fontColor.value = chara.fillColor;
    if (chara.strokeColor) {
      innerBorderColor.value = chara.strokeColor;
      innerBorder.value = 9;
    } else {
      innerBorderColor.value = '#FFFFFF';
      innerBorder.value = 0;
    }
  }
  pickerPopover.value.hide();
  img.src = src;
  refresh();
}

const download = () => {
  // 下载 canvas 为图片
  const link = document.createElement('a');
  link.href = canvas.value.toDataURL('image/png');
  link.download = 'image.png';
  link.click();
}

</script>

<style scoped lang="less">
.container {
  width: 500px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.image-wrapper {
  position: relative;
  margin-bottom: 10px;
  margin-left: 40px;
}

.x-axis {
  width: 300px;
  margin-left: 10px;

  & :deep(.el-slider__bar) {
    display: none;
  }
}

.y-axis {
  height: 300px;
  margin-left: 10px;

  & :deep(.el-slider__bar) {
    display: none;
  }
}

.image {
  width: 100%;
  height: 100%;
}

.controls {
  width: 100%;
  margin: 10px 0;
}

.slider {
  &-block {
    display: flex;
    padding: 0 40px;
  }

  &-text {
    display: flex;
    width: 70px;
    margin-right: 10px;
    align-items: center;
  }
}

.picker-color {
  display: flex;
  margin-bottom: 20px;

  & > * {
    margin-left: 28px;

    & > * {
      margin-right: 10px;
    }
  }

}

.input {
  &-block {
    display: flex;
    padding: 0 40px;
    margin-bottom: 30px;
  }

  &-text {
    display: flex;
    width: 50px;
    margin-right: 10px;
    align-items: center;
  }
}

.picker {

  &-container {
    display: grid;
    grid-template-columns: repeat(6, 1fr);
    gap: 5px;
    max-height: 500px;
    overflow-y: auto;
  }

  &-container::-webkit-scrollbar {
    display: none;
  }

  &-item {
    max-width: 60px;
    height: 60px;
    object-fit: contain;
  }
}
</style>
