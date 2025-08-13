<template>
    <div class="canvas" v-show="true" ref="canvasBox">
      <canvas id="canvas1" ref="canvas"></canvas>
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

import * as zrender from 'zrender';

interface imgList {
  src_error: string;
  src: string;
  id: string;
  job: string;
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

const imgList: imgList[] = [
  {
    src_error: network_error,
    src: network,
    id: 'network',
    job: '定位平台',
    state: true,
    x: 160,
    y: 150,
  },
  {
    src_error: database_error,
    src: database,
    id: 'database',
    job: '数据库',
    state: false,
    x: 120,
    y: 280,
  },
  {
    src_error: processor_error,
    src: processor,
    id: 'processor',
    job: '前置机',
    state: true,
    x: 200,
    y: 10,
  },
  {
    src_error: tianyuan_error,
    src: tianyuan,
    id: 'tianyuan',
    job: '站点平台',
    state: false,
    x: 40,
    y: 180,
  },
  {
    src_error: info_office_error,
    src: info_office,
    id: 'xinxihuaban',
    job: '信息化办',
    state: true,
    x: 270,
    y: 240,
  },
  {
    src_error: tianqing_error,
    src: tianqing,
    id: 'tianqing',
    job: '大数据平台',
    state: false,
    x: 290,
    y: 110,
  },
  {
    src_error: tianjing_error,
    src: tianjing,
    id: 'tianjing',
    job: '监控平台',
    state: false,
    x: 60,
    y: 50,
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
  _unit;
  // ctx.clearRect(0, 0, canvas_w, canvas_h);
  // ctx.translate(canvas_w / 2, canvas_h / 2);
  // ctx.scale(_unit, _unit);

  ctx && draw(ctx);
  bindEvent();
}
function bindEvent() {
  canvas.value!.addEventListener('mousemove', (e: any) => {
    // canvas容器相对于窗口的位置
    let canvasBox = canvas.value!.getBoundingClientRect();

    //e.clientX相对于窗口的位置，canvasBox.left canvas容器相对于窗口的左侧距离，mouseX鼠标在原坐标系的位置
    let mouseX = e.clientX - canvasBox.left;
    let mousY = e.clientY - canvasBox.top;
    showHover(mouseX, mousY);
  });
}

function showHover(mouseX: number, mouseY: number) {//mouseX,mouseY鼠标在原坐标系的位置
  //mouseX,mouseY鼠标在原坐标系的位置
  let flag = false;
  for (let i = 0; i < imgList.length; i++) {
    let item = imgList[i];

    //变换后在原坐标系的位置
    let minx = item.x;
    let maxx = item.x + imgWidth;
    let miny = item.y;
    let maxy = item.y + imgWidth;

    if (mouseX > minx && mouseX < maxx && mouseY > miny && mouseY < maxy) {
      // canvas.value &&(canvas.value.style.cursor = 'pointer');

      drawToolTip('节点', mouseX, mouseY);
      flag = true;
      break;
    } else {
      // canvas.value &&(canvas.value.style.cursor = 'default');
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
let zr: any;
function initZrender() {
  zr = zrender.init(document.getElementById('canvas1'));
  const _w = 375;
  const _h = 318;
  if (canvas_w / _w > canvas_h / _h) {
    _unit = canvas_h / _h;
  } else {
    _unit = canvas_w / _w;
  }
}
function drawImage(node: any) {
  let src = '';
  if (node.warn) {
    src = node.src;
  } else {
    src = node.state ? node.src : node.src_error;
  }
  let img = new zrender.Image({
    style: {
      image: src,
      x: node.x,
      y: node.y,
      width: imgWidth,
      height: imgWidth,
    },
    z: 100,
  });
  if (node.warn) {
    img.on('click', (data:any) => {
      alert('click');
      console.log(data)
    });
  }

  zr.add(img);
}
function drawText(node: any) {
  let x0 = 0;
  if (node.job.length > 4) {
    x0 = 3 * node.job.length;
  }

  let text = new zrender.Text({
    style: {
      fill: '#fffefe',
      text: node.job,
      fontSize: '10',
      x: node.x - x0,
      y: node.y + imgWidth,
    },
  });
  zr.add(text);
}
function drawCircle(cx: number, cy: number, r: number) {
  let circle = new zrender.Circle({
    shape: {
      cx: cx,
      cy: cy,
      r: r,
    },
    style: {
      fill: '#363c41',
    },
    z: 3,
  });
  zr.add(circle);
}
function drawLine(state: boolean, x1: number, y1: number, x2: number, y2: number) {
  let points = [
    [x1, y1],
    [x2, y2],
  ];
  let color = '#1296DB';
  if (!state) {
    color = '#DF0B01';
  }
  // 直线
  const line = new zrender.Polyline({
    shape: {
      points: points,
    },
    style: {
      stroke: color,
      lineWidth: 2,
    },
  });

  const triangle: any = new zrender.Polygon({
    shape: {
      points: [
        [0, -9],
        [9, 0],
        [-9, 0],
      ],
    },
    style: {
      fill: color,
    },
    z: 2,
  });
  // ZRender以逆时针为正
  triangle.rotation = -Math.PI / 2;

  let accLenList = [0];
  for (let i = 1; i < points.length; i++) {
    const p1 = points[i - 1];
    const p2 = points[i];
    const dist = zrender.vector.dist(p1, p2);

    accLenList.push(accLenList[i - 1] + dist);
  }

  let percentList = accLenList.map((acc) => {
    return acc / accLenList[accLenList.length - 1];
  });

  triangle.position = [points[0][0], points[0][1]];

  triangle.__t = 0;
  let frame = 1;
  triangle
    .animate('', true)
    .when(3000, { __t: 1 })
    .during((obj: any, i: any) => {
      obj;
      for (let j = 1; j < percentList.length; j++) {
        if (i > percentList[j - 1] && i < percentList[j]) {
          frame = j;
          break;
        }
      }

      const angle = -Math.atan2(
        points[frame][1] - points[frame - 1][1],
        points[frame][0] - points[frame - 1][0],
      );
      triangle.rotation = angle - Math.PI / 2;

      zrender.vector.lerp(
        triangle.position,
        points[frame - 1],
        points[frame],
        (i - percentList[frame - 1]) / (percentList[frame] - percentList[frame - 1]),
      );
    })
    .start();

  zr.add(line);
  zr.add(triangle);
}

function draw(ctx: CanvasRenderingContext2D) {
 console.log(ctx);
  initZrender();
  imgList.forEach((imageInfo: any) => {
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

    const x1 = imgList[0].x + imgWidth/2,
      y1 = imgList[0].y + imgWidth/2; // 直线起点
    // const x2 = imageInfo.x, y2 = imageInfo.y + (info.y * imgWidth) / 2; // 直线终点

    drawCircle(imageInfo.x + imgWidth/2, imageInfo.y + imgWidth/2, 14);
    if (imageInfo.job == '全闪定位仪') {
      drawLine(imageInfo.state, imageInfo.x + imgWidth/2, imageInfo.y + imgWidth/2, x1, y1);
    } else {
      drawLine(imageInfo.state, x1, y1, imageInfo.x + imgWidth/2, imageInfo.y + imgWidth/2);
    }

    drawImage(imageInfo);
    drawText(imageInfo);
  });
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