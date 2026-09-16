<template>
  <div class="mobile-viewport-shell">
    <!-- Vant 顶部原生导航栏 -->
    <van-nav-bar
      :title="pageTitle"
      left-text="全季驻地"
      :right-text="dateRange"
      class="app-nav-bar shadow-xs"
    >
      <template #left>
        <div class="flex items-center gap-1 text-white font-bold text-xs">
          <span>🏨</span>
          <span>全季驻地</span>
        </div>
      </template>
      <template #title>
        <span class="font-bold text-sm text-white flex items-center gap-1">
          <span>🌊</span>
          <span>平潭团建·{{ currentTabName }}</span>
        </span>
      </template>
      <template #right>
        <van-tag round color="rgba(255,255,255,0.25)" text-color="#ffffff">
          9/17 - 9/20
        </van-tag>
      </template>
    </van-nav-bar>

    <!-- 主体路由页面容器 (v-show 保留各页面状态与 AMap 实例) -->
    <main class="flex-1 w-full min-h-0 relative overflow-hidden">
      <MapTab v-show="activeTab === 'map'" ref="mapTabRef" class="w-full h-full" />
      <ItineraryTab v-show="activeTab === 'itinerary'" @switch-to-map="switchToMapDay" />
      <TransportTab v-show="activeTab === 'transport'" />
      <FoodTab v-show="activeTab === 'food'" />
      <BudgetTab v-show="activeTab === 'budget'" />
      <ChecklistTab v-show="activeTab === 'checklist'" />
    </main>

    <!-- Vant 底部原生 Tabbar (:fixed="false" 参与 flex 自然流式排版，永不脱离视口、永不遮挡任何内容) -->
    <van-tabbar
      v-model="activeTab"
      :fixed="false"
      active-color="#0284c7"
      inactive-color="#64748b"
      class="app-tabbar border-t border-slate-200/90 shrink-0 z-20"
      @change="onTabChange"
    >
      <van-tabbar-item name="map" icon="location-o">动线</van-tabbar-item>
      <van-tabbar-item name="itinerary" icon="notes-o">行程</van-tabbar-item>
      <van-tabbar-item name="transport" icon="logistics">交通</van-tabbar-item>
      <van-tabbar-item name="food" icon="fire-o">美食</van-tabbar-item>
      <van-tabbar-item name="budget" icon="gold-coin-o">账单</van-tabbar-item>
      <van-tabbar-item name="checklist" icon="todo-list-o">清单</van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import MapTab from './components/MapTab.vue';
import ItineraryTab from './components/ItineraryTab.vue';
import TransportTab from './components/TransportTab.vue';
import FoodTab from './components/FoodTab.vue';
import BudgetTab from './components/BudgetTab.vue';
import ChecklistTab from './components/ChecklistTab.vue';

const activeTab = ref('map');
const mapTabRef = ref(null);
const dateRange = '9/17 - 9/20';

const tabTitles = {
  map: '动线实测',
  itinerary: '逐日排程',
  transport: '全岛交通',
  food: '地道美食',
  budget: '费用账单',
  checklist: '行前装备'
};

const currentTabName = computed(() => tabTitles[activeTab.value] || '指南');
const pageTitle = computed(() => `平潭指南·${currentTabName.value}`);

function onTabChange(tabId) {
  window.location.hash = tabId;
  if (tabId === 'map' && mapTabRef.value) {
    setTimeout(() => {
      mapTabRef.value.ensureMapInit();
    }, 120);
  }
}

function switchToMapDay(dayId) {
  activeTab.value = 'map';
  window.location.hash = 'map';
  setTimeout(() => {
    if (mapTabRef.value) {
      mapTabRef.value.ensureMapInit();
      mapTabRef.value.handleDaySelect(dayId);
    }
  }, 120);
}

onMounted(() => {
  const hash = window.location.hash.replace('#', '');
  if (hash && tabTitles[hash]) {
    activeTab.value = hash;
  }
  if (activeTab.value === 'map' && mapTabRef.value) {
    setTimeout(() => {
      mapTabRef.value.ensureMapInit();
    }, 150);
  }

  window.addEventListener('hashchange', () => {
    const newHash = window.location.hash.replace('#', '');
    if (newHash && tabTitles[newHash]) {
      activeTab.value = newHash;
      if (newHash === 'map' && mapTabRef.value) {
        setTimeout(() => {
          mapTabRef.value.ensureMapInit();
        }, 120);
      }
    }
  });
});
</script>

<style>
.app-nav-bar {
  --van-nav-bar-background: #0284c7;
  --van-nav-bar-height: 48px;
}
.app-tabbar {
  --van-tabbar-height: 50px;
  height: 50px;
  background-color: #ffffff;
}
</style>
