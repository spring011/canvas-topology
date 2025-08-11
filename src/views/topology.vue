<template>
    <div class="canvas" v-show="true" ref="canvasBox">
      <canvas id="canvas" ref="canvas"></canvas>
        <div class="tooltip-wrapper" :style="{left:tooltipData.posLeft, top: tooltipData.posTop, display: tooltipData.display}">
          <p>{{ tooltipData.text}}</p>
        </div>
    </div>
</template>
<script lang="ts" setup>
import database from '@/images/database.png';
import database_error from '@/images/database_error.png';
import info_office from '@/images/info_office.png';
import info_office_error from '@/images/info_office_error.png';
import network from '@/images/network.png';
import network_error from '@/images/network_error.png';
import processor from '@/images/processor.png';
import processor_error from '@/images/processor_error.png';
import tianjing from '@/images/tianjing.png';
import tianjing_error from '@/images/tianjing_error.png';
import tianqing from '@/images/tianqing.png';
import tianqing_error from '@/images/tianqing_error.png';
import tianyuan from '@/images/tianyuan.png';
import tianyuan_error from '@/images/tianyuan_error.png';

import { debounce } from 'lodash';
import { onMounted, onUnmounted, reactive, ref } from 'vue';

interface imgList {
  src_error: string;
  src: string;
  id: string;
  name: string;
  state: boolean;
  x: number;
  y: number;
}

let tooltipData = reactive({posLeft:'0px',posTop:'0px',display:'none',text:''});

const canvas = ref<HTMLCanvasElement | null>(null);
const canvasBox = ref<HTMLElement | null>(null);
let ctx: CanvasRenderingContext2D | null;
let canvas_w: number = 0,
  canvas_h: number = 0;
const imgWidth = 36;
// const imgList: imgList[] = [
//   {
//     src_error: network_error,
//     src: network,
//     id: 'network',
//     name: '',
//     state: true,
//     x: 0,
//     y: 0,
//   },
//   // {
//   //   src_error: oss_error,
//   //   src: oss,
//   //   id: 'oss',
//   //   name: 'oss储存',
//   //   state: false,
//   //   x: 100,
//   //   y: 60,
//   // },
//   {
//     src_error: database_error,
//     src: database,
//     id: 'database',
//     name: '数据库',
//     state: false,
//     x: 0,
//     y: 90,
//   },
//   {
//     src_error: processor_error,
//     src: processor,
//     id: 'processor',
//     name: '前置机',
//     state: true,
//     x: -100,
//     y: 60,
//   },
//   {
//     src_error: tianyuan_error,
//     src: tianyuan,
//     id: 'tianyuan',
//     name: '天元',
//     state: false,
//     x: -40,
//     y: -120,
//   },
//   {
//     src_error: tianyuan_error,
//     src: tianyuan,
//     id: 'xinxihuaban',
//     name: '信息化办',
//     state: true,
//     x: -110,
//     y: -40,
//   },
//   {
//     src_error: tianyuan_error,
//     src: tianyuan,
//     id: 'tianqing',
//     name: '天擎',
//     state: false,
//     x: 80,
//     y: -100,
//   },
//   {
//     src_error: tianyuan_error,
//     src: tianyuan,
//     id: 'tianjing',
//     name: '天镜',
//     state: false,
//     x: 120,
//     y: -30,
//   },
// ];
const imgList: imgList[] = [
  {
    src_error: network_error,
    src: network,
    id: 'network',
    name: '定位平台',
    state: true,
    x: 0,
    y: 0,
  },
  {
    src_error: database_error,
    src: database,
    id: 'database',
    name: '数据库',
    state: false,
    x: 40,
    y: 110,
  },
  {
    src_error: processor_error,
    src: processor,
    id: 'processor',
    name: '前置机',
    state: true,
    x: -85,
    y: 95,
  },
  {
    src_error: tianyuan_error,
    src: tianyuan,
    id: 'tianyuan',
    name: '站点平台',
    state: false,
    x: -40,
    y: -120,
  },
  {
    src_error: info_office_error,
    src: info_office,
    id: 'xinxihuaban',
    name: '信息化办',
    state: true,
    x: -110,
    y: -40,
  },
  {
    src_error: tianqing_error,
    src: tianqing,
    id: 'tianqing',
    name: '大数据平台',
    state: false,
    x: 80,
    y: -100,
  },
  {
    src_error: tianjing_error,
    src: tianjing,
    id: 'tianjing',
    name: '监控平台',
    state: false,
    x: 120,
    y: -0,
  },
];

function initCanvas() {
  ctx = canvas.value!.getContext('2d');
  setTimeout(() => {
    resize();
  });
  
  window.addEventListener('resize', debounceResize);
}
const debounceResize = debounce(resize, 500)
function resize() {
  canvas_w = canvasBox.value!.offsetWidth;
  canvas_h = canvasBox.value!.offsetHeight;
  canvas.value!.width = canvas_w;
  canvas.value!.height = canvas_h;

  updateCanvas();
}
let _unit = 1;
function updateCanvas() {
  if (!ctx) return;
  const _w = 375;
  const _h = 318;
  if (canvas_w / _w > canvas_h / _h) {
    _unit = canvas_h / _h;
  } else {
    _unit = canvas_w / _w;
  }
  ctx.clearRect(0, 0, canvas_w, canvas_h);
  ctx.translate((canvas_w) / 2, (canvas_h) / 2);
  ctx.scale(_unit, _unit);
 
  ctx && draw(ctx);
  bindEvent();
}
function bindEvent() {
  canvas.value!.addEventListener('mousemove',(e: any) => {
    // canvas容器相对于窗口的位置
    let canvasBox = canvas.value!.getBoundingClientRect();

    //e.clientX相对于窗口的位置，canvasBox.left canvas容器相对于窗口的左侧距离，mouseX鼠标在原坐标系的位置
    let mouseX = e.clientX -  canvasBox.left;
    let mousY = e.clientY - canvasBox.top;
    checkMousePos(mouseX, mousY);
  })
}

function checkMousePos(mouseX: number, mouseY: number) {//mouseX,mouseY鼠标在原坐标系的位置
  let flag = false;
  for (let i = 0 ; i < imgList.length; i++) {
    
    //变换后在原坐标系的位置
    let minx = (imgList[i].x - imgWidth / 2 ) * _unit + canvas_w / 2;
    let maxx = (imgList[i].x  + imgWidth / 2) * _unit + canvas_w / 2;
    let miny = (imgList[i].y - imgWidth / 2)  * _unit + canvas_h / 2;
    let maxy = (imgList[i].y  + imgWidth / 2) * _unit + canvas_h / 2;
    
    if (mouseX > minx && mouseX < maxx && mouseY > miny && mouseY < maxy) {
      canvas.value &&(canvas.value.style.cursor = 'pointer');
      // showErrorDetails();
      drawToolTip(imgList[i].name, mouseX, mouseY);
      flag = true;
      break
    } else {
      canvas.value &&(canvas.value.style.cursor = 'default');
    }
  }
  if (!flag) {
    hideToolTip();
  }
}
function hideToolTip() {
  tooltipData.display = 'none';
}
function drawToolTip(text:string, x:number, y:number) {
  tooltipData.posLeft = x + 'px';
  tooltipData.posTop = y - 10 + 'px';
  tooltipData.text = text;
  tooltipData.display = 'block';
}


function draw(ctx: CanvasRenderingContext2D) {
  ctx.save();
  ctx.fillStyle = 'rgb(16, 16, 20)';
  ctx.arc(0, 0, imgWidth / 2 + 3, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();
  imgList.forEach((el: imgList) => {
    ctx.save();
    loadImage(ctx, el);
    ctx.fillStyle = '#fff';
    ctx.font = '11px serif';
    ctx.fillText(
      el.name,
      el.x - ctx.measureText(el.name).width / 2,
      imgWidth / 2 + el.y + 16,
    );
    ctx.restore();
  });
}
function loadImage(ctx: CanvasRenderingContext2D, imageInfo: imgList) {
  const img = new Image();
  img.src = imageInfo.state ? imageInfo.src : imageInfo.src_error;
  img.onload = () => {
    ctx.drawImage(
      img,
      imageInfo.x - imgWidth / 2,
      imageInfo.y - imgWidth / 2,
      imgWidth,
      imgWidth,
    );

    ctx.save();
    ctx.beginPath();
    if (imageInfo.id !== 'network') {
      ctx.globalCompositeOperation = 'destination-over';
      ctx.strokeStyle = '#fff';

      if (imageInfo.state) {
        ctx.strokeStyle = '#1296DB';
      } else {
        ctx.strokeStyle = '#DF0B01';
      }
      ctx.lineWidth = 2;
      const info = { x: 0, y: 0 };
      if (imageInfo.x > 0) {
        info.x = -1;
      } else {
        info.x = 1;
      }

      if (imageInfo.y > 0) {
        info.y = -1;
      } else {
        info.y = 1;
      }

      // const angle = 45; // 直线与x轴夹角
      const x1 = 0, y1 = 0; // 直线起点
      const x2 = imageInfo.x, y2 = imageInfo.y + (info.y * imgWidth) / 2; // 直线终点
      // 计算箭头坐标并绘制
      // const Lpoint = {x: x1 + r * Math.cos(angle + theta), y: y1 + r * Math.sin(angle + theta)};
      // const Rpoint = {x: x1 + r * Math.cos(angle - theta), y: y1 + r * Math.sin(angle - theta)};
      ctx.moveTo(x1,y1);
      ctx.lineTo(x2,y2);
      ctx.stroke();

      // 计算箭头的方向
      const angle = Math.atan2(y2 - y1, x2 - x1);
 
      // 计算两个端点，用于构成箭头的两个边
      const h = 6 / Math.cos(Math.PI / 6);  // 考虑到三角形的边长和角度
      const angle1 = (angle + Math.PI / 6); // 60度角的一边
      const angle2 = (angle - Math.PI / 6); // 60度角的另一边
      const topX = x2 - Math.cos(angle1) * h;
      const topY = y2 - Math.sin(angle1) * h;
      const botX = x2 - Math.cos(angle2) * h;
      const botY = y2 - Math.sin(angle2) * h;
  
      // 开始一个新的路径以绘制箭头头
      ctx.beginPath();
  
      // 移动到箭头的起点（即线段的终点）
      ctx.moveTo(x2, y2);
  
      // 绘制箭头头部的两个边
      ctx.lineTo(topX, topY);
      ctx.moveTo(x2, y2);
      ctx.lineTo(botX, botY);
      ctx.stroke();
    }
    ctx.restore();
  };
}

onMounted(() => {
  initCanvas();
});
onUnmounted(() => {
  window.removeEventListener('resize', resize);
});
</script>
<style lang="scss" scoped>
  .canvas {
    position: relative;
    z-index: 100;
    width: 80%;
    height: calc(100% - 20px);
    margin: 0 auto;
    .tooltip-wrapper {
      position: absolute;
      z-index: 200;
      display: block;
      left: 10px;
      top: 10px;
      padding: 6px 8px;
      background-color: rgba(255,255,0,0.8);
      color: #333;
      font-size: 12px;
      transform: translate(-50%, -100%);
    }
  }
</style>