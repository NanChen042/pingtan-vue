<template>
  <div class="map-tab-scroll-container w-full h-full overflow-y-auto bg-slate-100 pb-16 select-none">
    <!-- 1. 顶部 4 日日程横向滑动卡片栏 -->
    <div class="px-3 pt-2.5 pb-1">
      <div class="flex overflow-x-auto no-scrollbar gap-2 pb-0.5 -mx-1 px-1">
        <button
          v-for="day in daysList"
          :key="day.id"
          type="button"
          class="shrink-0 w-28 p-2 rounded-xl border text-left transition cursor-pointer flex flex-col justify-between shadow-2xs"
          :class="currentDayKey === day.id ? 'border-sky-500 bg-sky-50/90 ring-1 ring-sky-400/40' : 'border-slate-200 bg-white hover:border-slate-300'"
          @click="handleDaySelect(day.id)"
        >
          <div class="flex items-center justify-between">
            <span class="text-xs font-bold" :class="currentDayKey === day.id ? 'text-sky-700' : 'text-slate-800'">
              {{ day.label }}
            </span>
            <span
              class="text-[9px] px-1.5 py-0.2 rounded font-bold"
              :style="{
                backgroundColor: currentDayKey === day.id ? day.color + '22' : '#f1f5f9',
                color: currentDayKey === day.id ? day.color : '#64748b'
              }"
            >
              {{ day.theme }}
            </span>
          </div>
          <div class="text-[10px] text-slate-500 mt-1 truncate font-medium">
            {{ day.summary }}
          </div>
        </button>
      </div>
    </div>

    <!-- 2. 纯净平潭县高德地图卡片 (独立全景视窗) -->
    <div class="mx-3 mt-1 bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden flex flex-col">
      <!-- 地图画布 -->
      <div class="relative h-[320px] sm:h-[360px] w-full bg-slate-100">
        <div id="amap-vue-container" class="w-full h-full"></div>

        <!-- 右上角快捷操作工具胶囊 (极简纯净，仅保留卫星切换与最佳视野) -->
        <div class="absolute top-2.5 right-2.5 z-10 flex items-center gap-1.5">
          <!-- 卫星底图切换 -->
          <button
            type="button"
            class="bg-white/95 backdrop-blur-md px-2.5 py-1.5 rounded-lg border border-slate-200 text-xs font-bold text-slate-700 shadow-sm hover:bg-white active:scale-95 transition flex items-center gap-1 cursor-pointer"
            :title="isSatellite ? '切换矢量地图' : '切换卫星影像'"
            @click="toggleSatellite"
          >
            <span>{{ isSatellite ? '🗺️' : '🛰️' }}</span>
            <span>{{ isSatellite ? '矢量' : '卫星' }}</span>
          </button>

          <!-- 视角重置与自适应 -->
          <button
            type="button"
            class="bg-white/95 backdrop-blur-md px-2.5 py-1.5 rounded-lg border border-slate-200 text-xs font-bold text-sky-700 shadow-sm hover:bg-white active:scale-95 transition flex items-center gap-1 cursor-pointer"
            title="自适应最佳视野"
            @click="fitCurrentDayViewport"
          >
            <span>🎯</span>
            <span>视野</span>
          </button>
        </div>
      </div>

      <!-- 横向景点快捷导览链条 (第一项为全天总览，后跟各站点) -->
      <div class="px-2.5 py-2 border-t border-slate-100 bg-slate-50/90 flex items-center gap-2 overflow-x-auto no-scrollbar" ref="chipsBarRef">
        <!-- 全天动线总览 Chip -->
        <button
          type="button"
          :ref="el => chipRefs[-1] = el"
          class="shrink-0 px-3 py-1.5 rounded-full text-xs font-bold transition-all duration-200 flex items-center gap-1 cursor-pointer border"
          :class="currentSpotIndex === -1
            ? 'bg-sky-600 text-white border-sky-600 shadow-sm ring-2 ring-sky-300 scale-105'
            : 'bg-white text-slate-700 border-slate-200 hover:border-slate-300'"
          @click="showFullDayOverview"
        >
          <span>🗺️</span>
          <span class="whitespace-nowrap">全天总览</span>
        </button>

        <!-- 各站点 Chips -->
        <button
          v-for="(spot, idx) in currentDayData.spots"
          :key="spot.id || idx"
          :ref="el => chipRefs[idx] = el"
          type="button"
          class="shrink-0 px-3 py-1.5 rounded-full text-xs font-bold transition-all duration-200 flex items-center gap-1 cursor-pointer border"
          :class="idx === currentSpotIndex
            ? 'bg-gradient-to-r from-amber-500 to-orange-500 text-white border-orange-500 shadow-sm ring-2 ring-orange-300 scale-105'
            : 'bg-white text-slate-700 border-slate-200 hover:border-slate-300'"
          @click="handleSpotSelect(idx)"
        >
          <span class="text-xs shrink-0">{{ spot.icon }}</span>
          <span class="whitespace-nowrap">{{ spot.shortName || spot.name }}</span>
        </button>
      </div>
    </div>

    <!-- 3. 下方详情控制卡 (自适应切换：全天总览 vs 单段分支路线) -->

    <!-- 状态 A：全天路线总览卡片 (currentSpotIndex === -1) -->
    <div v-if="currentSpotIndex === -1" class="mx-3 mt-2.5 bg-white rounded-2xl border border-slate-200 shadow-sm p-3.5 space-y-3">
      <!-- 头部：标题与当日概况 -->
      <div class="flex items-center justify-between pb-2 border-b border-slate-100">
        <div class="flex items-center gap-2">
          <span class="w-2.5 h-2.5 rounded-full" :style="{ backgroundColor: currentDayData.color }"></span>
          <h3 class="font-extrabold text-slate-900 text-sm">
            {{ currentDayData.title }}
          </h3>
        </div>
        <span class="text-xs font-bold px-2 py-0.5 rounded-full text-sky-700 bg-sky-50 border border-sky-200 shrink-0">
          全天共 {{ currentDayData.spots.length }} 站
        </span>
      </div>

      <!-- 大本营提示 -->
      <div class="p-2 rounded-xl bg-slate-50 border border-slate-200/70 text-xs text-slate-600 flex items-center justify-between">
        <div class="flex items-center gap-1.5 font-medium truncate pr-2">
          <span>🏨</span>
          <span class="truncate">大本营驻地：全季酒店（平潭红湖东路32号）</span>
        </div>
        <button type="button" class="text-sky-600 font-bold shrink-0 hover:underline text-[11px]" @click="focusHotelBasecamp">
          定位
        </button>
      </div>

      <!-- 动线打卡节点流 (点击任意站即可切换查看该段路线) -->
      <div>
        <div class="text-xs font-bold text-slate-700 mb-2 flex items-center justify-between">
          <span>🎯 当日动线节点（点击任意站看单段接驳）:</span>
        </div>
        <div class="grid grid-cols-1 gap-1.5">
          <div
            v-for="(spot, idx) in currentDayData.spots"
            :key="spot.id || idx"
            class="p-2.5 rounded-xl border border-slate-100 bg-slate-50/70 hover:bg-slate-100/90 hover:border-slate-300 transition cursor-pointer flex items-center justify-between"
            @click="handleSpotSelect(idx)"
          >
            <div class="flex items-center gap-2 min-w-0">
              <span
                class="w-5 h-5 rounded-full flex items-center justify-center text-[10px] font-extrabold text-white shrink-0"
                :style="{ backgroundColor: currentDayData.color }"
              >
                {{ idx + 1 }}
              </span>
              <span class="text-xs font-bold text-slate-800 truncate flex items-center gap-1">
                <span>{{ spot.icon }}</span>
                <span>{{ spot.name }}</span>
              </span>
              <span class="text-[10px] text-slate-400 truncate hidden sm:inline">{{ spot.tag }}</span>
            </div>
            <div class="flex items-center gap-1 text-[11px] text-sky-600 font-bold shrink-0">
              <span>看分支</span>
              <span>➔</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 当日小贴士 -->
      <div class="text-[11px] text-slate-600 bg-amber-50/70 border border-amber-200/80 rounded-xl p-2.5 leading-relaxed">
        💡 <strong class="text-amber-950">行程全貌：</strong>{{ currentDayData.desc }}
      </div>
    </div>

    <!-- 状态 B：单分支路线详情卡片 (currentSpotIndex >= 0) -->
    <div v-else class="mx-3 mt-2.5 bg-white rounded-2xl border border-slate-200 shadow-sm p-3.5 space-y-3">
      <!-- 头部：当前站信息与返回全天路线按钮 -->
      <div class="flex items-center justify-between pb-2 border-b border-slate-100">
        <div class="flex items-center gap-1.5 min-w-0 pr-2">
          <span class="text-base shrink-0">{{ activeSpot?.icon || '📍' }}</span>
          <div class="min-w-0">
            <div class="flex items-center gap-1.5">
              <h3 class="font-extrabold text-orange-950 text-sm truncate">{{ activeSpot?.name }}</h3>
              <span class="text-[10px] font-bold px-1.5 py-0.2 bg-orange-100 text-orange-700 rounded-md shrink-0">
                第 {{ currentSpotIndex + 1 }} 站
              </span>
            </div>
            <p class="text-[10px] text-amber-700 font-medium truncate">{{ activeSpot?.tag }}</p>
          </div>
        </div>

        <!-- 一键返回全天总览路线 -->
        <button
          type="button"
          class="shrink-0 px-2.5 py-1 rounded-lg bg-sky-50 text-sky-700 border border-sky-200 text-xs font-bold hover:bg-sky-100 transition flex items-center gap-1 cursor-pointer"
          @click="showFullDayOverview"
        >
          <span>🗺️</span>
          <span>全天路线</span>
        </button>
      </div>

      <!-- 站点玩法说明与高德导航直达 -->
      <div class="flex items-start justify-between gap-2 bg-slate-50 p-2.5 rounded-xl text-xs border border-slate-100">
        <p class="text-xs text-slate-600 leading-relaxed flex-1">
          {{ activeSpot?.desc }}
        </p>
        <van-button
          size="mini"
          type="warning"
          round
          icon="guide-o"
          class="shrink-0 font-bold px-2 py-1 shadow-2xs"
          :url="amapNavUrl"
          target="_blank"
        >
          高德导航
        </van-button>
      </div>

      <!-- 单分支接驳方式与实时测算 (存在下一站时) -->
      <div v-if="hasNextSpot" class="space-y-2 pt-1">
        <div class="flex items-center justify-between text-xs">
          <span class="text-slate-700 font-bold flex items-center gap-1 truncate pr-2">
            <span>➔ 前往：</span>
            <strong class="text-orange-950 truncate">{{ nextSpot?.shortName || nextSpot?.name }}</strong>
          </span>
          <span class="text-[10px] text-sky-700 bg-sky-50 border border-sky-200 px-2 py-0.5 rounded-full font-medium shrink-0">
            {{ activeSpot?.transitToNext ? activeSpot.transitToNext.split('·')[0] : '推荐出行' }}
          </span>
        </div>

        <!-- 4 种交通方式快捷切换 -->
        <div class="grid grid-cols-4 gap-1.5">
          <button
            v-for="mode in travelModeOptions"
            :key="mode.key"
            type="button"
            class="h-7 rounded-lg border text-xs font-bold transition flex items-center justify-center gap-1 cursor-pointer"
            :class="selectedTravelMode === mode.key ? 'bg-sky-600 text-white border-sky-600 shadow-xs' : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-50'"
            @click="changeTravelMode(mode.key)"
          >
            <span>{{ mode.icon }}</span>
            <span>{{ mode.label }}</span>
          </button>
        </div>

        <!-- 路线测算状态提示 -->
        <div
          v-if="segmentStatusText"
          class="rounded-xl border border-sky-200 bg-sky-50/85 p-2 text-xs text-sky-900 leading-snug font-medium flex items-center justify-between gap-1 animate-fade-in"
        >
          <span class="truncate">{{ segmentStatusText }}</span>
          <button
            type="button"
            class="text-sky-600 font-bold shrink-0 hover:underline text-[11px]"
            @click="fitBranchViewport"
          >
            聚焦此段
          </button>
        </div>
      </div>

      <!-- 终点站提示 (到达当天最后一站) -->
      <div v-else class="text-xs text-emerald-800 font-medium flex items-center gap-1.5 bg-emerald-50/90 p-2.5 rounded-xl border border-emerald-200">
        <span class="text-base">🏁</span>
        <div class="leading-relaxed">
          <div class="font-bold">当日游玩动线已到终点站</div>
          <div class="text-[11px] text-emerald-700 mt-0.5">
            {{ activeSpot?.transitToNext || '游览结束后拼车返回全季酒店休息整备，完成全天闭环。' }}
          </div>
        </div>
      </div>

      <!-- 底部上一站/下一站快捷翻页 -->
      <div class="flex items-center justify-between pt-1 border-t border-slate-100">
        <button
          type="button"
          class="px-3 py-1.5 rounded-lg border border-slate-200 text-xs font-bold text-slate-700 hover:bg-slate-50 disabled:opacity-35 disabled:pointer-events-none cursor-pointer"
          :disabled="currentSpotIndex === 0"
          @click="handleSpotStep(-1)"
        >
          ◀ 上一站
        </button>
        <span class="text-xs text-slate-400 font-medium">
          {{ currentSpotIndex + 1 }} / {{ currentDayData.spots.length }}
        </span>
        <button
          type="button"
          class="px-3 py-1.5 rounded-lg bg-orange-500 border border-orange-500 text-xs font-bold text-white hover:bg-orange-600 disabled:opacity-35 disabled:pointer-events-none cursor-pointer shadow-xs"
          :disabled="currentSpotIndex === currentDayData.spots.length - 1"
          @click="handleSpotStep(1)"
        >
          下一站 ➔
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from 'vue';
import {
  DAILY_ROUTES,
  VERIFIED_HOTEL_COORD,
  TRAVEL_MODES,
  RECOMMENDED_TRAVEL_MODES,
  PINGTAN_BOUNDS,
  PINGTAN_CENTER,
  PINGTAN_ZOOM,
  AMAP_WEB_KEY
} from '../data/pingtanData.js';
import closedLoopPaths from '../data/closed_loop_paths.json';

const daysList = [
  { id: 'd1', label: 'Day 1', theme: '抵岛团建', summary: '高铁➔龙凤头➔海岛菜', color: '#4f46e5' },
  { id: 'd2', label: 'Day 2', theme: '北线风车', summary: '仙人井➔环岛➔落日', color: '#0284c7' },
  { id: 'd3', label: 'Day 3', theme: '南线地标', summary: '日出➔68海里➔白沙', color: '#059669' },
  { id: 'd4', label: 'Day 4', theme: '早市返程', summary: '早市➔特产➔车站', color: '#d97706' }
];

const currentDayKey = ref('d1');
// -1 表示全天路线总览模式；>= 0 表示选中具体的单站点/单分支路线
const currentSpotIndex = ref(-1);
const selectedTravelMode = ref('taxi');
const isSatellite = ref(false);
const segmentStatusText = ref('');
const chipsBarRef = ref(null);
const chipRefs = ref({});

let amapInstance = null;
let satelliteLayer = null;
let currentMarkers = [];
let currentPolyline = null;
let segmentPolyline = null;
let hotelMarker = null;
let routeRenderVersion = 0;

const travelModeOptions = computed(() => Object.values(TRAVEL_MODES));
const currentDayData = computed(() => DAILY_ROUTES[currentDayKey.value]);
const activeSpot = computed(() => {
  if (currentSpotIndex.value < 0) return null;
  return currentDayData.value?.spots[currentSpotIndex.value] || null;
});
const hasNextSpot = computed(() => {
  if (currentSpotIndex.value < 0) return false;
  return currentSpotIndex.value < (currentDayData.value?.spots.length || 0) - 1;
});
const nextSpot = computed(() => {
  if (!hasNextSpot.value) return null;
  return currentDayData.value?.spots[currentSpotIndex.value + 1] || null;
});

const amapNavUrl = computed(() => {
  if (!activeSpot.value || !isValidCoord(activeSpot.value.coord)) return '#';
  const c = activeSpot.value.coord;
  return `https://uri.amap.com/marker?position=${c[0]},${c[1]}&name=${encodeURIComponent(activeSpot.value.name)}`;
});

function isValidCoord(coord) {
  return Array.isArray(coord) && coord.length >= 2 && Number.isFinite(coord[0]) && Number.isFinite(coord[1]);
}

// 初始化高德地图 (极简纯净，移除桌面端 ToolBar 放大缩小控件)
function initMap() {
  if (typeof window.AMap === 'undefined') return;
  if (amapInstance) return;

  const container = document.getElementById('amap-vue-container');
  if (!container || container.offsetWidth === 0 || container.offsetHeight === 0) return;

  try {
    amapInstance = new window.AMap.Map('amap-vue-container', {
      zoom: PINGTAN_ZOOM,
      center: PINGTAN_CENTER,
      viewMode: '3D',
      pitch: 20,
      mapStyle: 'amap://styles/fresh'
    });

    if (window.AMap.Scale) {
      amapInstance.addControl(new window.AMap.Scale());
    }

    amapInstance.on('complete', () => {
      handleDaySelect(currentDayKey.value);
    });

    window.vueAmap = amapInstance;
  } catch (err) {
    console.error('AMap init error:', err);
  }
}

function ensureMapInit() {
  if (!amapInstance) {
    initMap();
  } else {
    nextTick(() => {
      amapInstance.resize();
      if (currentSpotIndex.value === -1) {
        fitCurrentDayViewport();
      } else if (segmentPolyline) {
        fitBranchViewport();
      }
    });
  }
}

// 绘制大本营及当天所有打卡点 Marker
function renderDayMarkers(dayKey) {
  if (!amapInstance) return;
  const route = DAILY_ROUTES[dayKey];
  if (!route) return;

  // 清除已有景点 Marker 与酒店 Marker
  if (currentMarkers.length) {
    currentMarkers.forEach(item => {
      if (item && item.marker) amapInstance.remove(item.marker);
    });
    currentMarkers = [];
  }
  if (hotelMarker) {
    amapInstance.remove(hotelMarker);
    hotelMarker = null;
  }

  // 绘制大本营据点：全季酒店 (专属黑金色图钉)
  const hotelHtml = `
    <div class="custom-amap-marker" style="color: #1e293b; z-index: 120;">
      <div class="marker-badge-bubble shadow-md flex items-center gap-1 font-bold" style="background: linear-gradient(135deg, #1e293b, #0f172a); color: #ffffff; border: 1.5px solid #fbbf24; padding: 2px 8px;">
        <span>🏨</span>
        <span style="font-size: 10px; color: #fef08a;">全季酒店(大本营)</span>
      </div>
      <div class="marker-pin-tip" style="border-top-color: #0f172a;"></div>
      <div class="marker-pin-shadow"></div>
    </div>
  `;
  hotelMarker = new window.AMap.Marker({
    position: VERIFIED_HOTEL_COORD,
    content: hotelHtml,
    anchor: 'bottom-center',
    offset: new window.AMap.Pixel(0, 0),
    zIndex: 120
  });
  hotelMarker.on('click', () => {
    segmentStatusText.value = '🏨 全季酒店（平潭红湖东路32号）：团队唯一核心驻地大本营';
    amapInstance.panTo(VERIFIED_HOTEL_COORD);
  });
  amapInstance.add(hotelMarker);

  // 绘制景点 Marker
  const defaultColor = route.color || '#0284c7';
  route.spots.forEach((spot, idx) => {
    if (!isValidCoord(spot.coord)) return;

    const isHotelCoord = Math.hypot(spot.coord[0] - VERIFIED_HOTEL_COORD[0], spot.coord[1] - VERIFIED_HOTEL_COORD[1]) < 0.0001;
    if (isHotelCoord) return;

    const isActive = idx === currentSpotIndex.value;
    const markerHtml = createMarkerHtml(spot, idx, isActive, defaultColor);

    const marker = new window.AMap.Marker({
      position: spot.coord,
      content: markerHtml,
      anchor: 'bottom-center',
      offset: new window.AMap.Pixel(0, 0),
      zIndex: isActive ? 999 : 100 + idx
    });

    marker.on('click', () => {
      handleSpotSelect(idx);
    });

    amapInstance.add(marker);
    currentMarkers.push({ marker, spot, idx });
  });
}

// 绘制全天闭环动线折线
function renderFullDayPolyline(dayKey) {
  if (!amapInstance) return;

  if (currentPolyline) {
    amapInstance.remove(currentPolyline);
    currentPolyline = null;
  }
  if (segmentPolyline) {
    amapInstance.remove(segmentPolyline);
    segmentPolyline = null;
  }

  const path = closedLoopPaths[dayKey] || [];
  if (!path.length) return;

  const route = DAILY_ROUTES[dayKey];
  const strokeColor = route?.color || '#0284c7';

  currentPolyline = new window.AMap.Polyline({
    path,
    isOutline: true,
    outlineColor: '#ffffff',
    borderWeight: 2,
    strokeColor: strokeColor,
    strokeOpacity: 0.88,
    strokeWeight: 6,
    strokeStyle: 'solid',
    lineJoin: 'round',
    lineCap: 'round',
    showDir: true,
    zIndex: 50
  });

  amapInstance.add(currentPolyline);
}

// 构建 Marker HTML：选中项为醒目橙色高亮，普通项为当日主题色
function createMarkerHtml(spot, idx, isActive, defaultColor) {
  if (isActive) {
    return `
      <div class="custom-amap-marker marker-active" style="color: #ea580c; z-index: 999;">
        <div class="marker-badge-bubble active-highlight flex items-center gap-1 font-bold" style="background: linear-gradient(135deg, #f97316, #ea580c); color: #ffffff; border: 2.5px solid #ffffff; padding: 3px 9px; box-shadow: 0 0 0 3.5px rgba(249, 115, 22, 0.45), 0 8px 18px rgba(0, 0, 0, 0.35); transform: scale(1.15);">
          <span style="background: #ffffff; color: #ea580c; border-radius: 999px; width: 15px; height: 15px; display: inline-flex; align-items: center; justify-content: center; font-size: 10px; font-weight: 800;">${idx + 1}</span>
          <span style="font-size: 12px; font-weight: 800; letter-spacing: 0.2px;">${spot.shortName || spot.name}</span>
          <span style="width: 6px; height: 6px; border-radius: 999px; background: #ffffff; margin-left: 2px; display: inline-block;"></span>
        </div>
        <div class="marker-pin-tip" style="border-top-color: #ea580c; border-width: 7px 5px 0 5px;"></div>
        <div class="marker-pin-shadow" style="width: 10px; height: 10px;"></div>
      </div>
    `;
  }

  const markerColor = defaultColor || '#0284c7';
  return `
    <div class="custom-amap-marker" style="color: ${markerColor}">
      <div class="marker-badge-bubble shadow-sm flex items-center gap-1 font-bold" style="background-color: ${markerColor}; color: #ffffff; border: 1.5px solid #ffffff; opacity: 0.94;">
        <span style="background: rgba(255,255,255,0.28); border-radius: 999px; width: 14px; height: 14px; display: inline-flex; align-items: center; justify-content: center; font-size: 9px; font-family: monospace;">${idx + 1}</span>
        <span>${spot.shortName || spot.name}</span>
      </div>
      <div class="marker-pin-tip" style="border-top-color: ${markerColor};"></div>
      <div class="marker-pin-shadow"></div>
    </div>
  `;
}

// 动态更新 Marker 高亮状态 (activeIdx 为 -1 时全部不高亮)
function updateSpotMarkersHighlight(activeIdx) {
  if (!amapInstance || !currentMarkers.length) return;
  const route = currentDayData.value;
  if (!route) return;

  const defaultColor = route.color || '#0284c7';

  currentMarkers.forEach(({ marker, spot, idx }) => {
    const isActive = idx === activeIdx;
    marker.setContent(createMarkerHtml(spot, idx, isActive, defaultColor));
    marker.setzIndex(isActive ? 999 : 100 + idx);
  });
}

// 聚焦当天全景视野 (包含全部景点与全天闭环折线)
function fitCurrentDayViewport() {
  if (!amapInstance) return;
  const overlays = currentMarkers.map(item => item.marker).filter(Boolean);
  if (currentPolyline) overlays.push(currentPolyline);
  if (hotelMarker) overlays.push(hotelMarker);
  if (overlays.length > 0) {
    amapInstance.setFitView(overlays, false, [35, 25, 25, 25]);
  } else {
    fitPingtanCounty();
  }
}

// 聚焦当前单分支路线视野
function fitBranchViewport() {
  if (!amapInstance) return;
  if (segmentPolyline) {
    amapInstance.setFitView([segmentPolyline], false, [50, 40, 40, 40]);
  } else if (activeSpot.value && isValidCoord(activeSpot.value.coord)) {
    amapInstance.panTo(activeSpot.value.coord);
  }
}

// 呈现平潭县全县轮廓
function fitPingtanCounty() {
  if (!amapInstance) return;
  try {
    const bounds = new window.AMap.Bounds(PINGTAN_BOUNDS[0], PINGTAN_BOUNDS[1]);
    amapInstance.setBounds(bounds);
  } catch (e) {
    amapInstance.setCenter(PINGTAN_CENTER);
    amapInstance.setZoom(PINGTAN_ZOOM);
  }
}

function focusHotelBasecamp() {
  if (!amapInstance) return;
  amapInstance.setCenter(VERIFIED_HOTEL_COORD);
  amapInstance.setZoom(15);
  segmentStatusText.value = '🏨 全季酒店（平潭红湖东路32号）：团队大本营驻地';
}

// 核心逻辑 1：点击 Day 标签时，显示全天路线！
function handleDaySelect(dayId) {
  currentDayKey.value = dayId;
  const recommendedMode = RECOMMENDED_TRAVEL_MODES[dayId]?.[0] || 'taxi';
  selectedTravelMode.value = recommendedMode;
  renderDayMarkers(dayId);
  showFullDayOverview();
}

// 显示全天动线全貌
function showFullDayOverview() {
  currentSpotIndex.value = -1;
  segmentStatusText.value = '';

  if (segmentPolyline && amapInstance) {
    amapInstance.remove(segmentPolyline);
    segmentPolyline = null;
  }

  renderFullDayPolyline(currentDayKey.value);
  updateSpotMarkersHighlight(-1);
  fitCurrentDayViewport();
  scrollChipIntoView(-1);
}

// 核心逻辑 2：点击具体景点节点时，隐藏全天大线，仅渲染并聚焦该单段分支动线！
function handleSpotSelect(idx) {
  const route = currentDayData.value;
  if (!route || idx < 0 || idx >= route.spots.length) return;

  currentSpotIndex.value = idx;
  updateSpotMarkersHighlight(idx);
  scrollChipIntoView(idx);

  // 关键控制：彻底移除全天动线，使地图纯净聚焦当前分支！
  if (currentPolyline && amapInstance) {
    amapInstance.remove(currentPolyline);
    currentPolyline = null;
  }

  const spot = route.spots[idx];
  const next = route.spots[idx + 1];

  if (next && isValidCoord(spot.coord) && isValidCoord(next.coord)) {
    const recMode = RECOMMENDED_TRAVEL_MODES[currentDayKey.value]?.[idx] || 'taxi';
    selectedTravelMode.value = recMode;
    loadRouteLeg(recMode, spot, next, idx);
  } else {
    if (segmentPolyline && amapInstance) {
      amapInstance.remove(segmentPolyline);
      segmentPolyline = null;
    }
    if (isValidCoord(spot.coord) && amapInstance) {
      amapInstance.panTo(spot.coord);
    }
    if (!next) {
      segmentStatusText.value = `🏁 ${spot.name}：当日最后一站，行程圆满收官`;
    }
  }
}

function handleSpotStep(delta) {
  const newIdx = currentSpotIndex.value + delta;
  if (newIdx >= 0 && newIdx < currentDayData.value.spots.length) {
    handleSpotSelect(newIdx);
  }
}

function changeTravelMode(modeKey) {
  selectedTravelMode.value = modeKey;
  if (activeSpot.value && nextSpot.value) {
    loadRouteLeg(modeKey, activeSpot.value, nextSpot.value, currentSpotIndex.value);
  }
}

// 异步测算并高亮绘制单段分支路线
async function loadRouteLeg(modeKey, from, to, fromIdx) {
  if (!amapInstance || !isValidCoord(from?.coord) || !isValidCoord(to?.coord)) return;
  const mode = TRAVEL_MODES[modeKey] || TRAVEL_MODES.taxi;
  const renderVersion = ++routeRenderVersion;

  segmentStatusText.value = `${from.shortName || from.name} ➔ ${to.shortName || to.name} · 计算中...`;

  try {
    const params = new URLSearchParams({
      key: AMAP_WEB_KEY,
      origin: from.coord.join(','),
      destination: to.coord.join(','),
      extensions: 'base'
    });
    if (modeKey === 'taxi' || modeKey === 'ebike') params.set('strategy', '0');
    if (modeKey === 'bus') {
      params.set('city', '350128');
      params.set('cityd', '350128');
      params.set('strategy', '0');
    }

    const res = await fetch(`https://restapi.amap.com/v3/direction/${mode.endpoint}?${params.toString()}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const json = await res.json();
    if (json.status !== '1') throw new Error(json.info || '未能规划路线');

    if (renderVersion !== routeRenderVersion) return;

    let path = [];
    let distanceKm = '--';
    let durationMin = '--';

    if (modeKey === 'bus') {
      const transit = json.route?.transits?.[0];
      if (transit) {
        distanceKm = (transit.distance / 1000).toFixed(1);
        durationMin = Math.round(transit.duration / 60);
      }
    } else {
      const p = json.route?.paths?.[0];
      if (p) {
        distanceKm = (p.distance / 1000).toFixed(1);
        durationMin = Math.round(p.duration / 60);
        path = (p.steps || [])
          .flatMap(st => (st.polyline || '').split(';').map(xy => xy.split(',').map(Number)))
          .filter(isValidCoord);
      }
    }

    if (path.length && amapInstance) {
      if (segmentPolyline) {
        amapInstance.remove(segmentPolyline);
      }
      segmentPolyline = new window.AMap.Polyline({
        path,
        isOutline: true,
        outlineColor: '#ffffff',
        borderWeight: 2,
        strokeColor: mode.color,
        strokeOpacity: 0.98,
        strokeWeight: 7,
        strokeStyle: mode.strokeStyle,
        lineJoin: 'round',
        lineCap: 'round',
        showDir: true,
        zIndex: 70
      });
      amapInstance.add(segmentPolyline);
      amapInstance.setFitView([segmentPolyline], false, [50, 40, 40, 40]);
    }

    segmentStatusText.value = `${from.shortName || from.name} ➔ ${to.shortName || to.name} · ${mode.label} ${distanceKm}km · 约 ${durationMin}分钟`;

  } catch (err) {
    if (renderVersion !== routeRenderVersion) return;
    segmentStatusText.value = `${from.shortName || from.name} ➔ ${to.shortName || to.name}：参考推荐交通出行`;
  }
}

function toggleSatellite() {
  if (!amapInstance) return;
  if (!satelliteLayer) {
    satelliteLayer = new window.AMap.TileLayer.Satellite();
    amapInstance.add(satelliteLayer);
  }
  isSatellite.value = !isSatellite.value;
  if (isSatellite.value) {
    satelliteLayer.show();
  } else {
    satelliteLayer.hide();
  }
}

function scrollChipIntoView(idx) {
  nextTick(() => {
    const el = chipRefs.value[idx];
    if (el && el.parentElement) {
      el.scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' });
    }
  });
}

onMounted(() => {
  nextTick(() => {
    initMap();
  });
});

onBeforeUnmount(() => {
  if (amapInstance) {
    amapInstance.destroy();
    amapInstance = null;
  }
});

defineExpose({
  handleDaySelect,
  showFullDayOverview,
  fitCurrentDayViewport,
  fitPingtanCounty,
  ensureMapInit
});
</script>

<style scoped>
.map-tab-scroll-container {
  -webkit-overflow-scrolling: touch;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
  animation: fadeIn 0.2s ease-out;
}
</style>
