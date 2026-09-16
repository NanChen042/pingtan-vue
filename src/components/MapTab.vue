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
      <div class="relative h-[310px] sm:h-[350px] w-full bg-slate-100">
        <div id="amap-vue-container" class="w-full h-full"></div>

        <!-- 右上角快捷操作工具胶囊 -->
        <div class="absolute top-2 right-2 z-10 flex items-center gap-1.5">
          <!-- 卫星影像切换 -->
          <button
            type="button"
            class="bg-white/95 backdrop-blur-md px-2 py-1 rounded-lg border border-slate-200/90 text-[11px] font-semibold text-slate-700 shadow-sm hover:bg-white active:scale-95 transition flex items-center gap-1 cursor-pointer"
            :title="isSatellite ? '矢量地图' : '卫星底图'"
            @click="toggleSatellite"
          >
            <span>{{ isSatellite ? '🗺️' : '🛰️' }}</span>
            <span>{{ isSatellite ? '矢量' : '卫星' }}</span>
          </button>

          <!-- 纯净地点 vs 单段路线切换 (解决路线混乱问题) -->
          <button
            type="button"
            class="bg-white/95 backdrop-blur-md px-2 py-1 rounded-lg border text-[11px] font-bold shadow-sm active:scale-95 transition flex items-center gap-1 cursor-pointer"
            :class="showRouteLine ? 'text-amber-700 border-amber-300 bg-amber-50/95 ring-1 ring-amber-300/60' : 'text-slate-700 border-slate-200/90 hover:bg-white'"
            :title="showRouteLine ? '当前显示单段接驳动线（点击切换仅看地点）' : '当前为纯净地点标记模式（点击显示单段接驳动线）'"
            @click="toggleRouteDisplay"
          >
            <span>{{ showRouteLine ? '🛣️' : '📍' }}</span>
            <span>{{ showRouteLine ? '单段动线' : '仅看地点' }}</span>
          </button>

          <!-- 当日动线最佳视野 -->
          <button
            type="button"
            class="bg-white/95 backdrop-blur-md px-2 py-1 rounded-lg border border-slate-200/90 text-[11px] font-semibold text-sky-700 shadow-sm hover:bg-white active:scale-95 transition flex items-center gap-1 cursor-pointer"
            title="聚焦当天景点与大本营"
            @click="fitCurrentDayViewport"
          >
            <span>🎯</span>
            <span>视野</span>
          </button>

          <!-- 平潭县全县完整视野 -->
          <button
            type="button"
            class="bg-white/95 backdrop-blur-md px-2 py-1 rounded-lg border border-slate-200/90 text-[11px] font-semibold text-emerald-700 shadow-sm hover:bg-white active:scale-95 transition flex items-center gap-1 cursor-pointer"
            title="完整呈现平潭县全境"
            @click="fitPingtanCounty"
          >
            <span>🏝️</span>
            <span>全县</span>
          </button>
        </div>

        <!-- 左下角平潭县地理标识 -->
        <div class="absolute bottom-2 left-2 z-10 bg-slate-900/70 backdrop-blur-md text-white text-[9px] px-2 py-0.5 rounded-md pointer-events-none flex items-center gap-1">
          <span class="w-1.5 h-1.5 rounded-full bg-emerald-400"></span>
          <span>平潭县全境 · GCJ-02</span>
        </div>
      </div>

      <!-- 横向景点快速轻触链条 (带高亮突出选中的景点) -->
      <div class="px-2.5 py-2 border-t border-slate-100 bg-slate-50/95 flex items-center gap-2 overflow-x-auto no-scrollbar" ref="chipsBarRef">
        <button
          v-for="(spot, idx) in currentDayData.spots"
          :key="spot.id || idx"
          :ref="el => chipRefs[idx] = el"
          type="button"
          class="shrink-0 px-3.5 py-1.5 rounded-full text-xs font-bold transition-all duration-200 flex items-center gap-1.5 cursor-pointer border"
          :class="idx === currentSpotIndex
            ? 'bg-gradient-to-r from-amber-500 via-orange-500 to-orange-600 text-white border-orange-500 shadow-md ring-2 ring-orange-300/80 scale-105'
            : 'bg-white text-slate-700 border-slate-200 hover:border-slate-300 hover:bg-slate-50 shadow-2xs'"
          @click="handleSpotSelect(idx)"
        >
          <span class="text-sm shrink-0">{{ spot.icon }}</span>
          <span class="whitespace-nowrap">{{ spot.shortName || spot.name }}</span>
          <span
            v-if="idx === currentSpotIndex"
            class="w-2 h-2 rounded-full bg-white shrink-0 animate-pulse shadow-xs"
          ></span>
        </button>
      </div>
    </div>

    <!-- 3. 当前选中景点·全景交互控制卡 (标准 Vant Inset 卡片) -->
    <van-cell-group inset class="!mx-3 !mt-2.5 shadow-sm border border-slate-200/80 !p-3 space-y-2.5 bg-white">
      <!-- 站点步进控制器 (上站 / 下站推进景点) -->
      <div class="flex items-center justify-between gap-2 pb-2 border-b border-slate-100">
        <van-button
          size="small"
          icon="arrow-left"
          round
          class="font-bold shrink-0"
          :disabled="currentSpotIndex === 0"
          @click="handleSpotStep(-1)"
        >
          上站
        </van-button>

        <div class="min-w-0 flex-1 text-center">
          <div class="flex items-center justify-center gap-1.5">
            <span class="text-base">{{ activeSpot?.icon || '📍' }}</span>
            <h3 class="font-extrabold text-orange-950 text-xs sm:text-sm truncate">{{ activeSpot?.name }}</h3>
            <van-tag color="#f97316" round size="small" class="shrink-0 font-bold">
              {{ activeSpot?.badge || `${currentSpotIndex + 1}/${currentDayData.spots.length}` }}
            </van-tag>
          </div>
          <p class="text-[11px] text-amber-700 font-medium mt-0.5 truncate">{{ activeSpot?.tag }}</p>
        </div>

        <van-button
          size="small"
          type="primary"
          round
          class="font-bold shrink-0 shadow-xs"
          :disabled="currentSpotIndex === currentDayData.spots.length - 1"
          @click="handleSpotStep(1)"
        >
          下站 ➔
        </van-button>
      </div>

      <!-- 站点游玩说明与高德导航直达 -->
      <div class="flex items-start justify-between gap-2 bg-slate-50 p-2.5 rounded-xl text-xs border border-slate-100">
        <p class="text-[11px] text-slate-600 leading-relaxed flex-1">
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

      <!-- 前往下一站接驳方式与测算 -->
      <div v-if="hasNextSpot" class="pt-1 space-y-2">
        <div class="flex items-center justify-between text-xs">
          <span class="text-slate-600 font-semibold flex items-center gap-1 truncate pr-2">
            <span>➔ 下一站：</span>
            <strong class="text-slate-900 truncate">{{ nextSpot?.shortName || nextSpot?.name }}</strong>
          </span>
          <div class="flex items-center gap-1 shrink-0">
            <button
              type="button"
              class="px-2 py-0.5 rounded-full text-[10px] font-bold border transition cursor-pointer"
              :class="showRouteLine ? 'bg-amber-50 text-amber-700 border-amber-300' : 'bg-sky-50 text-sky-700 border-sky-200 hover:bg-sky-100'"
              @click="toggleRouteDisplay"
            >
              {{ showRouteLine ? '隐藏路线(仅看地点)' : '🛣️ 显示该段路线' }}
            </button>
            <van-tag type="primary" plain size="small" class="shrink-0">
              {{ activeSpot?.transitToNext ? activeSpot.transitToNext.split('·')[0] : '推荐出行' }}
            </van-tag>
          </div>
        </div>

        <!-- 特殊接驳中转提示 (例如龙王头看日出后回全季早餐补觉) -->
        <div v-if="activeSpot?.transitToNext && activeSpot.transitToNext.includes('全季')" class="bg-indigo-50/80 border border-indigo-100 p-2 rounded-xl text-[11px] text-indigo-900 leading-snug flex items-center gap-1.5">
          <span>🏨</span>
          <span>{{ activeSpot.transitToNext }}</span>
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

        <!-- 路线实时测算提示框 -->
        <div
          v-if="segmentStatusText"
          class="rounded-xl border border-sky-200 bg-sky-50/85 p-2 text-[11px] text-sky-900 leading-snug font-medium flex items-center justify-between gap-1 animate-fade-in"
        >
          <span class="truncate">{{ segmentStatusText }}</span>
          <button
            type="button"
            class="text-sky-600 font-bold shrink-0 hover:underline text-[10px]"
            @click="fitCurrentDayViewport"
          >
            全景
          </button>
        </div>
      </div>

      <!-- 终点闭环通知 (到达当天最后一个游玩景点) -->
      <div v-else class="text-xs text-emerald-800 font-medium flex items-center gap-1.5 bg-emerald-50/90 p-2.5 rounded-xl border border-emerald-200">
        <span class="text-base">🏁</span>
        <div class="leading-relaxed">
          <div class="font-bold">当日游玩动线圆满结束</div>
          <div class="text-[11px] text-emerald-700 mt-0.5">
            {{ activeSpot?.transitToNext || '游览结束后拼车返回全季酒店休息整备，完成全天闭环。' }}
          </div>
        </div>
      </div>
    </van-cell-group>

    <!-- 4. 当日游玩动线清单 (垂直时间轴流式列表) -->
    <van-cell-group inset class="!mx-3 !mt-2.5 shadow-sm border border-slate-200/80 !p-3 space-y-2 bg-white">
      <div class="flex items-center justify-between pb-1.5 border-b border-slate-100 text-xs">
        <span class="font-bold text-slate-800 flex items-center gap-1.5">
          <span class="w-2 h-2 rounded-full" :style="{ backgroundColor: currentDayData.color }"></span>
          <span>{{ currentDayData.title }} ({{ currentDayData.spots.length }} 大核心节点)</span>
        </span>
        <span class="text-[10px] text-slate-400">点击任意节点切换定位</span>
      </div>

      <!-- 驻地大本营出发提示栏 -->
      <div class="p-2 rounded-xl bg-slate-50 border border-slate-200/60 text-[11px] text-slate-600 flex items-center justify-between">
        <div class="flex items-center gap-1.5 font-medium">
          <span>🏨</span>
          <span>大本营驻地：全季酒店（平潭红湖东路32号）</span>
        </div>
        <button type="button" class="text-sky-600 font-bold hover:underline" @click="focusHotelBasecamp">
          定位
        </button>
      </div>

      <div class="space-y-1.5 pt-0.5">
        <div
          v-for="(spot, idx) in currentDayData.spots"
          :key="spot.id || idx"
          class="p-2.5 rounded-xl border transition-all duration-200 cursor-pointer flex items-start gap-2.5"
          :class="idx === currentSpotIndex ? 'border-orange-500 bg-orange-50/75 shadow-xs ring-1 ring-orange-400/50' : 'border-slate-100 hover:border-slate-200 bg-slate-50/60'"
          @click="handleSpotSelect(idx)"
        >
          <span
            class="w-5 h-5 rounded-full flex items-center justify-center text-xs font-bold text-white shrink-0 mt-0.5 shadow-2xs transition-colors"
            :class="idx === currentSpotIndex ? 'bg-gradient-to-br from-amber-500 to-orange-500 ring-2 ring-orange-200' : ''"
            :style="idx !== currentSpotIndex ? { backgroundColor: currentDayData.color } : {}"
          >
            {{ idx + 1 }}
          </span>
          <div class="flex-1 min-w-0">
            <div class="flex items-center justify-between gap-1">
              <span class="text-xs font-bold truncate flex items-center gap-1" :class="idx === currentSpotIndex ? 'text-orange-950 font-extrabold' : 'text-slate-900'">
                <span>{{ spot.icon }}</span>
                <span>{{ spot.name }}</span>
              </span>
              <van-tag size="small" :color="idx === currentSpotIndex ? '#f97316' : currentDayData.color">{{ spot.badge }}</van-tag>
            </div>
            <p class="text-[11px] text-slate-500 mt-0.5 line-clamp-2 leading-relaxed">{{ spot.desc }}</p>
            <div v-if="spot.transitToNext" class="mt-1 text-[10px] text-sky-700 font-medium flex items-center gap-1">
              <span>➔</span>
              <span class="truncate">{{ spot.transitToNext }}</span>
            </div>
          </div>
        </div>
      </div>
    </van-cell-group>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from 'vue';
import {
  DAILY_ROUTES,
  VERIFIED_PATHS,
  VERIFIED_HOTEL_COORD,
  TRAVEL_MODES,
  RECOMMENDED_TRAVEL_MODES,
  PINGTAN_BOUNDS,
  PINGTAN_CENTER,
  PINGTAN_ZOOM,
  AMAP_WEB_KEY
} from '../data/pingtanData.js';

const daysList = [
  { id: 'd1', label: 'Day 1', theme: '抵岛团建', summary: '高铁➔龙凤头➔海岛菜', color: '#4f46e5' },
  { id: 'd2', label: 'Day 2', theme: '北线风车', summary: '仙人井➔环岛➔落日', color: '#0284c7' },
  { id: 'd3', label: 'Day 3', theme: '南线地标', summary: '日出➔68海里➔白沙', color: '#059669' },
  { id: 'd4', label: 'Day 4', theme: '早市返程', summary: '早市➔特产➔车站', color: '#d97706' }
];

const currentDayKey = ref('d1');
const currentSpotIndex = ref(0);
const selectedTravelMode = ref('taxi');
const isSatellite = ref(false);
const showRouteLine = ref(false); // 默认纯净地点模式（只标记地点不画路线，选中景点仅在需要时查看单段路线）
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
const activeSpot = computed(() => currentDayData.value?.spots[currentSpotIndex.value]);
const hasNextSpot = computed(() => currentSpotIndex.value < (currentDayData.value?.spots.length || 0) - 1);
const nextSpot = computed(() => hasNextSpot.value ? currentDayData.value.spots[currentSpotIndex.value + 1] : null);

const amapNavUrl = computed(() => {
  if (!activeSpot.value) return '#';
  const c = activeSpot.value.coord;
  return `https://uri.amap.com/marker?position=${c[0]},${c[1]}&name=${encodeURIComponent(activeSpot.value.name)}`;
});

function isValidCoord(coord) {
  return Array.isArray(coord) && coord.length >= 2 && Number.isFinite(coord[0]) && Number.isFinite(coord[1]);
}

// 初始化高德地图
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
    if (window.AMap.ToolBar) {
      amapInstance.addControl(new window.AMap.ToolBar({ position: 'RB', offset: new window.AMap.Pixel(10, 10) }));
    }

    amapInstance.on('complete', () => {
      renderDayRoute(currentDayKey.value);
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
      fitCurrentDayViewport();
    });
  }
}

// 绘制单日路线与景点
function renderDayRoute(dayKey) {
  if (!amapInstance) return;
  const route = DAILY_ROUTES[dayKey];
  if (!route) return;

  routeRenderVersion++;
  segmentStatusText.value = '';

  // 1. 清除已有覆盖物
  if (currentPolyline) {
    amapInstance.remove(currentPolyline);
    currentPolyline = null;
  }
  if (segmentPolyline) {
    amapInstance.remove(segmentPolyline);
    segmentPolyline = null;
  }
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

  const overlaysToFit = [];

  // 注意：彻底不绘制全天多段环岛交错的乱线，保持地图纯净开阔！仅在用户需要时呈现单段路线

  // 3. 绘制核心大本营据点：全季酒店 (常驻高贵深色金边专属图钉，绝不与景点混淆)
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
  overlaysToFit.push(hotelMarker);

  // 4. 绘制景点 Marker (高亮当前选中项)
  const defaultColor = route.color || '#0284c7';
  route.spots.forEach((spot, idx) => {
    if (!isValidCoord(spot.coord)) return;

    // 如果该景点就在酒店坐标上，避免完全重叠
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
    overlaysToFit.push(marker);
  });

  // 5. 动线模式检测：如果用户开启了单段路线模式，绘制当前站点的单段路线；若为仅看地点模式，保持地图纯净！
  if (showRouteLine.value) {
    const spot = route.spots[currentSpotIndex.value];
    const next = route.spots[currentSpotIndex.value + 1];
    if (next && isValidCoord(spot?.coord) && isValidCoord(next?.coord)) {
      loadRouteLeg(selectedTravelMode.value, spot, next, currentSpotIndex.value);
    }
  } else {
    const spot = route.spots[currentSpotIndex.value];
    if (spot) {
      segmentStatusText.value = `📍 纯净地点模式：已定位至 ${spot.shortName || spot.name}（点击“显示该段路线”可测算接驳）`;
    }
  }

  // 6. 自适应最佳动线视野
  fitCurrentDayViewport();
}

// 动态构建 Marker HTML：高亮项为醒目橙色，普通项为当日主题色
function createMarkerHtml(spot, idx, isActive, defaultColor) {
  if (isActive) {
    return `
      <div class="custom-amap-marker marker-active" style="color: #ea580c; z-index: 999;">
        <div class="marker-badge-bubble active-highlight flex items-center gap-1 font-bold" style="background: linear-gradient(135deg, #f97316, #ea580c); color: #ffffff; border: 2.5px solid #ffffff; padding: 3px 9px; box-shadow: 0 0 0 3.5px rgba(249, 115, 22, 0.45), 0 8px 18px rgba(0, 0, 0, 0.35); transform: scale(1.18);">
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
      <div class="marker-badge-bubble shadow-sm flex items-center gap-1 font-bold" style="background-color: ${markerColor}; color: #ffffff; border: 1.5px solid #ffffff; opacity: 0.92;">
        <span style="background: rgba(255,255,255,0.28); border-radius: 999px; width: 14px; height: 14px; display: inline-flex; align-items: center; justify-content: center; font-size: 9px; font-family: monospace;">${idx + 1}</span>
        <span>${spot.shortName || spot.name}</span>
      </div>
      <div class="marker-pin-tip" style="border-top-color: ${markerColor};"></div>
      <div class="marker-pin-shadow"></div>
    </div>
  `;
}

// 当用户切换景点时，仅动态更新 Marker 状态
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

// 聚焦当天动线视野
function fitCurrentDayViewport() {
  if (!amapInstance) return;
  const overlays = currentMarkers.map(item => item.marker).filter(Boolean);
  if (currentPolyline) overlays.push(currentPolyline);
  if (hotelMarker) overlays.push(hotelMarker);
  if (overlays.length > 0) {
    amapInstance.setFitView(overlays, false, [35, 20, 25, 20]);
  } else {
    fitPingtanCounty();
  }
}

// 完整呈现平潭县全县轮廓
function fitPingtanCounty() {
  if (!amapInstance) return;
  try {
    const bounds = new window.AMap.Bounds(PINGTAN_BOUNDS[0], PINGTAN_BOUNDS[1]);
    amapInstance.setBounds(bounds);
    segmentStatusText.value = '🏝️ 已切换至平潭县全县完整视野';
  } catch (e) {
    amapInstance.setCenter(PINGTAN_CENTER);
    amapInstance.setZoom(PINGTAN_ZOOM);
  }
}

function focusHotelBasecamp() {
  if (!amapInstance) return;
  amapInstance.setCenter(VERIFIED_HOTEL_COORD);
  amapInstance.setZoom(14.5);
  segmentStatusText.value = '🏨 全季酒店（平潭红湖东路32号）：团队大本营驻地';
}

function handleDaySelect(dayId) {
  currentDayKey.value = dayId;
  currentSpotIndex.value = 0;
  const recommendedMode = RECOMMENDED_TRAVEL_MODES[dayId]?.[0] || 'taxi';
  selectedTravelMode.value = recommendedMode;
  renderDayRoute(dayId);
  scrollChipIntoView(0);
}

function handleSpotSelect(idx) {
  const route = currentDayData.value;
  if (!route || idx < 0 || idx >= route.spots.length) return;

  currentSpotIndex.value = idx;
  updateSpotMarkersHighlight(idx);
  const spot = route.spots[idx];
  const next = route.spots[idx + 1];

  scrollChipIntoView(idx);

  if (isValidCoord(spot.coord)) {
    amapInstance.panTo(spot.coord);
  }

  // 关键控制：仅在用户开启“显示单段路线”模式且存在下一站时，绘制该单段路线；默认仅标记地点，绝不乱画路线！
  if (showRouteLine.value && next && isValidCoord(spot.coord) && isValidCoord(next.coord)) {
    const recMode = RECOMMENDED_TRAVEL_MODES[currentDayKey.value]?.[idx] || 'taxi';
    selectedTravelMode.value = recMode;
    loadRouteLeg(recMode, spot, next, idx);
  } else {
    if (segmentPolyline && amapInstance) {
      amapInstance.remove(segmentPolyline);
      segmentPolyline = null;
    }
    if (!next && isValidCoord(spot.coord)) {
      segmentStatusText.value = `🏁 ${spot.name}：当日行程圆满结束，晚间返回全季酒店休息`;
    } else {
      segmentStatusText.value = `📍 已定位至：${spot.name}（纯净地点模式，点击“显示该段路线”可测算接驳）`;
    }
  }
}

function toggleRouteDisplay() {
  showRouteLine.value = !showRouteLine.value;
  if (!showRouteLine.value) {
    if (segmentPolyline && amapInstance) {
      amapInstance.remove(segmentPolyline);
      segmentPolyline = null;
    }
    segmentStatusText.value = `📍 仅看地点模式：已定位至 ${activeSpot.value?.shortName || activeSpot.value?.name || ''}，无路线干扰`;
  } else {
    if (activeSpot.value && nextSpot.value) {
      loadRouteLeg(selectedTravelMode.value, activeSpot.value, nextSpot.value, currentSpotIndex.value);
    } else {
      segmentStatusText.value = `🏁 ${activeSpot.value?.name || ''}：当日最后一站，晚间返回全季酒店休息`;
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
  showRouteLine.value = true; // 切换具体交通方式时自动开启该单段路线
  if (activeSpot.value && nextSpot.value) {
    loadRouteLeg(modeKey, activeSpot.value, nextSpot.value, currentSpotIndex.value);
  }
}

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
      // 叠层高亮该段导航折线，完全不破坏全天底线
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
