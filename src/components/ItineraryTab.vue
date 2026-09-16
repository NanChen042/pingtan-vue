<template>
  <div class="itinerary-tab-container flex-1 overflow-y-auto no-scrollbar p-3 space-y-3">
    <!-- 顶部单日切换胶囊栏 (Vant Tabs) -->
    <div class="sticky top-0 z-20 bg-slate-50/95 backdrop-blur py-1 border-b border-slate-200/60">
      <van-tabs
        v-model:active="activeDayKey"
        color="#0284c7"
        shrink
        line-width="24px"
        class="van-itinerary-tabs"
      >
        <van-tab name="d1" title="D1 抵岛团建" />
        <van-tab name="d2" title="D2 北线精华" />
        <van-tab name="d3" title="D3 南线地标" />
        <van-tab name="d4" title="D4 老城返程" />
      </van-tabs>
    </div>

    <!-- 当天主题卡片 -->
    <div class="bg-white rounded-2xl p-4 border border-slate-200/90 shadow-xs space-y-2.5">
      <div class="flex items-center justify-between">
        <van-tag :color="currentDay.color" round size="medium">
          {{ currentDay.dateBadge }}
        </van-tag>
        <van-tag type="primary" plain size="medium">
          {{ currentDay.stats }}
        </van-tag>
      </div>
      <h2 class="text-base font-bold text-slate-900">{{ currentDay.title }}</h2>
      <p class="text-xs text-slate-600 leading-relaxed bg-slate-50 p-2.5 rounded-xl border border-slate-100">
        {{ currentDay.desc }}
      </p>

      <!-- 快速联动：在地图上全景查看 -->
      <div class="pt-1 flex justify-end">
        <van-button
          size="small"
          type="primary"
          plain
          round
          icon="location-o"
          @click="$emit('switch-to-map', activeDayKey)"
        >
          在地图上全景查看
        </van-button>
      </div>
    </div>

    <!-- 关键警示横幅 (Vant NoticeBar) -->
    <div v-if="activeDayKey === 'd4'" class="rounded-2xl overflow-hidden shadow-xs border border-amber-200">
      <van-notice-bar
        wrapable
        :scrollable="false"
        color="#92400e"
        background="#fef3c7"
        left-icon="warning-o"
      >
        <strong>【绝对红线】10:50 前必须进平潭高铁站！</strong><br />
        11:43 发车赴福州，福州站换乘仅 36 分钟直奔上海虹桥。万勿因老城特产采购延误！
      </van-notice-bar>
    </div>

    <div v-if="activeDayKey === 'd1'" class="rounded-2xl overflow-hidden shadow-xs border border-indigo-200">
      <van-notice-bar
        wrapable
        :scrollable="false"
        color="#3730a3"
        background="#e0e7ff"
        left-icon="fire-o"
      >
        <strong>18:30 团建海鲜晚餐：</strong><br />
        君山镇【岭上·海岛菜】观海露台享用团建晚餐，品尝地道海岛大餐。
      </van-notice-bar>
    </div>

    <div v-if="activeDayKey === 'd2'" class="rounded-2xl overflow-hidden shadow-xs border border-sky-200">
      <van-notice-bar
        wrapable
        :scrollable="false"
        color="#0369a1"
        background="#e0f2fe"
        left-icon="clock-o"
      >
        <strong>长江澳落日约车重要提示：</strong><br />
        18:30 退潮日落时需提前 15-20 分钟在滴滴或高德提前呼叫网约快车（按3人/车组队），避免落日后数百人集中排队。
      </van-notice-bar>
    </div>

    <div v-if="activeDayKey === 'd3'" class="rounded-2xl overflow-hidden shadow-xs border border-emerald-200">
      <van-notice-bar
        wrapable
        :scrollable="false"
        color="#065f46"
        background="#d1fae5"
        left-icon="smile-o"
      >
        <strong>龙王头日出自选参与：</strong><br />
        05:20 出发观赏日出，随后回酒店早餐补觉至 09:00；不看日出可睡到自然醒集合。
      </van-notice-bar>
    </div>

    <!-- 逐日时间轴节点流 -->
    <div class="space-y-3 relative pl-3 border-l-2 border-slate-200/80 ml-2 mt-2 pb-6">
      <div
        v-for="(spot, index) in currentDay.spots"
        :key="spot.id || index"
        class="relative space-y-2"
      >
        <!-- 时间轴圆形针点 -->
        <span
          class="absolute -left-[19px] top-2 w-6 h-6 rounded-full border-2 border-white shadow-xs flex items-center justify-center text-[10px] font-bold text-white"
          :style="{ backgroundColor: currentDay.color }"
        >
          {{ index + 1 }}
        </span>

        <!-- 节点卡片主体 -->
        <div class="bg-white rounded-2xl p-3.5 border border-slate-200/80 shadow-xs hover:shadow-sm transition space-y-2 ml-2">
          <div class="flex items-start justify-between gap-2">
            <div>
              <div class="flex items-center gap-1.5 flex-wrap">
                <span class="text-base">{{ spot.icon || '📍' }}</span>
                <h3 class="text-sm font-bold text-slate-900">{{ spot.name }}</h3>
              </div>
              <van-tag type="warning" plain size="small" class="mt-1">
                {{ spot.tag }}
              </van-tag>
            </div>
            <van-tag size="medium" plain round>
              {{ spot.badge }}
            </van-tag>
          </div>

          <p class="text-xs text-slate-600 leading-relaxed bg-slate-50 p-2.5 rounded-xl border border-slate-100">
            {{ spot.desc }}
          </p>

          <!-- 导航至下一段交通说明 -->
          <div
            v-if="spot.transitToNext"
            class="pt-2 border-t border-slate-100 flex items-center justify-between text-xs text-sky-700 font-medium"
          >
            <div class="flex items-center gap-1 truncate pr-2">
              <span>➔</span>
              <span class="truncate">{{ spot.transitToNext }}</span>
            </div>
            <van-button
              size="mini"
              type="warning"
              plain
              round
              icon="guide-o"
              :url="`https://uri.amap.com/marker?position=${spot.coord[0]},${spot.coord[1]}&name=${encodeURIComponent(spot.name)}`"
              target="_blank"
            >
              导航
            </van-button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { DAILY_ROUTES } from '../data/pingtanData.js';

const emit = defineEmits(['switch-to-map']);

const activeDayKey = ref('d1');
const currentDay = computed(() => DAILY_ROUTES[activeDayKey.value]);
</script>

<style scoped>
.itinerary-tab-container {
  height: 100%;
}
.van-itinerary-tabs :deep(.van-tabs__wrap) {
  height: 38px;
}
.van-itinerary-tabs :deep(.van-tab) {
  font-size: 12px;
  font-weight: 700;
}
</style>
